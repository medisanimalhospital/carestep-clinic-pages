# CARESTEP Clinic v7.2 · Disease-specific Patient Journey Builder

v7.2는 v7.0의 SaaS 사용량·과금·운영관리와 기존 Google Calendar / Outlook / SOLAPI 후속관리 흐름 위에 **질환·수술별 Patient Journey Builder**를 추가합니다.

> v7.1을 별도 배포하지 않고 진행한 버전입니다. v7.2 안에 Journey 운영에 필요한 최소 템플릿 편집·복제 기능을 함께 포함했습니다.

## 핵심 변경

### 1. 질환·수술별 Patient Journey Library
병원 후속관리 화면에서 질환/수술 Journey를 검색하고 현재 케이스에 적용할 수 있습니다.

기본 제공 Journey 9종:
- 슬개골 수술 회복 Journey
- 십자인대 수술 회복 Journey
- 중성화 수술 Journey
- 췌장염 퇴원관리 Journey
- 만성신장병 장기관리 Journey
- 심장병 진단 후 Journey
- 당뇨병·인슐린 관리 Journey
- 경련·발작 관리 Journey
- 고양이 요도폐색·FLUTD Journey

기본 Journey는 CARESTEP 공통 템플릿이며 병원에서 직접 덮어쓰지 않습니다.

### 2. D+1~365 자유 단계
기존 고정 D+1 / D+3 / D+7 / D+14 중심 구조를 확장했습니다.

Journey별 각 단계는 다음을 가질 수 있습니다.
- D+ 일수: 1~365
- 단계 제목
- 오늘의 핵심
- 확인 항목
- 빠른 확인이 필요한 변화
- Google/Outlook 캘린더 업무 생성 여부
- 보호자 자동발송 여부

Journey당 최대 12개 단계를 저장합니다.

### 3. 현재 케이스에 Journey 적용
Journey를 선택하고 `현재 케이스에 적용`을 누르면:
- 후속관리 D+ 단계가 해당 Journey 기준으로 전환
- 후속관리 카드 내용 자동 전환
- Google Calendar / Outlook 업무 내용 자동 전환
- SOLAPI 예약발송 문구 자동 전환
- .ics / 발송문구 TXT / 후속관리 ZIP 출력 자동 전환

현재 적용 중인 Journey ID만 브라우저 sessionStorage에 보관합니다. 환자명·보호자명·퇴원일 등 개별 케이스 정보는 Journey 템플릿 DB에 저장하지 않습니다.

### 4. 단계별 캘린더 / 보호자 발송 분리
예를 들어:
- D+1: 캘린더 + 문자
- D+3: 캘린더 + 문자
- D+14: 캘린더 + 문자
- D+30: 캘린더만

처럼 단계별 운영방식을 설정할 수 있습니다.

### 5. 병원 전용 Journey Builder
Owner/Admin은 CARESTEP 기본 Journey를 `병원용으로 복제`하거나 새 Journey를 만들 수 있습니다.

병원 전용 Journey에서:
- 이름 / 카테고리 / 대상 종
- 설명
- 단계 추가·삭제
- D+ 일수
- 핵심 안내
- 체크항목
- 빠른 확인 변화
- 캘린더 여부
- 자동발송 여부

를 수정할 수 있습니다.

병원 템플릿은 보관(archive) 처리할 수 있으며 CARESTEP 기본 Journey는 수정하지 않습니다.

### 6. Patient Journey + 기존 자동화 연결
기존 기능을 그대로 유지합니다.
- Google Calendar 직접 등록/업데이트
- Outlook Calendar 직접 등록/업데이트
- SOLAPI 예약발송
- 발송상태 동기화
- 실패 확인 / 안전 재발송
- 발신번호 온보딩
- 병원별 월 메시지 사용량 / 과금

Journey가 활성화된 경우 Calendar event private metadata에 비식별 Journey ID를 추가하고, SOLAPI 운영 메타데이터의 template ID도 `journey:<id>` 형식으로 연결합니다.

### 7. HQ Patient Journey Library
HQ에 `Patient Journey` 메뉴를 추가했습니다.

확인 항목:
- CARESTEP 기본 Journey 수
- 병원 커스텀 Journey 수
- 활성 Journey 수
- Journey 누적 적용 횟수
- 병원 / 카테고리 / 단계 구성

HQ의 적용 횟수는 비식별 SaaS 사용 이벤트를 집계합니다.

