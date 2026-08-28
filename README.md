# CARESTEP Clinic v6.2 · DIRECT AI DRAFTING

## 핵심 변경

v6.1.1의 Content Studio / Quality Gate / 수의사 검수 / 버전관리 / 카테고리 필터를 유지하면서, AI 초안 생성 단계를 직접 연결했습니다.

새 흐름:

1. 질환명, 대상, 카테고리, 사용상황, 관리기간, 관리포인트, 위험신호를 입력
2. `AI 초안 생성 → Draft` 클릭
3. GitHub Pages → Cloudflare Worker → OpenAI Responses API
4. JSON Schema에 맞는 CARESTEP 콘텐츠 생성
5. CARESTEP 로컬 Schema 재검사
6. Draft 자동 저장
7. Quality Gate + 수의사 검수
8. Published

AI 결과가 바로 Published 되는 경로는 없습니다.

## 파일

- `index.html` : GitHub Pages용 CARESTEP 전체 앱
- `worker.js` : Cloudflare Worker용 AI Gateway
- `wrangler.jsonc` : Wrangler 배포 설정 예시
- `.gitignore` : API 키/로컬 Secret 커밋 방지
- `SETUP_AI_GATEWAY.md` : Worker 연결 순서

## 보안 구조

### 브라우저에 저장하지 않는 것
- OpenAI API Key
- 환자명 / 보호자명
- 환자별 처방 및 복약정보

### 브라우저에 저장되는 것
- Worker Endpoint URL: localStorage
- CARESTEP Access Key: sessionStorage (브라우저 세션 종료 시 삭제)
- 기존 Content Studio 자료 및 검수정보: 기존과 동일한 localStorage

### 서버 Secret
- `OPENAI_API_KEY`
- `CARESTEP_ACCESS_KEY`

`CARESTEP_ACCESS_KEY`는 OpenAI API Key가 아닙니다. 공개된 Worker를 임의 호출하는 것을 줄이기 위한 Pilot용 접근키입니다. 정식 SaaS에서는 병원 로그인/JWT 기반 인증으로 교체해야 합니다.

## AI 모델

기본 설정은 `gpt-5.6-terra`입니다. `OPENAI_MODEL` Worker 변수로 변경할 수 있습니다.

## 의료 콘텐츠 안전장치

Direct AI 생성 후에도 기존 v6.1 Quality Gate를 그대로 거칩니다.

Publish 조건:
- CARESTEP Schema 통과
- Quality 80점 이상
- Critical 위험표현 0건
- 수의사 검수 체크리스트 6개 완료
- 최종 수의사 승인

Published 콘텐츠를 수정하면 다시 Review 상태로 내려갑니다.

## GitHub Pages 업데이트

기존 Pages 저장소의 루트 `index.html`만 v6.2 파일로 교체하고 Commit 하면 됩니다.

Worker 파일은 GitHub Pages에 실행되는 파일이 아니며 별도 Cloudflare Worker에 배포해야 합니다.
