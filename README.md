# CARESTEP Clinic v6.6.3 · Dedicated Follow-up Calendar

## 배포
이번 패치는 **병원용 `index.html`만 교체**합니다.

1. GitHub Pages 저장소 루트 `index.html`을 v6.6.3 파일로 교체
2. Commit
3. 배포 후 브라우저에서 Ctrl+F5

`hq.html`과 Cloudflare Worker는 v6.6.1/v6.6.2에서 사용 중인 것을 그대로 유지합니다. 새 D1 테이블이나 Secret은 없습니다.

## Google OAuth에 추가할 Scope
기존 `calendar.events` 외에 아래 두 Scope가 추가됩니다.

- `https://www.googleapis.com/auth/calendar.calendarlist.readonly`
  - 사용 가능한 Google Calendar 목록을 불러오기 위해 사용
- `https://www.googleapis.com/auth/calendar.app.created`
  - CARESTEP이 전용 보조 캘린더를 만들고 그 캘린더의 이벤트를 관리하기 위해 사용

Google Cloud Console → Google Auth Platform → Data Access → Add or remove scopes에서 추가하세요.
기존 테스트 계정은 v6.6.3 배포 후 CARESTEP에서 **Google 연결**을 다시 눌러 새 권한에 동의해야 합니다.

## 새 기능
- Google Calendar 연결 후 사용 가능한 쓰기 가능 캘린더 목록 표시
- 기본 캘린더 또는 병원 전용 캘린더 선택
- **전용 캘린더 만들기** 버튼
  - 같은 계정에 `CARESTEP 후속관리` 캘린더가 이미 있으면 새로 만들지 않고 기존 캘린더 선택
  - 없으면 `CARESTEP 후속관리` 보조 캘린더 생성 후 자동 선택
- D+1·D+3·D+7·D+14 일정은 선택한 캘린더에만 등록
- 업무보드의 오늘/미완료/완료/예정도 선택한 캘린더에서만 조회
- 완료 처리/미완료 되돌리기도 선택한 캘린더의 이벤트에 적용
- 캘린더 선택값은 병원별로 **현재 브라우저 localStorage에만** 저장
- OAuth access token과 환자별 업무 큐는 CARESTEP D1에 저장하지 않음

## 기존 일정
초기 선택은 `기본 캘린더`이므로 v6.6.1/v6.6.2에서 이미 만든 일정은 그대로 보입니다.
`CARESTEP 후속관리` 전용 캘린더로 전환한 이후 새로 등록하는 일정부터 전용 캘린더에 들어갑니다. 기존 기본 캘린더 일정은 자동 이동하지 않습니다.

## 권장 운영
병원 공용 Google 계정으로 연결하고, `CARESTEP 후속관리` 전용 캘린더를 한 번 생성한 뒤 해당 캘린더를 병원 직원들과 Google Calendar에서 공유하는 방식이 가장 단순합니다.
