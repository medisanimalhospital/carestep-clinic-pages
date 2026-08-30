# CARESTEP v9.0.2 — Production Pilot Audit / Hotfix 15

검수 대상
- Clinic: v9.0.2 Hotfix 14 기반 `index.html`
- Backend: v9.0.2 Cloudflare Worker
- HQ: v9.0.2 `hq.html` (변경 없음)
- Public pilot URL 확인: `https://medisanimalhospital.github.io/carestep-clinic-pages/`

## 결론

**Controlled Pilot: CONDITIONAL GO**

3–10개 정도의 신뢰된 동물병원 파일럿에는 GitHub Pages를 그대로 사용할 수 있습니다. 다만 아래 `파일럿 배포 전 필수` 항목을 완료한 뒤 배포하는 것을 권장합니다. 불특정 다수를 대상으로 한 정식 공개/유료 상용화는 아직 `NO-GO`입니다.

## Hotfix 15에서 즉시 보완한 항목

1. **로그인 세션 토큰 저장 강화**
   - 기존: SaaS bearer token을 `localStorage`에 저장
   - 변경: `sessionStorage`에만 저장
   - 기존 브라우저의 legacy localStorage token은 한 번 읽어 sessionStorage로 이동한 뒤 삭제
   - 브라우저를 완전히 닫으면 로그인 토큰이 브라우저 저장소에서 사라짐

2. **병원별 로컬 설정 분리**
   - 기존: `carestep_hospital`, `carestep_workflow`가 브라우저 전체에서 공통 키
   - 변경: 로그인 상태에서는 clinic ID별 키로 분리
   - 같은 사용자가 여러 병원을 전환해도 병원 A 설정과 병원 B 설정이 섞이지 않도록 함
   - 기존 로컬 설정은 병원명이 현재 로그인 병원과 일치할 때만 안전하게 1회 마이그레이션
   - 병원별 기존 클라우드 설정이 있으면 해당 병원 상태로 hydrate

3. **비밀번호 해시 강화**
   - PBKDF2-HMAC-SHA256 반복 횟수: 100,000 → 600,000
   - 기존 100,000회 해시는 로그인 성공 시 자동으로 600,000회 해시로 재저장
   - 기존 계정/비밀번호 호환 유지

## PASS — 구조적으로 확인된 항목

- HTTPS GitHub Pages 앱 정상 공개
- 프런트/HQ/Worker에서 실제 Secret 값 하드코딩 흔적 없음
- Worker CORS가 `ALLOWED_ORIGINS` allowlist를 검사
- HQ API는 별도 `CARESTEP_HQ_KEY`로 분리
- SaaS 로그인은 Bearer session token 기반
- 서버는 session token 원문 대신 SHA-256 hash를 D1에 저장
- 로그인 실패 제한: 15분 창에서 5회 실패 시 15분 잠금
- 병원 가입은 `pending` 상태로 생성되고 HQ 승인 전 로그인 차단
- Owner/Admin/Vet/Staff 역할별 권한 분리
- 병원 상태/멤버십 상태/가입 승인 상태가 세션 검증에 포함
- 병원 설정, 직원, 메시지, CRM, Journey 등의 핵심 SaaS 쿼리가 세션 `clinic_id`를 기준으로 제한
- 환자명·보호자명·복약정보 등 patient case data는 서버 cloud-state/usage log 저장에서 차단
- 후속관리 메시지의 보호자 전화번호/본문은 CARESTEP D1에 저장하지 않고 provider(SOLAPI)로 전달
- Toss billingKey는 Worker에서 암호화 후 저장하도록 구성
- 구성 백업/복구, 계정 탈퇴, 병원 데이터 삭제 요청/유예/실행 구조 존재
- HQ 감사로그와 장애로그 구조 존재
- Hotfix 15 index inline JS syntax PASS
- Hotfix 15 Worker JS syntax PASS
- HQ inline JS syntax PASS
- index duplicate static HTML ID: 0
- HQ duplicate static HTML ID: 0
- hardcoded-secret-like assignment scan: 0

## 파일럿 배포 전 필수

### 1. Worker API URL 기본값 내장 — 현재 남은 UX blocker
새 브라우저에서 공개 CARESTEP 주소를 열면 로그인 화면에 `CARESTEP Worker API URL` 입력란이 노출됩니다.

