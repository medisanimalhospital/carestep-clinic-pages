# CARESTEP Clinic v8.1.2 · Journey Apply & Messaging History Hotfix

v8.1.2는 v8.1.1의 기능을 유지하면서, 실제 운영에서 확인된 두 가지 문제를 수정한 핫픽스입니다.

## 수정 1 · CARESTEP 기본 Journey를 현재 케이스에 바로 적용

### 증상
- CARESTEP 기본 Journey에서 `현재 케이스에 적용`을 누르면 상단의 초록색 적용 배너는 바뀌지만, 실제 D+ 실행 카드가 바로 아래에 보이지 않아 적용되지 않은 것처럼 보일 수 있었습니다.
- 병원용 복제 Journey는 편집 영역이 바로 아래에 열리기 때문에 상대적으로 정상 적용된 것처럼 보였습니다.

### 원인
- 실제 D+ 실행 카드 영역(`followup-grid`)이 Patient Journey Builder 바로 아래가 아니라 업무보드, CRM, 자동발송, 설문, 분석 영역 뒤쪽에 위치해 있었습니다.
- 또한 자동발송 예약목록 렌더러가 누락되어 후속관리 전체 렌더 흐름에서 JavaScript 오류가 발생할 수 있었습니다.

### 변경
- 실제 `후속관리 기본 설정 + D+ 실행 카드` 영역을 Patient Journey Builder 바로 아래로 자동 이동합니다.
- `현재 케이스 Journey 실행` 안내를 표시합니다.
- CARESTEP 기본 Journey와 병원 전용 Journey 모두 동일한 방식으로 직접 적용합니다.
- 적용 직후 해당 D+ 실행 카드로 자동 스크롤합니다.
- D+ 단계가 0개인 비정상 Journey는 적용하지 않고 안내합니다.

## 수정 2 · 최근 자동발송 예약 내역 복구

### 증상
- SOLAPI 예약발송은 접수되었는데 `최근 자동발송 예약` 목록이 표시되지 않거나 새로고침이 작동하지 않을 수 있었습니다.

### 원인
- 화면 코드에서 `renderMessagingJobs()` 호출은 남아 있었지만 함수 정의가 이전 버전 변경 과정에서 누락되어 있었습니다.

### 변경
- `renderMessagingJobs()`를 복구했습니다.
- 예약/처리중, 완료, 실패 건수를 다시 표시합니다.
- 예약 시각, D+ 단계, 문자/알림톡, Journey 이름, SOLAPI 상태와 취소 버튼을 표시합니다.
- `journey:<id>` 형태의 예약도 실제 Patient Journey 이름으로 표시합니다.
- 예약 POST 성공 즉시 응답값으로 목록에 먼저 표시하고, 이후 Worker에서 최신 상태를 다시 동기화합니다.
- SOLAPI 예약 성공과 목록 조회를 분리하여, 목록 새로고침에 문제가 생겨도 이미 성공한 예약을 `예약 실패`로 잘못 안내하지 않도록 했습니다.

## 버전 표기 및 캐시
- 병원 페이지 / HQ 화면 표시를 `v8.1.2`로 통일했습니다.
- HTML에 no-cache 힌트를 추가했습니다.
- 배포 후에는 `Ctrl + Shift + R` 강력 새로고침을 권장합니다.

## Worker / D1
- 새 Secret 없음
- D1 직접 SQL 실행 없음
- 기존 SOLAPI 예약/조회 API 및 데이터 구조 유지

## 배포 순서
1. Cloudflare Worker → `worker-paste-ready.txt` 전체 교체 → Deploy
2. GitHub/호스팅의 `index.html` 교체
3. `hq.html` 교체
4. 브라우저 `Ctrl + Shift + R`
5. 후속관리 화면 상단이 `v8.1.2`인지 확인
6. CARESTEP 기본 Journey 선택 → `현재 케이스에 적용`
7. 바로 아래 `현재 케이스 Journey 실행`에 D+ 카드가 나타나는지 확인
8. 테스트 번호로 예약발송 → `최근 자동발송 예약`에 즉시 표시되는지 확인
9. `예약내역 새로고침` 동작 확인

## 회귀 테스트
`TEST_REPORT.txt` 기준 20/20 PASS이며, 별도로 index/HQ/Worker JavaScript 문법검사를 통과했습니다.
