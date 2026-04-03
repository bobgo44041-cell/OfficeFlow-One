## Public Deploy Security Checklist

### Code-side changes completed
- Removed client-side external holiday API key flow from `smartwork-calendar-original.html`
- Removed anonymous auth fallback from iframe apps
- Tightened account deletion to require current email confirmation
- Removed insecure Firestore rule guidance text from the travel app

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
      allow read: if signedIn();
      allow create: if signedIn();
      allow update, delete: if isAdmin() || (
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
        ownsApp(appId, 'ofw-corp-assets') ||
        ownsApp(appId, 'ofw-fixed-assets') ||
        ownsApp(appId, 'ofw-smartwork-calendar') ||
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

### Must-do before public launch
- Publish the rules above in Firebase Console
- Verify existing ceremony records include `ownerUid`; old documents without it will not appear after the rule tighten
- Verify Google sign-in is the only enabled public auth provider you intend to support
- Remove remaining 업무 data from `localStorage` if the data must be cloud-only
- Review every iframe app for any mock/demo fallback data
- Add CSP and hosting headers if your hosting platform supports them

### Manual smoke test
- Google sign-in
- Account profile completion modal
- Board post create/read/delete
- Admin console: user list, banner save, usage stats
- Each iframe app opens without cloud auth errors
- Logout and relogin preserve profile and app access