외부 병원에는 GitHub Pages 주소 하나만 보내는 것이 좋으므로 실제 운영 Worker URL을 `index.html`의 기본 endpoint로 내장하고 일반 사용자 화면에서는 기술 입력란을 숨기는 것을 권장합니다.

- Worker URL은 Secret이 아니므로 프런트 기본값으로 넣을 수 있음
- 현재 검수 자료에는 실제 운영 `*.workers.dev` 주소가 포함되어 있지 않아 자동 반영하지 못함

### 2. 이용약관/개인정보 처리방침 실제 운영자 정보 입력
현재 문서에는 다음 항목이 아직 “상용 출시 전에 확정” 문구로 남아 있습니다.
- 실제 사업자/운영자 정보
- 서비스 문의 채널
- 개인정보 보호책임자/연락처
- 처리위탁/국외이전의 실제 계약 기준 고지

외부 병원의 이름/이메일 계정을 실제 수집하는 시점에는 최소한 실제 운영자·문의·개인정보 연락처를 확정해 표시하는 것을 권장합니다.

### 3. HQ → Production Launch 확인
실제 Worker 환경에서 다음을 확인합니다.
- `ALLOWED_ORIGINS` = 현재 GitHub Pages origin, HTTPS only, wildcard 없음
- CARESTEP_ACCESS_KEY / ADMIN_KEY / HQ_KEY 설정
- Billing encryption key 설정
- SOLAPI/OpenAI/Toss 상태
- 데이터 삭제/복구/감사로그/보안점검 수동 체크

무료 PILOT만 운영하고 실제 결제를 받지 않는 동안은 Toss LIVE 전환은 파일럿 시작의 필수 조건이 아닙니다. 실제 카드/자동결제를 받기 전에는 Live 계약·실결제·취소 검증이 필요합니다.

### 4. 실제 런타임 smoke test
배포 후 테스트 병원 2개(A/B)를 만들어 아래를 실제 Worker/D1에서 확인합니다.
- A Owner 로그인 / B Owner 로그인
- A의 직원 목록·병원 설정·Journey·메시지 job이 B에서 보이지 않음
- B의 데이터가 A에서 보이지 않음
- 같은 브라우저에서 A↔B 병원 전환 시 로컬 병원정보/프리셋이 섞이지 않음
- 로그아웃 후 browser restart 시 재로그인 필요
- Google Calendar 연결 및 테스트 일정 1건
- SOLAPI 테스트 문자 1건 + 상태 확인
- 탈퇴/세션 강제 종료 테스트
- 실제 결제를 파일럿에서 사용한다면 Toss TEST → LIVE 별도 검증

## 파일럿에서 허용 가능하지만 정식 상용화 전 강화 권장

- HQ는 현재 공유 `CARESTEP_HQ_KEY` 구조: 파일럿에는 가능하나 상용화 전 Cloudflare Access/Zero Trust 또는 개별 관리자 계정+MFA 권장
- 공개 가입 API에 별도 IP rate limit이 없음: 소수 controlled pilot에는 가능하나 공개 확장 전 Cloudflare Rate Limiting/WAF 권장
- GitHub Pages는 보안 헤더를 세밀하게 제어하기 어렵고 앱이 inline JS + 외부 CDN을 사용: 정식 서비스 시 Cloudflare Pages/Netlify/custom domain + CSP/보안 헤더 검토 권장
- Google/Microsoft/SOLAPI/Toss/OpenAI 등 외부 processor/subprocessor에 대한 실제 고지/계약 흐름 최종 검토

## 파일럿 운영 권장 방식

1. Hotfix 15 배포
2. Worker 기본 URL을 앱에 넣어 사용자에게 기술 설정을 요구하지 않기
3. 운영자/개인정보 연락처 확정
4. 테스트 병원 2개로 tenant isolation smoke test
5. 처음 1–2일은 가상 환자/테스트 번호 사용
6. 이상 없으면 3–10개 병원에 Controlled Pilot 배포
7. 피드백/장애 로그를 확인하며 1–2주 운영
8. 이후 custom domain 및 상용 보안/법무 마감

## 배포 파일

Hotfix 15는 **index + Worker를 함께 배포해야 합니다.**

- `index.html`: 세션/병원별 로컬 저장소 hardening
- `worker-paste-ready.txt`: PBKDF2 600k + legacy hash 자동 upgrade
- `hq.html`: 기존 v9.0.2와 동일, 변경 없음

새 D1 SQL, 새 Secret, 새 Cron은 필요하지 않습니다.
