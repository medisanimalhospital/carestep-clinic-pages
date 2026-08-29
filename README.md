# CARESTEP Clinic v6.6.4 · Follow-up Outcome & Escalation

## 배포
이번 버전은 병원용 `index.html`만 변경합니다.

1. GitHub Pages 저장소 루트의 `index.html`을 이 버전으로 교체
2. Commit
3. CARESTEP 페이지에서 Ctrl+F5

`hq.html`, Cloudflare Worker, D1 schema, Secrets/Variables는 v6.6.3 그대로 사용합니다.

## 주요 변경

### 1) 연락 결과 기록
Google Calendar 후속관리 일정에서 다음 결과를 기록할 수 있습니다.
- 상태 양호
- 변화 없음
- 증상 지속
- 상태 악화
- 재진 예약
- 연락 안 됨
- 문자만 발송

`상태 악화`를 선택하면 `수의사 확인 필요`가 기본 활성화됩니다.

### 2) 수의사 Escalation
업무보드에 `🚨 확인 필요` 탭과 KPI가 추가됩니다.
- 확인 필요 상태는 완료된 연락 업무와 독립적으로 유지됩니다.
- 담당 수의사가 확인한 뒤 `수의사 확인 완료`로 닫을 수 있습니다.
- 확인자/확인시각이 Google Calendar event private properties에 기록됩니다.

### 3) 담당자 / 담당 수의사 배정
CARESTEP 병원 계정의 활성 직원 목록을 불러와 일정에 담당자를 배정할 수 있습니다.
- 내 업무
- 미배정
필터가 추가됩니다.

### 4) 연락 안 됨 → 재연락 일정
결과 입력 시 아래 재연락 예약을 선택할 수 있습니다.
- 1시간 후
- 3시간 후
- 내일 오전 10시

선택하면 원래 연락 시도는 완료 처리되고 동일한 CARESTEP 후속관리 Google Calendar에 새 미완료 재연락 일정이 생성됩니다.

### 5) 처리 메모
최대 500자 처리 메모를 입력할 수 있습니다.
이 메모는 CARESTEP D1이나 localStorage에 저장하지 않고 선택한 Google Calendar 일정의 `extendedProperties.private`에만 저장합니다.

## 업무보드 필터
- 오늘
- 🚨 확인 필요
- 내 업무
- 미배정
- 미완료
- 완료
- 예정

## 개인정보 원칙
CARESTEP D1에는 환자별 후속관리 결과, 담당자 배정, 처리 메모, 수의사 확인 상태를 새로 저장하지 않습니다.
이 정보는 병원이 선택한 Google Calendar 일정에 직접 저장됩니다.

Google Calendar에 환자명을 포함하는 옵션은 기존과 같이 기본 OFF입니다.

## 호환성
v6.6.1~v6.6.3에서 이미 만든 CARESTEP Google Calendar 일정도 그대로 읽습니다.
기존 일정은 결과 미입력 상태로 표시되고, v6.6.4에서 결과/담당자 정보를 추가할 수 있습니다.

같은 D+ 일정을 다시 동기화할 때 기존 완료/결과/담당자/수의사 확인 메타데이터를 보존하도록 Google event upsert 로직을 보강했습니다.
