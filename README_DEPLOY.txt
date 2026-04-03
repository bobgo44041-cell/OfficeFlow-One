OfficeFlow One 배포 폴더

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
- DEPLOY_SECURITY_CHECKLIST.md

배포 전 확인:
1. Firebase Authorized Domains에 실제 배포 도메인 추가
2. Firestore Rules에 firestore.rules 내용 반영
3. index.html과 iframe 앱 파일들이 같은 경로에 함께 배포되는지 확인
