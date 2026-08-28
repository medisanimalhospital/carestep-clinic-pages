# CARESTEP Clinic v6.0 · AI CONTENT STUDIO

## 목적
질환이 늘어날 때 모든 보호자용 문구를 처음부터 직접 작성하지 않고, **AI 초안 → 구조검사 → 수의사 검수 → Publish** 흐름으로 콘텐츠를 확장하기 위한 GitHub Pages 호환 버전입니다.

## 핵심 기능
- 새 질환 기본정보 입력
- CARESTEP 전용 AI 생성 프롬프트 자동 작성/복사
- AI JSON 결과 붙여넣기 및 Schema 검증
- 로컬 구조 초안(비-AI) 생성
- 기존 20종/Published 자료 복제
- Draft / Review / Published / Archived 상태
- 버전 자동 증가
- 수의사 검수 체크 없이는 Publish 불가
- Published 자료는 즉시 기존 자료실/퇴원 Wizard에서 사용
- Content Studio JSON 백업/복원

## 중요한 구조
현재 GitHub Pages는 정적 사이트이므로 OpenAI API 키를 브라우저 코드에 넣지 않았습니다. 브라우저에 키를 넣으면 공개될 수 있습니다. 따라서 v6.0은 프롬프트 복사 → AI JSON 생성 → 붙여넣기 방식입니다. 향후 서버(Supabase Edge Function/Cloudflare Worker 등)를 붙이면 같은 Schema를 그대로 사용해 `AI 초안 생성` 버튼을 직접 API 호출로 변경할 수 있습니다.

## 개인정보
Content Studio에는 질환 교육 콘텐츠와 병원 설정만 저장합니다. 환자명, 보호자명, 개별 복약정보는 Content Studio에 저장하지 않습니다.

## 배포
GitHub Pages 저장소의 최상위 `index.html`을 이 파일로 교체하고 Commit 하면 됩니다.
