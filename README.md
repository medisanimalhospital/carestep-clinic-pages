# CARESTEP Clinic v6.7 · Automated Guardian Messaging

## 핵심 변경

v6.7은 기존 Google Calendar 후속관리 흐름에 **SOLAPI 기반 보호자 예약 자동발송**을 추가합니다.

- D+1 / D+3 / D+7 / D+14 문자(SMS/LMS) 예약발송
- 카카오 알림톡(ATA) 예약발송
- 알림톡 실패 시 문자 대체발송 옵션
- 병원별 발신번호 / 카카오 pfId / 승인 템플릿 ID 설정
- 병원 PC나 CARESTEP 페이지가 꺼져 있어도 SOLAPI가 예약 시각에 발송
- 예약 취소
- SOLAPI 그룹 상태 동기화(예약 / 발송중 / 완료 / 실패 / 취소)
- 동일 케이스·D+ 단계 재예약 시 기존 미래 예약 교체
- 요금제 권한 `automatedMessaging`
  - PILOT: ON
  - BASIC: OFF
  - PRO: ON
  - ENTERPRISE: ON

## 개인정보 구조

이번 버전은 CARESTEP 자체 대기열에서 보호자 전화번호를 보관하지 않습니다.

예약 시:

`브라우저 → CARESTEP Worker → SOLAPI 예약발송 API`

CARESTEP D1에는 다음만 저장합니다.

- 병원 ID
- D+ 단계
- 예약 시각
- 채널(SMS/Kakao)
- 콘텐츠 template ID
- 비식별 case key
- SOLAPI group ID / message ID
- SOLAPI 상태
- 동의 확인 시각
- 생성 사용자 / 생성 시각

**보호자 전화번호와 메시지 본문은 CARESTEP D1에 저장하지 않습니다.**
전화번호와 발송 내용은 실제 메시지 제공자인 SOLAPI로 전송됩니다.

## 왜 Cloudflare Cron을 사용하지 않나

SOLAPI 메시지 발송 API가 `scheduledDate` 예약발송을 지원하므로, CARESTEP Worker가 개인정보가 포함된 대기열을 유지할 필요가 없습니다.

따라서 v6.7 최종 구조에서는:

- Cloudflare Cron Trigger 불필요
- `FOLLOWUP_ENCRYPTION_KEY` 불필요
- CARESTEP D1 전화번호 암호문 저장 불필요

## 배포 파일

- `index.html` : 병원용 CARESTEP
- `hq.html` : CARESTEP HQ 관리자
- `worker.js` : Cloudflare Worker
- `worker-paste-ready.txt` : Worker 전체 복붙용

## 배포 순서

### 1. GitHub Pages

저장소 루트에서 아래 2개를 교체합니다.

- `index.html`
- `hq.html`

교체 후 Commit 합니다.

### 2. Cloudflare Worker

Worker → Edit code → 기존 코드 전체 삭제 → `worker-paste-ready.txt` 전체 붙여넣기 → Deploy

### 3. 기존 환경변수/Secret 유지

기존 항목은 그대로 유지합니다.

- `OPENAI_API_KEY`
- `CARESTEP_ACCESS_KEY`
- `CARESTEP_ADMIN_KEY`
- `CARESTEP_HQ_KEY`
- `ALLOWED_ORIGINS`
- `OPENAI_MODEL`
- D1 binding `DB`

### 4. 새 Worker Secrets 추가

Cloudflare Worker의 Settings → Variables and Secrets에서 아래 2개를 **Secret**으로 추가합니다.

- `SOLAPI_API_KEY`
- `SOLAPI_API_SECRET`

`FOLLOWUP_ENCRYPTION_KEY`는 만들 필요가 없습니다.
Cloudflare Cron Trigger도 만들 필요가 없습니다.

## SOLAPI 사전 설정

SOLAPI에서 아래 준비가 필요합니다.

### 문자 사용

1. SOLAPI 계정 생성
2. API Key / API Secret 발급
3. 병원 발신번호 사전 등록
4. Worker Secret에 API Key / Secret 입력
5. CARESTEP 병원 페이지 → 후속관리 → 병원 발송 연동에서 등록 발신번호 입력
6. 테스트 1건 발송

### 카카오 알림톡 사용

문자 설정에 더해 아래가 필요합니다.

1. SOLAPI에 카카오 비즈니스 채널 연결
2. 알림톡 템플릿 등록 및 승인
3. 병원 발송 연동에 `pfId` 입력
4. 승인된 `templateId` 입력
5. 알림톡 실패 시 문자 대체발송 사용 여부 선택

CARESTEP 권장 알림톡 템플릿 변수:

- `#{병원명}`
- `#{단계}`
- `#{주제}`
- `#{핵심}`
- `#{안내}`

실제 SOLAPI 승인 템플릿의 변수명과 정확히 일치해야 합니다.

## 실제 사용 흐름

1. 퇴원 케이스에서 교육자료와 관리 시작일 설정
2. 후속관리에서 D+1 / 3 / 7 / 14 단계와 발송 시각 확인
3. 보호자 휴대전화 입력
4. 발송 동의 확인 체크
5. 문자 또는 알림톡 선택
6. `선택 D+ 자동발송 예약` 클릭
7. CARESTEP Worker가 즉시 SOLAPI 예약발송 API에 각 D+ 메시지를 접수
8. SOLAPI가 지정 날짜/시간에 자동 발송
9. CARESTEP `예약내역 새로고침`에서 provider 상태 확인

## 예약 취소

아직 SOLAPI에서 `SCHEDULED/PENDING` 상태인 예약은 CARESTEP에서 `예약 취소`할 수 있습니다.
CARESTEP은 SOLAPI 예약 취소 후 해당 그룹도 삭제하여 발송되지 않도록 처리합니다.
이미 발송 단계에 들어간 그룹은 취소할 수 없습니다.

## 실패 메시지

보호자 전화번호와 메시지 본문을 CARESTEP이 보관하지 않는 구조이므로, 이미 Provider에서 실패한 메시지를 CARESTEP이 전화번호 없이 임의 재전송하지 않습니다.
필요하면 보호자 번호를 다시 입력하고 새 예약을 생성합니다.

## 상용화 전 운영 주의

- 보호자 연락처 처리 및 메시지 발송에 관한 병원 개인정보 안내/동의 절차를 마련하세요.
- SOLAPI가 실제 메시지 처리 사업자로 보호자 연락처와 발송 내용을 처리한다는 점을 개인정보 처리방침/위탁 또는 관련 고지에 반영할 필요가 있습니다.
- 카카오 알림톡은 승인된 정보성 템플릿을 사용하세요.
- 광고성 메시지를 발송할 경우 정보성 후속관리 메시지와 별도의 광고성 정보 전송 관련 동의/표시 기준을 검토하세요.
- 실제 상용화 전 테스트 번호로 문자 → 알림톡 → 예약 → 취소 → 발송완료 상태까지 순서대로 검증하는 것을 권장합니다.

## 데이터베이스

새 D1 Binding은 필요하지 않습니다. 기존 `DB`를 사용합니다.
Worker가 아래 테이블을 자동 생성/보완합니다.

- `clinic_messaging_settings`
- `followup_message_jobs`
- `followup_message_audit`

이전에 시험용 v6.7 대기열 스키마가 생성된 경우에도 필요한 provider-side scheduling 컬럼을 자동 추가하며, 신규 예약에는 기존 개인정보 암호화 컬럼을 사용하지 않습니다.
