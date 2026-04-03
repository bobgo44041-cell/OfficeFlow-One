OfficeFlow One 배포 폴더

로컬 테스트 서버:
- `cd /mnt/c/Users/정지원/Desktop/my-app/officeflow-deploy`
- `node server.js`
- Windows에서 바로 실행: `run-local-server.bat`
- 기본 주소: `http://127.0.0.1:4173`
- Google 로그인 테스트 전 Firebase Authorized Domains에 `127.0.0.1` 추가 필요

업로드 기준:
- 메인 파일: index.html
- iframe 앱 파일:
  - ceremony-original.html
  - pdf-editor-original.html
  - corp-assets-original.html
  - smartwork-calendar-original.html
  - vehicle-log-original.html
  - fixed-assets-original.html
  - travel-manager-original.html

같이 포함한 참고 파일:
- firestore.rules
- storage.rules
- DEPLOY_SECURITY_CHECKLIST.md

배포 전 확인:
1. Firebase Authorized Domains에 실제 배포 도메인 추가
2. Firestore Rules에 firestore.rules 내용 반영
3. Firebase Storage Rules에 storage.rules 내용 반영
4. Firebase Authentication에서 Google 로그인만 활성화
5. index.html과 iframe 앱 파일들이 같은 경로에 함께 배포되는지 확인
