# CARESTEP Clinic v8.6 — Subscription Billing Automation

v8.6은 v8.5.1의 요금제·Founding Clinic 정책 위에 **실제 카드 정기결제 흐름**을 연결한 버전입니다.

## 1. 이번 버전 핵심

- Toss Payments 자동결제(빌링) 카드 등록
- 카드번호/CVC를 CARESTEP가 직접 받지 않는 Hosted Billing Auth 방식
- 30일 PILOT → BASIC/PRO 자동 유료전환
- BASIC ↔ PRO 셀프서비스 변경 예약
- 월 결제 / 연 결제
- Founding Clinic 12개월 가격보장 유지
- 결제 실패 → `past_due` → 7일 유예 → 일 1회 자동 재시도
- 자동 재시도 최대 5회 이후 카드 변경/수동 재시도 유도
- 결제 성공/실패 청구내역 저장
- Toss Payments 카드 매출전표 `receipt.url` 연결
- 기간 종료 시 구독해지
- 해지 전 예약취소
- 종료된 구독의 재가입 흐름
- HQ 결제 운영 대시보드
- HQ Action Center에 결제 실패 병원 포함
- Cloudflare Cron Trigger 기반 만기 자동결제
- 결제 승인 POST에 Toss `Idempotency-Key` 사용
- Billing Key AES-GCM 암호화 저장

## 2. 병원 화면

`관리 · 설정 → 계정 · 직원 → 결제 · 구독 관리`

Owner/Admin:
1. BASIC 또는 PRO 선택
2. 월/연 결제 선택
3. `플랜 변경 예약`
4. `카드 등록 / 변경`
5. Toss Payments 카드 등록창에서 인증
6. CARESTEP로 돌아오면 빌링키가 서버에서 발급·암호화 저장

Vet/Staff는 결제상태와 청구내역을 조회만 할 수 있습니다.

### 무료체험 중
카드를 미리 등록해도 즉시 결제하지 않습니다.
`PILOT trialEndsAt`에 도달하면 Cron이 선택한 BASIC/PRO를 결제하고 활성화합니다.

카드 또는 유료 플랜이 준비되지 않은 상태로 무료체험이 끝나면 `paused`가 됩니다.

### 유료 구독 중 플랜 변경
BASIC ↔ PRO 변경은 **다음 결제일부터 적용**합니다.
현재 기간의 일할계산(proration)은 v8.6에서 하지 않습니다.

### Founding Clinic
Founding 적용 중에는 월 결제만 사용할 수 있습니다.
이미 확정된 Founding 가격은 `price_override_krw`를 기준으로 12개월 동안 유지됩니다.

## 3. 결제 실패 정책

결제 실패 시:
- 구독 상태: `past_due`
- 유예기간: 7일
- 유예기간 중 기존 기능 사용 가능
- 하루 1회 자동 재시도
- 최대 자동 재시도: 5회
- 병원 화면에서 카드 변경 및 `결제 다시 시도` 가능
- HQ Dashboard / 요금제·구독에서 실패 병원 확인 가능

유예기간이 끝나면 유료 기능 접근이 차단되고 결제수단 확인을 안내합니다.

## 4. 구독해지

`구독해지 예약`을 누르면 즉시 기능을 차단하지 않습니다.

- 현재 결제기간 종료일까지 사용
- 종료 전 `해지 예약 취소` 가능
- 기간 종료 시 Cron이 `canceled` 처리
- 저장된 Toss billingKey 삭제 API 호출
- CARESTEP D1의 billingKey cipher도 비움

다시 구독하려면 BASIC/PRO를 선택하고 카드를 다시 등록하면 됩니다.

## 5. 청구내역 · 영수증

D1 `subscription_invoices`에 다음 운영 정보가 저장됩니다.

- orderId
- 플랜 / 결제주기
- 결제금액
- 성공/실패
- 시도 횟수
- 실패 코드/메시지
- paymentKey
- 승인시각
- 결제기간
- Toss receipt URL

카드 원문 번호, 유효기간, CVC는 CARESTEP에 저장하지 않습니다.

## 6. 새 Worker 설정 — 필수

Cloudflare Worker → Settings → Variables and Secrets에서 다음 3개를 추가합니다.

`TOSS_PAYMENTS_CLIENT_KEY`는 브라우저 SDK 초기화에 사용되는 공개 Client Key라 일반 Variable로 두어도 됩니다. `TOSS_PAYMENTS_SECRET_KEY`와 `CARESTEP_BILLING_ENCRYPTION_KEY`는 반드시 Secret으로 저장합니다.

### `TOSS_PAYMENTS_CLIENT_KEY`
Toss Payments 자동결제(빌링)용 Client Key

### `TOSS_PAYMENTS_SECRET_KEY`
Toss Payments 자동결제(빌링)용 Secret Key

