# CARESTEP Clinic v6.6 · FOLLOW-UP AUTOMATION

## 핵심 변경
- 병원용 사이드바에 **후속관리** 메뉴 추가.
- 현재 케이스의 관리 시작일을 기준으로 D+1 / D+3 / D+7 / D+14 실제 날짜를 자동 계산합니다.
- 병원 기본 설정으로 후속관리 단계, 알림 시각, 캘린더 사전 알림을 저장하며 이 설정은 클라우드 동기화 가능합니다.
- 환자명·보호자명·퇴원일 등 환자별 정보는 CARESTEP 서버/localStorage의 후속관리 큐로 저장하지 않습니다.
- **후속관리 자동알림 .ics**: 선택한 D+ 일정들을 여러 VEVENT로 생성하고 각 일정에 VALARM을 포함합니다. 기본 캘린더 제목에는 환자명이 들어가지 않습니다.
- **D+ 발송문구 TXT**: D+ 단계별 카카오/문자 안내문을 자동 생성합니다.
- **후속관리 자동화 ZIP**: .ics + TXT + 선택 D+ 모바일 카드를 한 번에 묶습니다.
- 퇴원·후속관리 패키지에도 `후속관리 자동알림` 옵션이 추가됩니다.
- 플랜 기능 `followupAutomation` 추가: PILOT/PRO/ENTERPRISE ON, BASIC OFF 기본값. HQ 요금제 화면에서 변경 가능합니다.
- 기존 plan_catalog의 사용자 설정을 덮어쓰지 않고 새 기능 키만 1회 마이그레이션합니다.

## 자동화 범위
이 버전은 **캘린더 기반 자동 알림 + 발송문구 준비 자동화**입니다. 카카오/SMS를 CARESTEP 서버가 환자에게 직접 예약 발송하는 기능은 포함하지 않습니다. 직접 발송을 하려면 향후 메시징 사업자/API 연동과 개인정보 처리·동의 정책이 추가로 필요합니다.

## 배포
1. GitHub Pages root `index.html` 교체
2. GitHub Pages root `hq.html` 교체
3. Cloudflare Worker 전체 코드 교체 후 Deploy
4. 새 Secret / 새 D1 binding은 없습니다.
