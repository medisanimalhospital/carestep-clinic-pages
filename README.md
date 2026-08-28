# CARESTEP Clinic v6.5.3 · PRICING & 30-DAY TRIAL POLICY

## 변경사항
- PILOT: 30일 무료체험, 직원 5명, 월 패키지 500건. 신규 병원은 HQ 승인 시 trialing으로 자동 시작하고 종료일이 30일 뒤로 설정됩니다.
- BASIC: 월 49,000원 / 연 490,000원, 직원 3명, 월 패키지 500건.
- PRO: 월 99,000원 / 연 990,000원, 직원 15명, 월 패키지 3,000건, 운영 분석 + 일괄출력·ZIP.
- ENTERPRISE: 월 199,000원 / 연 1,990,000원 시작가, 직원/패키지 0=무제한, 별도계약 권장.
- AI Content Studio는 요금제 항목에서 제거했습니다. 일반 병원 기능이 아니라 CARESTEP 운영자 전용 기능으로 유지됩니다.
- HQ 기능명: Pilot 분석 → 운영 분석, 복수 ZIP·고급출력 → 일괄출력·ZIP.
- 구독 사용기한을 실제 접근제어에 반영합니다. trialing은 trial_ends_at/current_period_end, active는 current_period_end가 지나면 기능 접근이 제한됩니다.
- HQ 병원 상세에서 사용 시작일/종료일/체험 종료일을 확인·수정할 수 있습니다.
- 기존 승인 완료된 PILOT 병원에 30일 만료를 소급 적용하지 않습니다. 새 승인부터 자동 30일 체험이 적용됩니다.
- v6.5.2 빠른 교육자료 검색과 v6.5.1 운영자/카테고리 관리 기능을 유지합니다.

## 배포
1. GitHub Pages root의 index.html 교체
2. GitHub Pages root의 hq.html 교체
3. Cloudflare Worker 전체 코드를 worker.js로 교체 후 Deploy
4. 새 Secret / 새 D1 binding은 없습니다.
5. 기존 DB에 system_migrations 테이블이 자동 생성되고, 권장 요금제 기본값은 1회만 적용됩니다.

## 중요
현재 자동 카드결제/PG 결제는 연결되어 있지 않습니다. HQ가 구독 원장과 사용기한을 관리하며, 추후 결제사 웹훅과 연결할 수 있는 단계입니다.
