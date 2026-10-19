# 역사 방탈출: 생활 기록 복원 작전

Firebase 익명 로그인과 Cloud Firestore 실시간 순위 기능을 사용하는 정적 HTML 웹앱입니다.

## 파일
- `index.html`: 게임 화면과 문제/점수 처리
- `vercel.json`: Vercel 정적 배포 설정
- `firebase.json`, `firestore.rules`: Firestore 보안 규칙 배포 설정

## 1. Firebase 프로젝트 만들기
1. [Firebase Console](https://console.firebase.google.com/)에서 프로젝트를 생성합니다.
2. 프로젝트 설정(톱니바퀴) → 일반 → 내 앱에서 웹 앱(`</>`)을 추가하고 SDK 설정의 `firebaseConfig` 값을 복사합니다.
3. Authentication → Sign-in method에서 **Anonymous(익명)** 로그인을 사용 설정합니다.
4. Firestore Database를 생성합니다. 가능하면 운영 데이터와 분리된 새 프로젝트에서 먼저 테스트하세요.
5. `index.html`의 `firebaseConfig` 예시값을 실제 프로젝트 값으로 교체합니다. 이 설정은 웹앱에서 보이는 값이므로 비밀키처럼 취급하지 않습니다. 보안은 Authentication과 Firestore 규칙으로 지켜야 합니다.

## 2. 방(참여 코드) 열기
앱에는 교사용 방 관리 화면이 없습니다. Firebase Console → Firestore Database → 데이터에서 직접 문서를 만듭니다.
- 컬렉션: `rooms`
- 문서 ID: 참여 코드(예: `HISTORY56`; 앱은 입력 코드를 대문자로 변환합니다)
- 필드: `open` (boolean) = `true`

참여자는 `rooms/{참여코드}/players/{익명 UID}` 하위 문서에 저장됩니다. 활동 종료 시 방 문서의 `open`을 `false`로 바꾸세요.

## 3. 보안 규칙 검토
`firestore.rules`는 다음을 제한합니다.
- 로그인한 사용자만 방 문서를 읽을 수 있고, 클라이언트가 방 설정을 생성/수정/삭제할 수 없습니다.
- 참가자는 자신의 UID 문서만 생성/수정할 수 있습니다.
- 새 참가자는 점수 0으로 시작하고, 기존 참가자는 한 번에 한 문제씩 진행하며 점수 증가는 최대 10점으로 제한됩니다.
- 열린 방에서만 참가자 목록을 읽을 수 있어 실시간 순위가 동작합니다.

규칙은 앱의 데이터 경로에 맞춰 작성했지만, Firebase 프로젝트가 아직 없으므로 실제 프로젝트에서 Rules Playground/에뮬레이터 테스트를 완료한 것은 아닙니다. Firebase Console의 Firestore → Rules에서 내용을 검토·테스트한 후 게시하세요. **규칙을 게시하기 전에 기존 데이터베이스 규칙을 백업/검토하세요.**

### 점수 부정 방지의 한계
정답과 점수 계산이 브라우저 코드에 있으므로, 사용자가 개발자 도구를 이용해 정답/점수 흐름을 조작할 가능성을 완전히 막지는 못합니다. 현재 규칙은 참가자 간 문서 변조를 줄이는 기본 방어선입니다. 시험이나 상금 등 공정성이 중요한 상황에서는 정답 검증과 점수 계산을 Cloud Functions 등 서버 측으로 옮겨야 합니다.

## 4. GitHub → Vercel 배포
1. 이 저장소의 `index.html`, `vercel.json`, `firebase.json`, `firestore.rules`를 커밋합니다.
2. [Vercel](https://vercel.com/)에 로그인 → Add New → Project → GitHub 저장소 `kyuny1/escape-room`를 Import합니다.
3. Framework Preset은 **Other**로 선택합니다. 별도 빌드 명령이나 출력 폴더는 필요하지 않습니다.
4. Deploy를 실행합니다. 이후 `main` 브랜치 커밋이 새 배포를 트리거합니다.
5. Firebase Authentication → Settings → Authorized domains에 Vercel에서 발급한 도메인을 추가합니다(화면에 자동 등록되지 않은 경우).

## 5. 배포 전 테스트 체크리스트
- [ ] `firebaseConfig`의 모든 예시값을 실제 값으로 교체
- [ ] Anonymous 로그인 사용 설정
- [ ] Firestore 생성 및 `firestore.rules` 검토/테스트/게시
- [ ] `rooms/HISTORY56` 문서에 `open: true` 생성
- [ ] Firebase Authentication의 Authorized domains에 배포 도메인 추가
- [ ] 서로 다른 브라우저 두 개에서 참가·점수 갱신·순위 갱신 확인
- [ ] 방을 닫은 뒤 추가 답안 제출이 거부되는지 확인
