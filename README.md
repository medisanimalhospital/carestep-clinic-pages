# CARESTEP Clinic v8.4.2 · Content Delete + Journey Status Workflow

## 목적
v8.4.1의 Daily Workflow UX를 유지하면서 운영자가 테스트/오작성 질환을 안전하게 정리하고, HQ Patient Journey 승인 작업을 상태별로 빠르게 분류할 수 있게 개선한 버전입니다.

## 1. Content Studio 질환 삭제

### Draft / Review
- 중앙 Published 이력이 없는 Draft/Review는 Content Library의 `삭제` 버튼으로 바로 제거할 수 있습니다.
- 이전에 Published된 자료를 수정해서 Review 상태가 된 경우에는 중앙의 기존 Published 버전이 남아 있을 수 있으므로 `게시 중단 → 삭제` 2단계를 적용합니다.

### Published
- Published 상태에서는 바로 삭제할 수 없습니다.
- 먼저 `게시 중단`을 눌러 Archived로 변경합니다.
- 중앙 Published 자료인 경우 중앙 Master Content도 Archived가 되고 연결된 자동생성 System Journey도 `archived`로 비활성화됩니다.
- 다시 게시하면 연결 Journey도 기존 검토 상태에 맞춰 복원됩니다.

### Archived → 삭제
- `삭제`를 누르면 로컬 Content Studio 목록에서 제거됩니다.
- 중앙에 존재했던 자료는 중앙 상태가 `deleted`로 변경됩니다.
- 해당 교육자료에서 자동 생성된 System Patient Journey도 `deleted` 처리되어 병원 Journey Library에서 제거됩니다.
- 과거 CRM·설문·사용 통계 참조를 깨뜨리지 않기 위해 D1의 최소 메타데이터/이력은 보존합니다.
- 병원이 별도로 복제해 편집한 커스텀 Journey는 독립 자료이므로 자동 삭제하지 않습니다.

## 2. HQ 중앙 콘텐츠에서도 삭제 가능
- `중앙 콘텐츠`에서 Published → `게시 중단`
- Archived 상태 → `삭제`
- 삭제 시 연결된 자동생성 System Journey도 함께 제거됩니다.
- Deleted 자료는 이력 확인용으로 중앙 콘텐츠 상태 필터에서 확인할 수 있으나 바로 재게시 버튼은 제공하지 않습니다. 다시 사용하려면 Content Studio에서 검수 후 다시 Publish하는 흐름을 권장합니다.

## 3. HQ Patient Journey 상태 빠른 분류
Patient Journey 화면에 상태 필터와 빠른 분류 버튼을 추가했습니다.

- 전체
- 승인 필요
- 사용 중
- 자동초안
- 업데이트 대기
- 보관·비활성

상태 의미:
- `사용 중`: 현재 병원에서 사용 가능한 active Journey
- `승인 대기`: 신규 교육자료에서 생성된 Journey 초안
- `업데이트 승인`: 기존 active Journey는 유지되지만 새 교육자료 버전 검토가 필요한 상태
- `보관·비활성`: 교육자료 게시가 중단되어 현재 신규 적용에서 제외되는 Journey

승인 필요 항목은 목록 상단에 자동 정렬됩니다. `초안 승인`과 `업데이트 승인` 버튼을 구분해 표시합니다.

## 4. 삭제/게시중단 안전정책
- Published 질환은 2단계(게시 중단 → 삭제)로 처리합니다.
- 중앙 콘텐츠 변경/삭제는 `CARESTEP_ADMIN_KEY`가 필요한 운영자 작업입니다.
- 환자명, 보호자명, 전화번호, 복약정보는 이 기능에서 저장하거나 처리하지 않습니다.
- 기존 예약문자·과거 설문·CRM 기록은 콘텐츠 삭제로 함께 삭제하지 않습니다.

## 5. 배포
1. `worker-paste-ready.txt` 전체 내용으로 Cloudflare Worker 교체 → Deploy
2. `index.html` 교체
3. `hq.html` 교체
4. `Ctrl + Shift + R`

새 Secret 및 수동 D1 SQL 작업은 없습니다.
