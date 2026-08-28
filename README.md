# CARESTEP Clinic v6.5 · PLAN & SUBSCRIPTION

v6.4 HQ Admin Console + v6.3.2.4 Dynamic Medication Checklist를 유지하면서,
병원 가입 승인 / 요금제 / 구독 원장 / 플랜별 기능 제한을 추가한 버전입니다.

## 1. 신규 병원 가입 승인
- 신규 병원 가입 시 바로 로그인되지 않고 `pending` 상태로 생성됩니다.
- HQ → `가입 승인`에서 승인/거절합니다.
- 승인 시 기본 PILOT 구독이 `active`로 전환됩니다.
- 기존 v6.4 이전 병원은 자동 마이그레이션 시 `approved`로 유지되어 기존 사용이 끊기지 않습니다.

## 2. 요금제
기본 플랜: PILOT / BASIC / PRO / ENTERPRISE

가격은 임의의 상용 가격을 강제하지 않기 위해 기본 0원(가격 미설정)으로 시작합니다.
HQ → `요금제 · 구독`에서 월/연 가격을 직접 입력할 수 있습니다.

기본 기능 정책:
- PILOT: 전체 기능 / 직원 20 / 월 패키지 5000 / 월 AI 500
- BASIC: AI Content Studio·Pilot 분석·복수 ZIP 고급출력 제한 / 직원 3 / 월 패키지 300
- PRO: 전체 기능 / 직원 15 / 월 패키지 3000 / 월 AI 300
- ENTERPRISE: 전체 기능 / 0은 무제한

HQ에서 각 플랜의 기능 체크박스와 한도를 직접 바꿀 수 있습니다.

## 3. 구독 상태
- trialing
- active
- past_due
- paused
- canceled

`active`와 `trialing`만 플랜 기능을 사용할 수 있습니다.
그 외 상태에서는 계정 확인은 가능하지만 자료 생성/AI 등 구독 기능이 제한됩니다.

## 4. 병원용 페이지의 기능 제한
- AI Content Studio: `aiContent`
- Pilot 대시보드: `pilotAnalytics`
- 클라우드 설정 동기화: `cloudSync`
- 직원 초대/관리: `teamManagement`
- 중앙 Published 콘텐츠: `centralContent`
- 복수 출력 ZIP: `advancedExports`
- 퇴원 패키지: `packageOutput` + 월 사용 한도

AI Draft는 로그인 병원에서는 `/saas/ai/draft`를 사용하여 Worker에서도 플랜/월 한도를 확인합니다.

## 5. HQ 기능
- 승인 대기 병원 KPI 및 승인함
- 병원별 승인/거절/대기 상태
- 병원별 구독 플랜/상태/주기/가격 override/기간/메모
- 플랜별 월/연 가격
- 플랜별 기능 ON/OFF
- 직원/월 패키지/월 AI 한도
- 전체 구독 목록
- 가입 승인·구독·플랜 변경 Audit Log

## 6. 배포
1. GitHub Pages 루트 `index.html` 교체
2. GitHub Pages 루트 `hq.html` 교체
3. Cloudflare Worker 전체 코드를 `worker.js`로 교체 후 Deploy
4. 기존 D1 `DB`, Secrets/Variables는 그대로 유지
5. 새 Secret/새 D1은 필요 없음
6. Worker가 기존 D1에 v6.5 테이블/컬럼을 자동 추가합니다.

## 7. 결제 관련 범위
v6.5는 **SaaS 요금제/구독/권한 엔진과 HQ 구독 원장**까지입니다.
카드 결제, 자동 정기결제, 세금계산서, 결제사 webhook 연동은 아직 포함하지 않았습니다.
결제 연동 시 `clinic_subscriptions`를 결제사 상태와 동기화하면 됩니다.

## 8. 개인정보
기존 원칙을 유지합니다.
환자명 / 보호자명 / 진단·수술 케이스 / 개별 복약정보는 SaaS 계정·구독 DB 저장 대상이 아닙니다.
