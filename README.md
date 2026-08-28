# CARESTEP Clinic v6.4 · HQ ADMIN CONSOLE

## 새 파일
- `index.html`: 병원 사용자용 CARESTEP Clinic 최신본(v6.3.2.4 기능 유지)
- `hq.html`: CARESTEP 본사 운영자 전용 Admin Console
- `worker.js`: v6.4 HQ API가 추가된 Cloudflare Worker
- `schema_v64.sql`: D1 참고 스키마

## 이번 버전 핵심
- HQ Dashboard: 활성/중지 병원, 사용자, 30일 패키지, AI Draft/토큰/예상원가, 중앙 Published, Customer Voice, 오류 집계
- 병원 관리: 플랜(PILOT/BASIC/PRO/ENTERPRISE), 활성/중지, 직원/사용량/오류 확인
- 병원 중지 시 해당 병원 SaaS 세션 즉시 종료
- 중앙 콘텐츠: Published/Archived 전환 및 병원별 사용/숨김 + 병원 메모
- Customer Voice: 로그인 병원 요청에 `clinic_id` 자동 연결, HQ에서 상태 처리
- HQ Audit Log: 병원/콘텐츠/배포/요청 상태 변경 기록
- HQ는 별도 `CARESTEP_HQ_KEY` Secret으로 보호

## Cloudflare 새 Secret
Worker Settings → Variables and Secrets에 다음 Secret을 **새로 추가**하세요.
- Name: `CARESTEP_HQ_KEY`
- Type: Secret
- Value: 기존 키와 겹치지 않는 긴 랜덤 문자열

`CARESTEP_ACCESS_KEY`, `CARESTEP_ADMIN_KEY`, `OPENAI_API_KEY`와 서로 다른 값을 권장합니다.

## D1
새 D1 DB를 만들지 않습니다. 기존 `DB` binding을 그대로 사용합니다. Worker가 `hq_audit_events`를 만들고 기존 `customer_voice`에 `clinic_id`를 자동 추가합니다.

## 배포
1. GitHub Pages 루트의 `index.html` 교체
2. 같은 루트에 `hq.html` 추가
3. Cloudflare Worker 코드를 `worker.js` 전체로 교체 후 Deploy
4. Cloudflare에 `CARESTEP_HQ_KEY` Secret 추가
5. `https://<GitHub-Pages-Origin>/hq.html` 접속
6. Worker URL + CARESTEP_HQ_KEY 입력 → HQ 연결

## 보안
HQ Key는 `sessionStorage`에만 보관합니다. 브라우저를 닫으면 사라집니다. Pilot용 구조이며 상용화 전 HQ 전용 계정/MFA/Cloudflare Access 또는 Zero Trust/IP 정책/보안감사를 권장합니다.
