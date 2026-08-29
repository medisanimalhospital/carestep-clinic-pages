# CARESTEP Clinic v8.5 — Action Center & Production Stability

v8.5는 새 의료 기능을 늘리기보다, 병원 직원과 CARESTEP HQ 운영자가 **지금 처리해야 할 일과 연동 상태를 한눈에 확인**하고 문제 발생 시 바로 복구 동선으로 이동할 수 있도록 정리한 운영 안정화 버전입니다.

## 1. 병원용 Action Center

홈 화면에 `ACTION CENTER`가 추가됩니다.

한 화면에서 다음을 집계합니다.

- 오늘 Google Calendar 후속관리 업무
- 상태 악화 / 수의사 확인 필요
- SOLAPI 문자 발송 실패
- 보호자 설문 경고
- Follow-up CRM 고위험·검토 필요

각 항목의 `확인하기 / 실패 처리 / 설문 확인 / CRM 열기 / 오늘 업무` 버튼을 누르면 후속관리 운영 화면의 해당 영역으로 바로 이동합니다.

## 2. 병원용 Production Status

홈 화면에 시스템 상태가 추가됩니다.

- CARESTEP API
- SOLAPI API
- Google Calendar
- 병원 발신번호

상태는 `정상 / 연결 필요 / 설정 필요 / 보완 필요`로 구분합니다. `상태 다시 확인`을 누르면 현재 연동상태와 Action Center 데이터를 다시 조회합니다.

BASIC 등 해당 기능을 사용하지 않는 플랜은 장애로 표시하지 않고 `현재 플랜 미사용`으로 표시합니다.

## 3. HQ 승인 · 처리 센터

HQ Dashboard 상단에 다음 처리대상을 집계합니다.

- 병원 가입 승인
- 발신번호 승인 대기 + SOLAPI 등록중
- Patient Journey 자동초안 + 업데이트 승인
- 중앙 콘텐츠 재검토 기한 도래
- 전체 처리 필요 건수

카드를 누르면 해당 관리 화면으로 바로 이동하며 Journey는 `승인 필요` 필터가 자동 적용됩니다.

## 4. HQ 시스템 상태

승인센터 아래에서 다음 상태를 확인합니다.

- CARESTEP API
- D1 Database
- SOLAPI
- 발신번호 처리 대기

API 또는 DB 설정 문제가 있으면 같은 화면에 확인 필요 메시지를 표시합니다.

## 5. Patient Journey 일괄 승인

HQ → Patient Journey에 체크박스가 추가됩니다.

- 승인 대상 전체 선택
- 개별 선택
- `선택 Journey 일괄 승인`

대상은 `자동초안 / 업데이트 대기` 상태만 선택할 수 있습니다. 실행 전 최종 확인창이 표시됩니다.

의료 콘텐츠는 **Quality Gate와 수의사 검수가 완료된 중앙 Published 교육자료만 승인**하는 운영 원칙을 유지합니다.

## 6. Archived 중앙 콘텐츠 일괄 삭제

HQ → 중앙 콘텐츠에서 Archived 항목에만 체크박스가 표시됩니다.

- Archived 전체 선택
- 개별 선택
- `선택 Archived 일괄 삭제`

삭제 시 연결된 자동생성 Journey도 기존 v8.4.2 정책대로 삭제 처리하며, 과거 CRM·설문·사용통계의 최소 이력 메타데이터는 유지합니다.

## 7. 기존 기능 유지

다음 기능은 그대로 유지됩니다.

- Smart Case Flow / 새 환자 초기화 보호
- 자료 생성 → 후속관리 자동 인계
- Patient Journey Auto Sync
- 작성 질환 게시 중단 / 삭제
- 병원 전용 Journey 편집·보관·삭제
- Google Calendar / Outlook 직접 연동
- SOLAPI 예약발송 / 예약취소 / 실패 재발송
- 발신번호 온보딩 및 HQ 승인
- 보호자 설문 / 만족도
- 데이터 기반 Follow-up CRM
- 사용량 / 과금 / 요금제 관리

## 8. 데이터베이스 / Secret

- 신규 D1 SQL 직접 실행: **없음**
- 신규 Cloudflare Secret: **없음**
- Worker는 기존 API를 그대로 사용하며 v8.5 버전 표기와 health 응답만 통일했습니다.

## 9. 배포 순서

1. Cloudflare Worker → `worker-paste-ready.txt` 전체 교체 → Deploy
2. 병원용 `index.html` 교체
3. HQ용 `hq.html` 교체
4. 브라우저 `Ctrl + Shift + R`
5. 병원 HOME에서 `ACTION CENTER`와 `시스템 상태` 확인
6. HQ Dashboard에서 `승인 · 처리 센터` 확인
7. HQ → Patient Journey에서 승인 대상 체크박스 / 일괄 승인 확인
8. HQ → 중앙 콘텐츠에서 Archived 일괄 삭제 UI 확인

## 10. 권장 첫 테스트

### 병원 화면
1. PRO 계정 로그인
2. 홈 Action Center `다시 확인`
3. CARESTEP / SOLAPI / Google Calendar / 발신번호 상태 확인
4. 문자 실패 또는 설문 경고가 있다면 Action Center 버튼으로 후속관리 이동 확인

### HQ
1. HQ 로그인
2. `승인 · 처리 센터` 숫자 확인
3. Patient Journey 카드 클릭 → 승인 필요 필터 확인
4. 테스트용 자동초안 2개 선택 → 일괄 승인
5. 테스트용 Archived 콘텐츠 선택 → 일괄 삭제
