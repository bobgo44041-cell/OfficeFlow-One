## Public Deploy Security Checklist

### Code-side changes completed
- Removed client-side external holiday API key flow from `smartwork-calendar-original.html`
- Removed anonymous auth fallback from iframe apps
- Tightened account deletion to require current email confirmation
- Removed insecure Firestore rule guidance text from the travel app
- Removed client-side email/password login from `index.html`
- Removed client-side admin password gate from `corp-assets-original.html`
- Removed private post passwords and restricted private board reads to author/admin
- Switched iframe apps to read session bridge data without direct parent storage access

### Firebase Rules required before public deploy
Use the current app structure with owner-based restrictions where available:

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function signedIn() {
      return request.auth != null;
    }

    function isAdmin() {
      return signedIn() && request.auth.token.email == "bobgo44041@gmail.com";
    }

    function isSelfEmail(docId) {
      return signedIn() && request.auth.token.email == docId;
    }

    function ownsApp(appId, prefix) {
      return signedIn() && appId.matches('^' + prefix + '-' + request.auth.uid + '$');
    }

    match /ofw_board_posts/{postId} {
      allow read: if signedIn() && (
        !resource.data.isPrivate ||
        isAdmin() ||
        resource.data.authorEmail == request.auth.token.email
      );
      allow create: if signedIn() && request.resource.data.authorEmail == request.auth.token.email;
      allow update: if isAdmin() || (
        signedIn() &&
        resource.data.authorEmail == request.auth.token.email &&
        request.resource.data.authorEmail == request.auth.token.email
      ) || (
        // 조회수만 1 올리는 것은 누구나 허용한다.
        // 다른 필드는 함께 바꿀 수 없고, 증가폭도 1로 제한된다.
        signedIn() &&
        request.resource.data.diff(resource.data).affectedKeys().hasOnly(['viewCount']) &&
        request.resource.data.viewCount == resource.data.get('viewCount', 0) + 1
      );
      allow delete: if isAdmin() || (
        signedIn() &&
        resource.data.authorEmail == request.auth.token.email
      );
    }

    match /ofw_user_profiles/{docId} {
      allow read: if isAdmin() || isSelfEmail(docId);
      allow create, update, delete: if isSelfEmail(docId);
    }

    match /ofw_admin_config/{docId} {
      allow read, write: if isAdmin();
    }

    match /artifacts/{appId}/public/data/{document=**} {
      allow read, write: if
        ownsApp(appId, 'ofw-certification') ||
        ownsApp(appId, 'ofw-corp-assets') ||
        ownsApp(appId, 'ofw-fixed-assets') ||
        ownsApp(appId, 'ofw-lease') ||
        ownsApp(appId, 'ofw-smartwork-calendar') ||
        ownsApp(appId, 'ofw-software-license') ||
        ownsApp(appId, 'ofw-travel-manager') ||
        ownsApp(appId, 'ofw-vehicle-log');
    }

    match /corp_events/{docId} {
      allow create: if signedIn() && request.resource.data.ownerUid == request.auth.uid;
      allow read, update, delete: if signedIn() && resource.data.ownerUid == request.auth.uid;
    }

    match /corp_regulations/{docId} {
      allow create: if signedIn() && request.resource.data.ownerUid == request.auth.uid;
      allow read, update, delete: if signedIn() && resource.data.ownerUid == request.auth.uid;
    }

    match /corp_events_deleted/{docId} {
      allow create: if signedIn() && request.resource.data.ownerUid == request.auth.uid;
      allow read, update, delete: if signedIn() && resource.data.ownerUid == request.auth.uid;
    }

    match /corp_event_executions/{docId} {
      allow create: if signedIn() && request.resource.data.ownerUid == request.auth.uid;
      allow read, update, delete: if signedIn() && resource.data.ownerUid == request.auth.uid;
    }
  }
}
```

### Firebase Storage Rules

이 프로젝트는 Firebase Storage 를 사용하지 않는다.

- Storage 가 활성화돼 있지 않다(Blaze 요금제 필요). 콘솔에 규칙 화면 자체가
  없으므로 지금 게시할 것은 없다.
- 앱에서도 파일 업로드 UI 를 모두 제거했다.
- 나중에 Storage 를 켜게 되면 아래 규칙을 그대로 게시해 전 경로를 막아둔다.

```txt
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if false;
    }
  }
}
```

주의: 이 규칙은 SDK 를 통한 접근만 막는다. 과거에 업로드된 파일의
다운로드 URL 은 토큰이 포함된 링크라 규칙과 무관하게 계속 열린다.
완전히 차단하려면 Firebase Console 에서 해당 파일을 삭제해야 한다.

### Must-do before public launch
- Publish the rules above in Firebase Console
- Storage 는 활성화돼 있지 않아 게시할 규칙이 없다. 켜게 되면 위 규칙을 게시할 것
- Verify existing ceremony records include `ownerUid`; old documents without it will not appear after the rule tighten
- Verify Google sign-in is the only enabled public auth provider
- Disable Email/Password auth provider in Firebase Authentication if it is still enabled
- Remove remaining 업무 data from `localStorage` if the data must be cloud-only
- Review every iframe app for any mock/demo fallback data
- Add CSP and hosting headers if your hosting platform supports them
- Add Firebase App Check for Hosting/Firestore/Storage if you are publicly exposing the site

### Manual smoke test
- Google sign-in
- Account profile completion modal
- Board post create/read/delete
- Admin console: user list, banner save, usage stats
- Each iframe app opens without cloud auth errors
- Logout and relogin preserve profile and app access