## 플랜 권한
기본 정책:
- PILOT: ON
- BASIC: OFF
- PRO: ON
- ENTERPRISE: ON

기능 키: `patientJourneyBuilder`

HQ의 `요금제 · 구독`에서 기존 feature 정책과 함께 관리할 수 있습니다.

## 개인정보 구조
`patient_journey_templates`에는 병원 공통 운영 템플릿만 저장합니다.

저장 가능한 항목:
- 병원 ID(병원 전용 Journey만)
- Journey 이름 / 카테고리 / 대상 종
- 단계 D+ 일수
- 일반 관리 안내 / 체크항목 / 위험신호 안내
- 캘린더/자동발송 사용 여부
- 버전 / 생성·수정 시각

Journey 템플릿에 저장하지 않는 항목:
- 환자명
- 보호자명
- 보호자 전화번호
- 퇴원일
- 개별 약·용량
- 환자별 진료/검사 결과

보호자 전화번호와 실제 예약발송 내용에 대한 기존 개인정보 최소화 구조도 그대로 유지합니다.

## 중요한 운영 안전사항
### Journey를 변경해도 기존 외부 예약은 자동취소하지 않습니다
이미 SOLAPI에 예약된 문자나 Google/Outlook에 만들어진 일정이 있는 상태에서 다른 Journey로 전환해도 기존 외부 예약을 자동 삭제하지 않습니다.

실제 케이스에서 Journey를 변경할 때는:
1. 기존 자동발송 예약 확인/취소
2. 기존 Calendar 일정 확인
3. 새 Journey 적용
4. 새 일정/문자 등록

순서로 운영하는 것을 권장합니다.

### 보호자 문자 예약은 기존 동의 절차 유지
Journey 적용만으로 보호자에게 문자가 즉시 예약되는 것은 아닙니다.
보호자 번호 입력 및 예약발송 동의 확인 후 `선택 D+ 자동발송 예약`을 실행해야 합니다.

### 임상 검토
기본 Journey는 병원 운영을 돕는 일반 템플릿입니다. 개별 환자의 진단·처방·재검 간격·응급 기준을 대체하지 않습니다. 실제 병원 배포 전 병원의 진료 프로토콜에 맞게 수의사가 검토하고 필요하면 병원 전용 Journey로 복제해 조정하세요.

## D1 자동 마이그레이션
새 테이블:
- `patient_journey_templates`

새 migration key:
- `v7.2_patient_journey_builder`

Worker 최초 요청 시:
- 테이블/인덱스 생성
- 플랜에 `patientJourneyBuilder` 기능 키 보완
- CARESTEP 기본 Journey 9종 seed

을 자동 수행합니다.

**Cloudflare D1 Console에서 SQL을 직접 실행할 필요가 없습니다.**

## 새 Worker Secret
없습니다.

기존 설정을 그대로 유지합니다.
- `SOLAPI_API_KEY`
- `SOLAPI_API_SECRET`
- `CARESTEP_HQ_KEY`
- 기존 CARESTEP 인증/AI Secret
- D1 binding `DB`

## 배포 파일
- `worker.js`
- `worker-paste-ready.txt`
- `index.html`
- `hq.html`
- `README.md`
- `DEPLOY_CHECKLIST.txt`
- `TEST_REPORT.txt`

## 배포 순서
1. Cloudflare Worker를 `worker-paste-ready.txt` 전체 내용으로 교체 → Deploy
2. 병원용 `index.html` 교체
3. HQ용 `hq.html` 교체
4. 브라우저 강력 새로고침 (`Ctrl+Shift+R`)
5. 후속관리 → Patient Journey Builder 확인
6. 테스트 Journey 적용 후 Calendar/문자 단계 표시 확인
7. HQ → Patient Journey 메뉴 확인

## 권장 첫 실전 테스트
1. 테스트 케이스에서 관리 시작일 입력
2. `슬개골 수술 회복 Journey` 선택
3. `현재 케이스에 적용`
4. D+1 / 3 / 7 / 14 / 30 단계 표시 확인
5. D+30이 캘린더 전용으로 표시되는지 확인
6. 테스트용 Google Calendar로 일정 등록
7. D+1 메시지 1건을 가까운 미래로 예약해 SOLAPI 연동 확인
8. 실제 환자 적용 전 병원 프로토콜에 맞게 Journey 내용 검토
