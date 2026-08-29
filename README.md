# CARESTEP Clinic v8.1 · Education → Patient Journey Auto Sync

v8.1은 v8.0.1의 전체 21종 Patient Journey Library와 데이터 기반 Follow-up CRM 위에 **교육자료 → Patient Journey 자동동기화**와 **병원 전용 Journey 삭제**를 추가합니다.

## 핵심 목적

앞으로 질환/수술 교육자료를 계속 추가할 때마다 Worker 코드를 수정해 Patient Journey를 별도로 추가하지 않아도 되도록 구조를 변경했습니다.

정식 흐름은 다음과 같습니다.

`Content Studio Draft → 수의사 검수 → Published → 중앙 콘텐츠 반영 → Journey 초안 자동생성 → HQ 검토 → Journey 게시 → 모든 병원에 자동 노출`

### 중요

로컬 Content Studio에서 `Published`만 한 상태가 아니라 **CARESTEP 중앙 콘텐츠에 반영된 Published 자료**가 자동동기화의 기준입니다. 여러 동물병원에 동일한 Journey를 배포하기 위한 중앙 Source of Truth 구조입니다.

## 1. 새 질환 자동 Journey 생성

중앙에 새 교육자료가 Published되면 Worker가 `profile.days`를 읽어 자동으로 Patient Journey 초안을 만듭니다.

자동 변환 항목:

- 교육자료 ID → Journey `sourceId`
- 자료명 → Journey 이름
- 카테고리 → Journey 카테고리
- `profile.days` → D+ 단계
- 각 D+ `title` → 단계 제목
- `focus` / `note` → 오늘의 핵심
- `items` → 확인 항목
- 교육자료 `redFlags` → 빠른 확인 기준
- 단계별 Calendar 업무 기본 ON
- 단계별 보호자 자동발송 기본 ON
- 기본 설문 시점 자동 추천

고양이/강아지 표기 또는 Content Studio species 값을 이용해 종도 자동 추정합니다.

## 2. 수의사 검토 후 게시

자동 생성된 Journey는 곧바로 병원에 노출되지 않습니다.

HQ → **Patient Journey Library · Auto Sync**에서:

- `자동초안`
- `업데이트 대기`

를 확인한 뒤 **검토 후 게시**를 눌러야 활성화됩니다.

게시 완료 후 병원은 `Journey 새로고침` 또는 다음 로그인 때 새 Journey를 자동으로 받습니다.

## 3. 교육자료 수정 시 기존 Journey 보호

이미 운영 중인 교육자료/Patient Journey를 수정해서 다시 중앙 Published하더라도 기존 Journey를 즉시 덮어쓰지 않습니다.

대신 HQ에:

`업데이트 대기`

상태로 새 제안본이 올라옵니다.

HQ 검토 후 게시할 때만 기존 Journey가 새 교육자료 기준으로 업데이트됩니다.

즉 환자 후속관리 중 콘텐츠가 예고 없이 바뀌는 것을 방지합니다.

## 4. Journey 고급 설정(선택)

Content Studio JSON에 `journey` 객체를 넣으면 자동생성 규칙을 세밀하게 조정할 수 있습니다. v8.1부터 Content Studio 저장/버전이력에서도 이 객체를 보존합니다.

예시:

```json
{
  "journey": {
    "enabled": true,
    "name": "고양이 FIC 장기관리 Journey",
    "species": "cat",
    "recommendedStages": [1, 3, 7, 30],
    "surveyDays": [3, 30],
    "calendarDays": [1, 3, 7, 30],
    "messageDays": [1, 3, 7]
  }
}
```

이 객체를 입력하지 않아도 `profile.days`를 기준으로 자동 생성됩니다.

## 5. 병원 전용 Journey 삭제

기본 CARESTEP Journey를 `병원용으로 복제`한 사본 또는 병원이 직접 만든 Journey는 이제:

- 편집
- 보관
- **삭제**

할 수 있습니다.

삭제 버튼은 Owner/Admin에게만 표시됩니다.

### 삭제 안전정책

삭제하면 병원 Journey Library에서는 즉시 사라지지만, 과거 설문/CRM/사용통계의 Journey 이름을 유지하기 위해 D1에서는 `status=deleted` 형태의 최소 템플릿 메타데이터를 남깁니다.

또한 Journey를 삭제해도 이미 SOLAPI에 접수된 예약문자나 Google/Outlook에 생성된 일정은 자동 취소하지 않습니다. 필요하면 기존 예약을 먼저 취소한 뒤 Journey를 삭제하세요.

CARESTEP 기본/자동 게시 System Journey는 병원에서 삭제할 수 없습니다.

## 6. 신규 D1 구조

Worker가 자동으로 생성합니다.

- `patient_journey_sync_queue`
  - 중앙 교육자료 ID
  - 연결 Journey ID
  - 교육자료 버전
  - 자동생성/업데이트 제안 상태
  - HQ 검토 상태

상태:

- `draft` : 새 Journey 자동초안
- `update_available` : 기존 Journey 업데이트 대기
- `published` : HQ 검토·게시 완료

**Cloudflare D1 Console에서 SQL을 직접 실행하지 않습니다.**

## 7. 개인정보

자동동기화에는 질환 교육자료와 Journey 운영 템플릿만 사용합니다.

다음 항목은 Auto Sync 테이블에 저장하지 않습니다.

- 환자명
- 보호자명
- 보호자 전화번호
- 퇴원일
- 환자별 투약 내용

## 8. 새 Worker Secret

없습니다.

기존 설정을 그대로 유지합니다.

- `SOLAPI_API_KEY`
- `SOLAPI_API_SECRET`
- `CARESTEP_HQ_KEY`
- 기존 CARESTEP 인증/AI Secret
- D1 binding `DB`

## 9. 배포 순서

1. Cloudflare Worker를 `worker-paste-ready.txt` 전체 내용으로 교체 → Deploy
2. 병원용 `index.html` 교체
3. HQ용 `hq.html` 교체
4. 브라우저 `Ctrl + Shift + R`
5. HQ → Patient Journey → `교육자료 동기화`
6. 자동초안/업데이트 대기 건 확인
7. 검토 후 게시
8. 병원 화면 → `Journey 새로고침`

## 10. 권장 첫 테스트

### Auto Sync

1. Content Studio에서 테스트용 새 질환 교육자료 작성
2. 수의사 검수 완료
3. Published + 중앙 반영
4. HQ → Patient Journey에서 `자동초안` 확인
5. D+ 단계 제목/내용 확인
6. `검토 후 게시`
7. 병원 화면 → Journey 새로고침
8. 새 질환 Journey 표시 확인

### 병원용 Journey 삭제

1. CARESTEP 기본 Journey 하나 선택
2. `병원용으로 복제`
3. 저장
4. `삭제`
5. 목록에서 사라지는지 확인
6. 기본 CARESTEP Journey는 그대로 남아 있는지 확인

## 검증

`TEST_REPORT.txt` 기준 33/33 PASS:

- Worker / 병원 화면 / HQ JavaScript 문법
- HTML ID 중복
- Auto Sync queue / migration / routes
- 신규 자동초안 병원 비노출
- HQ 게시 후 병원 노출 상태
- 기존 Journey 업데이트 보호
- 병원 사본 삭제 및 이력 메타데이터 보존
