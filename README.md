# CARESTEP Clinic v8.5.1 — Pricing & Founding Clinic Policy

v8.5.1은 v8.5의 Action Center / Production Stability 기능을 유지하면서 상용화 전 요금 체계를 재정비한 버전입니다.

## 1. 기본 요금제

| 플랜 | 월 정가 | 연간 | 주요 한도 |
|---|---:|---:|---|
| PILOT | 무료 30일 | - | 직원 5명 · 패키지 100건 · 자동발송 100건 |
| BASIC | 79,000원 | 790,000원 | 직원 3명 · 패키지 500건 · 자동후속관리 미지원 |
| PRO | 169,000원 | 1,690,000원 | 직원 15명 · 패키지 3,000건 · 자동발송 500건 |
| ENTERPRISE | 329,000원 | 3,290,000원 | 직원/패키지 무제한 · 자동발송 2,000건 |

연간 가격은 정가 월 결제 약 10개월분으로 12개월을 사용하는 구조입니다.

## 2. Founding Clinic 출시가

초기 도입 병원용 월 결제 가격:

- BASIC: 59,000원/월
- PRO: 129,000원/월
- ENTERPRISE: 249,000원/월
- 적용일부터 12개월 가격 보장
- 연간 할인과 중복 적용하지 않음
- HQ에서 병원별로 Founding Clinic 적용 여부를 선택
- 초기 20개 병원 한정: 이미 Founding 이력이 있는 병원은 슬롯을 계속 차지하며, 20곳 배정 후 신규 적용은 서버에서 차단

Founding Clinic 적용 시 `price_override_krw`에 해당 플랜의 Founding 월 가격을 기록하며, `founding_locked_until`이 지난 뒤에는 정가 카탈로그 가격으로 계산합니다.

## 3. HQ 변경사항

`요금제 · 구독`에서 다음을 관리합니다.

- 정가 월 가격
- 정가 연간 가격
- Founding 월 가격
- PILOT / 플랜별 사용 한도
- 병원별 Founding Clinic 12개월 가격보장
- Founding 배정 수 / 20곳 한정 카운터
- Founding 만료일 및 현재 적용 가격

병원 상세에서 `Founding Clinic 12개월 월 가격 보장 적용`을 체크하면 월 결제로 고정됩니다. 연간 할인과 Founding 가격은 동시에 적용할 수 없습니다.

## 4. 병원 화면

계정 · 직원의 구독 상세에서 Founding Clinic 적용 병원은:

- 가격보장 적용 여부
- 보장 종료일
- 현재 적용 월 가격

을 확인할 수 있습니다.

## 5. 자동 D1 마이그레이션

직접 SQL을 실행하지 않습니다. Worker가 자동으로 다음을 처리합니다.

- `plan_catalog.founding_monthly_price_krw` 추가
- `clinic_subscriptions.founding_applied` 추가
- `clinic_subscriptions.founding_started_at` 추가
- `clinic_subscriptions.founding_locked_until` 추가
- v8.5.1 정가 / 연간 / Founding 가격으로 플랜 카탈로그 업데이트
- PILOT 한도를 패키지 100건 / 자동발송 100건으로 조정

기존 병원별 수동 가격 override는 그대로 유지됩니다. Founding을 해제할 경우 Founding 가격이 일반 수동 override로 남지 않도록 정가 기준으로 복귀합니다.

## 6. 기존 기능 유지

v8.5의 다음 기능을 그대로 유지합니다.

- 병원 Action Center
- HQ 승인 · 처리 센터
- 시스템 상태 표시
- Patient Journey Auto Sync / 상태 필터 / 일괄 승인
- 질환 게시 중단 · 삭제
- 자료 생성 → 후속관리 Smart Case Flow
- SOLAPI 예약발송 / 취소 / 발신번호 온보딩
- Google Calendar / Outlook
- 보호자 설문 / 만족도
- Follow-up CRM
- 사용량 · 정산

## 7. 배포

1. Cloudflare Worker를 `worker-paste-ready.txt`로 전체 교체 → Deploy
2. `index.html` 교체
3. `hq.html` 교체
4. 브라우저 `Ctrl + Shift + R`
5. HQ → 요금제 · 구독에서 새 가격 확인
6. PILOT 100/100, BASIC 79,000원, PRO 169,000원, ENTERPRISE 329,000원 확인
7. 테스트 병원 하나에 PRO Founding 적용 → 129,000원 / 12개월 보장 표시 확인

새 Secret은 없습니다. D1 SQL 직접 실행도 필요 없습니다.
