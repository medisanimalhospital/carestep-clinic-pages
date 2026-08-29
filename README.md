# CARESTEP Clinic v8.0 · Data-driven Follow-up CRM

v8.0은 v7.3의 Patient Journey, SOLAPI 자동발송, 보호자 회복 설문, Google Calendar 후속관리 결과를 **비식별 케이스 단위 CRM**으로 묶어 병원이 우선 확인해야 할 케이스를 자동 정렬합니다.

## 핵심 변경

### 1. 비식별 Follow-up CRM
후속관리 화면에 `데이터 기반 Follow-up CRM` 패널을 추가했습니다.

케이스별로 다음 운영 신호를 결합합니다.
- SOLAPI 발송 성공 / 실패 / 미확인 실패
- 보호자 설문 응답 / 미응답
- 상태 호전 / 비슷 / 악화
- 식욕 점수
- 걱정 증상
- 보호자 연락 요청
- Google Calendar에서 기록한 구조화된 연락 결과
- 수의사 확인 필요 / 해결 여부

CRM은 `case_key`로 묶이며 보호자 이름, 전화번호, 환자명은 저장하지 않습니다.

### 2. 위험점수 0~100
규칙 기반 위험점수를 계산해 다음 네 단계로 표시합니다.
- 안정: 0~19
- 주의: 20~39
- 고위험: 40~69
- 최우선: 70~100

가중 신호 예:
- 미확인 긴급 설문
- 상태 악화
- 식욕 1~2점
- 걱정 증상
- 보호자 연락 요청
- 자동발송 실패
- 연락 결과 `악화`
- 미해결 수의사 escalation

호전 응답과 호전 결과는 위험점수를 낮추는 방향으로 반영합니다.

> 이 점수는 병원 업무 우선순위용 운영 신호입니다. 진단, 예후 예측, 응급도 판정을 대신하지 않습니다.

### 3. 권장 다음 행동
각 CRM 케이스에 현재 데이터에 따른 권장 행동을 표시합니다.
예:
- 발송 실패 확인·재발송
- 보호자 연락 요청 확인
- 상태 악화 재평가
- 수의사 재평가
- 새 위험 신호 검토
- 다음 후속관리 예정
- 경과 관찰

### 4. 검토 완료 / 케이스 종료
병원 사용자는 CRM에서:
- `검토 완료`
- `케이스 종료`
- `다시 활성화`

를 처리할 수 있습니다.

종료된 케이스라도 **종료 이후 새로운 위험 신호가 발생하면 자동으로 다시 활성화**됩니다.

### 5. Google Calendar 연락 결과 → CRM 동기화
기존 후속관리 업무에서 연락 결과를 저장하면 다음 구조화 데이터가 CARESTEP CRM에도 동기화됩니다.
- 호전
- 유지
- 악화
- 연락 안 됨
- 수의사 escalation 여부
- escalation 해결 여부
- D+ 단계
- Journey ID

환자명, 일정 제목, 전화번호, 보호자명은 CRM outcome 테이블에 저장하지 않습니다.

### 6. 현재 케이스 표시
브라우저의 현재 후속관리 케이스 nonce와 병원 ID를 SHA-256으로 동일하게 계산해 CRM 목록에서 `현재 케이스` 배지를 표시합니다.

### 7. 병원 CRM KPI
선택 기간 기준으로 다음을 표시합니다.
- 활성 케이스
- 검토 필요
- 고위험 이상
- 설문 응답률
- 자동발송 성공률

기간:
- 최근 90일
- 최근 180일
- 최근 365일

### 8. HQ Follow-up CRM
HQ에 `Follow-up CRM` 메뉴를 추가했습니다.

HQ 지표:
- 전체 활성 케이스
- 검토 필요
- 최우선 케이스 수
- 평균 위험점수
- 설문 응답률
- 발송 성공률

비교:
- 병원별 CRM 운영 지표
- Patient Journey별 활성 케이스 / 검토 필요 / 고위험 / 호전율 / 발송 성공률

HQ에는 개별 case key를 노출하지 않고 병원·Journey 단위 집계만 표시합니다.

## 신규 D1 테이블
Worker가 자동 생성합니다.

- `followup_crm_outcomes`
- `followup_crm_case_reviews`

### followup_crm_outcomes
구조화된 연락 결과만 저장합니다.
- clinic_id
- anonymous case_key
- journey_id
- D+ stage
- outcome
- escalation / resolved
- source
- occurred_at
- 처리 직원

### followup_crm_case_reviews
CRM 검토 상태만 저장합니다.
- clinic_id
- anonymous case_key
- active / closed
- 검토 시각
- 검토 직원

**D1 Console에서 SQL을 직접 실행할 필요가 없습니다.**

## 개인정보 구조
v8.0 CRM에는 다음을 저장하지 않습니다.
- 보호자 이름
- 보호자 휴대전화
- 환자명
- 주소
- 이메일

자동발송에서 생성된 `case_key`는 기존과 동일하게:

`SHA-256(clinic_id | random case nonce)`의 앞 48자리

를 사용합니다.

보호자 설문의 자유메모는 기존 설문 테이블에 남지만 CRM API/HQ 집계에는 자유메모를 포함하지 않습니다.

## 요금제 권한
신규 feature: `followupCRM`

- PILOT: ON
- BASIC: OFF
- PRO: ON
- ENTERPRISE: ON

기존 `plan_catalog.features_json`은 v8.0 최초 요청에서 자동 보완합니다.

## 신규 API
병원:
- `GET /saas/crm/overview?days=180`
- `POST /saas/crm/outcomes`
- `POST /saas/crm/cases/:caseKey/review`

HQ:
- `GET /hq/followup-crm?days=180`

## 추가 Worker Secret
**없습니다.**

기존 Secret과 D1 binding을 그대로 사용합니다.
- SOLAPI_API_KEY
- SOLAPI_API_SECRET
- CARESTEP_HQ_KEY
- 기존 CARESTEP 인증 / AI 관련 Secret
- DB binding

## 배포 순서
1. Cloudflare Worker → `worker-paste-ready.txt` 전체 교체 → Deploy
2. 병원용 `index.html` 교체
3. HQ용 `hq.html` 교체
4. 브라우저 `Ctrl + Shift + R`
5. 병원 로그인 → 후속관리 → `데이터 기반 Follow-up CRM` 확인
6. HQ 로그인 → `Follow-up CRM` 메뉴 확인

## 권장 첫 테스트
1. 설문이 포함된 Journey로 테스트 환자 자동발송 예약
2. 보호자 설문에서 `악화` + `연락 요청`으로 제출
3. CARESTEP 후속관리 CRM 새로고침
4. 해당 비식별 케이스가 `최우선 / 검토 필요`로 표시되는지 확인
5. `검토 완료` 클릭
6. Google Calendar 업무에서 결과를 `호전`으로 기록
7. CRM 새로고침 후 위험도가 낮아지는지 확인
8. 케이스 종료 후 새로운 위험 신호를 만들고 자동 재활성화 여부 확인

## v7.3 기능 유지
- 질환별 Patient Journey Builder
- SOLAPI 예약 자동발송
- 발신번호 온보딩
- 발송 실패 감지 / 재발송
- 보호자 회복 설문 / 만족도
- Google Calendar / Outlook 후속관리
- SaaS 사용량 / 정산
- HQ 운영관리
