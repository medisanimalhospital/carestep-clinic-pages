# CARESTEP HQ v9.2.1 — OAuth & Guardian Response Operations

Clinic v9.2 / Hotfix 32 이후의 실제 운영 구조에 맞춰 HQ를 동기화한 버전입니다.

## 핵심 변경

### 1. Go-Live Center: 7단계 → 6단계
현재 Clinic 시작 설정과 동일한 기준으로 변경했습니다.

1. 병원 정보
2. 직원 계정
3. 문자 자동발송
   - 병원별 SOLAPI OAuth2 연결
   - senderid:read 권한
   - 병원 ACTIVE 발신번호 선택
   - 테스트 문자 실제 전달 확인
4. Google Calendar
5. 첫 퇴원자료
6. 첫 후속관리

결제카드 등록은 PILOT Go-Live 필수조건에서 제외하고 체험/결제 상태로 별도 표시합니다.

### 2. SOLAPI OAuth 운영 상태
Go-Live Center에서 병원별로 다음을 구분합니다.

- OAuth 정상 / 미연결
- senderid:read 권한 재승인 필요
- OAuth 재연결 필요
- 발신번호 선택 필요
- 테스트 문자 완료 / 미완료
- 메시지 비용: 병원 직접결제 / Legacy CARESTEP / 미연결

신규 외부 병원은 OAuth2 셀프연동이 기본입니다.

### 3. Legacy 발신번호 관리 분리
기존 `발신번호 관리`는 `Legacy 발신번호`로 명확히 변경했습니다.
중앙 SOLAPI 계정에 위임 등록한 기존 병원만 이 화면을 사용합니다.

### 4. Guardian Response HQ 운영
기존 보호자 설문 화면을 v9.2 Guardian Response 기준으로 개선했습니다.

- 응답 초대
- 응답률
- 빠른 확인 필요(urgent)
- 확인 권장(watch)
- 미확인 응답
- 보호자 연락 요청
- 평균 만족도
- 병원별 / Journey별 집계

HQ에는 보호자 이름, 전화번호, 환자명, 자유 메모를 노출하지 않습니다.

### 5. HQ Dashboard 처리센터
`Guardian Response` 미확인 건수를 승인·처리 센터에 추가했습니다.
빠른 확인 필요 미처리 건이 있으면 HQ 상단 확인사항에 표시됩니다.

### 6. 병원 상세
병원 상세에서 바로 확인할 수 있습니다.

- Go-Live 6단계
- SOLAPI OAuth 연결
- 선택 발신번호
- 테스트 문자 수신 확인
- 메시지 비용 방식
- Guardian Response 응답률 / 빠른 확인 / 미확인 / 연락요청

## 배포 파일

- `hq.html` — 현재 HQ 파일 교체용
- `CARESTEP_HQ_v9.2.1_OAUTH_GUARDIAN_OPERATIONS_hq.html` — 버전명 포함 동일 HQ 파일
- `CARESTEP_v9.2.1_OAUTH_GUARDIAN_OPERATIONS_worker.txt` — Cloudflare Worker 교체용
- `D1_VERIFY_HQ_v9.2.1.sql` — 선택적 확인 쿼리

## 배포 순서

1. 현재 Cloudflare Worker 코드를 `CARESTEP_v9.2.1_OAUTH_GUARDIAN_OPERATIONS_worker.txt` 전체 내용으로 교체 후 Deploy
2. 기존 D1 binding / Secrets / Cron은 변경하지 않음
3. 현재 운영자 사이트의 `hq.html`을 이 패키지의 `hq.html`로 교체
4. GitHub Pages 배포 완료 후 HQ에서 `Ctrl + Shift + R`
5. HQ 연결 후 상단에 `HQ v9.2.1 연결됨` 확인
6. `개통 센터`에서 6단계 표시 확인
7. 외부 OAuth 병원이 `OAuth 정상`, `테스트 문자 수신 확인`, `병원 직접`으로 보이는지 확인
8. `Guardian Response` 메뉴에서 응답 집계 확인

Clinic의 `index.html`은 v9.2 그대로 사용하며 이번 HQ 업데이트 때문에 다시 교체할 필요가 없습니다.

## 인프라 변경

- 새 D1 DB: 없음
- 수동 D1 migration: 없음
- 새 Worker Secret: 없음
- SOLAPI OAuth Client ID/Secret: 기존 v9.1 설정 그대로
- CARESTEP_INTEGRATION_ENCRYPTION_KEY: 기존 값 그대로
- Google OAuth: 변경 없음
- Toss: 변경 없음

## QA

- Worker JavaScript syntax check PASS
- HQ JavaScript syntax check PASS
- HQ 정적 HTML ID 중복 0
- 기존 Worker 함수 누락 0
