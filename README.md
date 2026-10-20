# 역사 방탈출: 생활 기록 복원 작전

역사 퀴즈 30문항과 Firebase 익명 로그인·Cloud Firestore 실시간 점수판을 사용하는 정적 HTML 웹앱입니다. 점수판은 레이싱 자동차가 점수에 따라 트랙을 달리는 형태로 표시됩니다.

## 저장소 파일
- `index.html`: 게임, Firebase 연결, 실시간 레이싱 점수판
- `firebase.json`: Firestore 규칙 배포 설정
- `firestore.rules`: 참가자 점수 쓰기 및 순위 읽기 규칙
- `vercel.json`: Vercel 정적 배포 설정

## 1. Firebase 프로젝트 설정
1. [Firebase Console](https://console.firebase.google.com/)에서 프로젝트를 생성하거나 기존 프로젝트를 선택합니다.
2. 프로젝트 설정 → 일반 → 내 앱에서 웹 앱을 등록하고 SDK 설정 값을 확인합니다.
3. Authentication → Sign-in method에서 **Anonymous(익명)** 로그인을 활성화합니다.
4. Firestore Database를 생성합니다.
5. `index.html`의 `firebaseConfig` 객체에서 `apiKey`, `authDomain`, `projectId`, `appId`를 실제 프로젝트 값으로 교체합니다.

웹 앱의 Firebase 설정값은 브라우저에서 보일 수 있습니다. 서비스 계정 키나 비공개 서버 키는 HTML에 넣지 마세요.

## 2. Firestore 방 만들기
Firebase Console → Firestore Database → Data에서 문서를 만듭니다.
- 컬렉션: `rooms`
- 문서 ID: 참여 코드(예: `HISTORY56`)
- 필드: `open` — Boolean 값 `true`

참가자는 입장 화면에서 이 코드를 입력합니다. 방을 닫으려면 `open`을 `false`로 변경합니다. 교사용 방 생성 화면은 포함되어 있지 않습니다.

## 3. 보안 규칙 게시
Firebase Console → Firestore Database → Rules에서 저장소의 `firestore.rules` 내용을 붙여 넣고 게시합니다. Firebase CLI를 사용한다면 저장소 폴더에서 `firebase deploy --only firestore:rules`를 실행할 수 있습니다.

규칙은 현재 30문항의 정답 인덱스 패턴 `[1, 2, 0, 3]`이 반복되는 점을 사용합니다. 문항 순서나 정답 위치를 바꾸면 규칙의 `correctAnswer()`도 함께 수정하고 테스트하세요. 실제 배포 전에 Rules Playground 또는 Emulator로 검증하세요.

## 4. Vercel 배포
1. [Vercel](https://vercel.com/)에서 GitHub 저장소 `kyuny1/escape-room`을 Import합니다.
2. Framework Preset은 **Other**로 선택하고 별도 Build Command/Output Directory 없이 Deploy합니다.
3. Firebase Authentication의 Authorized domains에 Vercel 도메인이 포함되어 있는지 확인합니다.
4. GitHub의 `main` 브랜치 변경 사항은 연결된 Vercel 프로젝트에 자동 배포됩니다.

## 5. 테스트
- [ ] Firebase 설정값 교체
- [ ] 익명 로그인 활성화
- [ ] Firestore 생성 및 규칙 게시
- [ ] `rooms/HISTORY56` 문서에 `open: true` 생성
- [ ] Vercel 도메인을 Firebase Authorized domains에 등록
- [ ] 서로 다른 브라우저에서 같은 참여 코드로 접속해 점수판이 실시간 갱신되는지 확인

## 알려진 제한
교사용 방 관리 화면은 없으므로 Firebase Console에서 참여 코드를 직접 만들어야 합니다. 점수 검증 규칙은 현재 문제 배열의 정답 순서에 의존합니다.