**반드시 Worker Secret으로 저장하세요.**

### `CARESTEP_BILLING_ENCRYPTION_KEY`
D1에 저장하는 billingKey를 AES-GCM으로 암호화하기 위한 CARESTEP 전용 Secret.

- 24자 이상
- 충분히 긴 랜덤 문자열 권장
- 운영 시작 후 임의 변경 금지
- 분실/변경 시 기존 billingKey를 복호화할 수 없음

예: 비밀번호 관리도구에서 32~64자 랜덤 문자열 생성.

## 7. Toss Payments 준비

테스트:
- Toss Payments 개발자센터의 자동결제 테스트 Client/Secret Key 사용
- 테스트 환경에서 카드 등록·빌링키 발급·자동결제 API 확인

라이브:
- Toss Payments 자동결제(빌링) 추가 리스크 검토/계약 필요
- 자동결제로 계약된 MID의 Live Client/Secret Key로 교체

CARESTEP은 SDK 기반 카드 등록창을 사용합니다.
카드정보를 CARESTEP 입력폼이나 Worker API로 직접 받지 않습니다.

## 8. Cloudflare Cron Trigger — 필수

Worker에 `scheduled()` handler가 추가되어 있습니다.

권장:
`15 0 * * *`

Cloudflare Cron은 UTC 기준이므로 위 설정은 **매일 09:15 KST**입니다.

Dashboard:
Workers & Pages → CARESTEP Worker → Settings → Triggers → Cron Triggers → Add

자동결제 API는 Toss Payments가 자체 스케줄링하지 않으므로 이 Cron Trigger가
무료체험 자동전환 / 월·연 갱신 / 결제실패 재시도를 실행합니다.

Wrangler를 사용할 경우 참고:
```toml
[triggers]
crons = ["15 0 * * *"]
```

## 9. HQ

`HQ → 요금제 · 구독 → 정기결제 자동화`

확인 가능:
- Toss Payments 연동 상태
- 등록 카드 수
- past_due 병원 수
- 최근 30일 결제 성공 수/금액
- 최근 30일 실패 수
- 최근 청구내역 및 영수증
- 만기 결제 수동 실행

HQ Dashboard의 `승인 · 처리 센터`에도 `결제 실패`가 포함됩니다.

## 10. D1 자동 마이그레이션

SQL 직접 실행 필요 없음.

`clinic_subscriptions` 추가 컬럼:
- pending_plan_code
- pending_billing_cycle
- auto_renew
- next_payment_at
- last_payment_at
- payment_failure_count
- payment_grace_ends_at
- billing_lock_until

새 테이블:
- `clinic_payment_methods`
- `subscription_payment_intents`
- `subscription_invoices`

## 11. 중복결제 안전장치

v8.6은 다음을 사용합니다.

- D1 병원별 billing lock
- 결제시도별 결정적 orderId
- Toss Payments `Idempotency-Key`
- 동일 orderId invoice upsert
- Provider 승인 성공 후 D1 저장 실패 시 같은 결제 요청으로 재조정

따라서 네트워크 타임아웃이나 Worker/D1 일시 장애에서 동일 승인 요청이 중복 결제로 이어질 위험을 낮춥니다.

## 12. 배포 순서

1. Toss Payments 테스트 키 준비
2. Worker Secrets 3개 설정
3. `worker-paste-ready.txt` 전체 교체 → Deploy
4. Cloudflare Cron `15 0 * * *` 등록
5. `index.html` 교체
6. `hq.html` 교체
7. 브라우저 `Ctrl + Shift + R`
8. HQ → 요금제 · 구독 → `Toss Payments 정상` 확인
9. 테스트 병원 Owner → 계정 · 직원 → 결제 · 구독 관리
10. PRO / 월 결제 선택 → 카드 등록
11. 청구내역 / 카드 상태 확인

## 13. 운영 전 권장 테스트

- 테스트 카드 등록 성공
- 카드 등록 취소/failUrl
- 무료체험 병원에 PRO pending 등록
- HQ에서 만기 결제 실행
- 성공 invoice + receipt URL 확인
- 실패 테스트 코드로 `past_due` 확인
- 유예기간 표시 확인
- 카드 변경 → 수동 재결제 성공
- BASIC ↔ PRO 다음 결제일 변경
- 구독해지 예약 → 취소
- 실제 기간 종료 테스트 후 canceled + billingKey revoke 확인

## 14. 기존 기능 유지

v8.5.1까지의 CARESTEP 기능을 모두 유지합니다.
- Action Center
- Journey 승인센터 / Auto Sync
- 질환 삭제
- Patient Journey
- SOLAPI
- Calendar
- 보호자 설문
- Follow-up CRM
- Founding Clinic
- 중앙 요금제/사용량 관리
