# CARESTEP Clinic v6.6.2 · Follow-up Workboard

## 배포
이번 패치는 **병원용 `index.html`만 교체**합니다.

1. GitHub Pages 저장소 루트의 `index.html`을 이 파일로 교체
2. Commit
3. 배포 후 브라우저에서 Ctrl+F5

`hq.html`과 Cloudflare Worker는 v6.6.1 그대로 사용합니다. 새 Secret / D1 테이블 / Google OAuth Scope 추가가 없습니다. 기존 `calendar.events` scope로 동작합니다.

## 새 기능
- 후속관리 화면 상단에 **오늘 후속관리 업무** 보드 추가
- Google Calendar의 CARESTEP 일정만 브라우저에서 실시간 조회
- 오늘 / 미완료 / 완료 / 예정 필터
- 오늘 대상, 지난 미완료, 오늘 완료, 오늘 완료율 KPI
- 보호자 발송문구 복사
- `완료 처리` / `미완료로 되돌리기`
- Google Calendar 원본 일정 바로 열기
- 완료 상태와 완료자/완료시각은 Google Calendar event `extendedProperties.private`에 저장
- v6.6.1에서 이미 만든 CARESTEP 이벤트도 `carestep_status`가 없으면 미완료로 인식
- 같은 케이스 D+ 일정을 다시 등록해도 이미 완료된 상태를 보존

## 개인정보 설계
CARESTEP 서버, D1, localStorage에는 환자별 업무 큐나 후속관리 날짜를 저장하지 않습니다. 업무보드는 Google Calendar에서 브라우저로 직접 조회합니다. 캘린더 제목에 환자명을 포함하도록 설정한 경우 화면에는 표시되지만 CARESTEP 서버로 전송하지 않습니다.

## 현재 범위
업무보드는 **Google Calendar 연동 기준**입니다. Outlook은 기존 D+ 일정 직접 등록 기능을 계속 사용할 수 있지만, 완료/미완료 업무보드 조회는 아직 Google Calendar만 지원합니다.
