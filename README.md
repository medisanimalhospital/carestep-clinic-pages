# CARESTEP Clinic v6.2.1 · AI COST METER

v6.2 Direct AI Drafting에 생성별 API 비용 추적 기능을 추가한 업데이트입니다.

## 핵심 추가 기능

- AI Draft 성공 시 OpenAI API usage 토큰을 자동 기록
- 최근 생성 예상비용 표시
- 최근 생성 Input / Output token 표시
- 이번 달 AI 생성 건수
- 이번 달 누적 token
- 이번 달 예상 API 비용
- 이번 달 질환당 평균 예상비용
- 전체 누적 생성 건수 / 누적 예상비용
- 모델별 단가 표시
- Cached input token이 Worker 응답에 포함되면 할인 단가 반영
- 비용 기록 CSV 다운로드
- 비용 통계만 별도 초기화

## 개인정보 원칙

비용 통계 localStorage에는 다음만 저장됩니다.

- 생성 시각
- 사용 모델
- input token
- cached input token
- output token
- total token
- 예상 USD 비용
- 계산에 사용한 가격표 snapshot 날짜

질환명, 환자명, 보호자명, 복약정보, 퇴원정보는 AI 비용 통계에 저장하지 않습니다.

## GPT-5.6 가격표 snapshot

v6.2.1의 기본 가격표는 2026-07-30 이후 공개된 OpenAI API 가격을 사용합니다.

| Model | Input / 1M | Cached input / 1M | Output / 1M |
|---|---:|---:|---:|
| GPT-5.6 Terra | $2.00 | $0.20 | $12.00 |
| GPT-5.6 Luna | $0.20 | $0.02 | $1.20 |
| GPT-5.6 Sol | $5.00 | $0.50 | $30.00 |

화면의 금액은 token 기반 추정치입니다. 세금, 환율, 별도 tool 비용, 향후 가격 변경 등은 포함하지 않으므로 실제 비용은 OpenAI Usage/Billing을 확인하십시오.

## 업데이트 방법

### 1. GitHub Pages

기존 `index.html`을 이 폴더의 `index.html`로 교체하고 Commit 합니다.

### 2. Cloudflare Worker

기존 v6.2 Worker도 기본 input/output token을 반환하므로 Cost Meter는 동작합니다.
하지만 cached input token까지 반영하려면 이 폴더의 `worker.js`로 Worker 코드를 교체하고 Deploy 하는 것을 권장합니다.

환경변수/Secret은 기존 값을 그대로 사용합니다.

- OPENAI_API_KEY = Secret
- CARESTEP_ACCESS_KEY = Secret
- ALLOWED_ORIGINS = 기존 GitHub Pages origin
- OPENAI_MODEL = 기존 모델 (예: gpt-5.6-terra)

추가 환경변수는 없습니다.

## 정상 동작 확인

1. CARESTEP에서 AI Gateway 연결 테스트
2. 새 질환 기준 입력
3. `AI 초안 생성 → Draft`
4. 생성 완료 후 AI COST METER 확인
5. 최근 생성 비용, input/output token, 이번 달 합계가 증가하면 정상입니다.

기존 v6.2에서 이미 실행한 생성은 당시 브라우저에 토큰별 비용 기록을 저장하지 않았으므로 v6.2.1 설치 이전 사용량을 자동 복구하지는 않습니다.
