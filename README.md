# CARESTEP Clinic v8.2.1 · Sender Number Management UX

v8.2 Guided Follow-up Workflow 위에 HQ 발신번호 운영 화면을 독립 메뉴로 분리하고, 병원이 늘어나도 찾기 쉬운 검색·상태 필터·등록 진행 상태를 추가한 운영 개선 버전입니다.

## 핵심 변경
- HQ 사이드바에 `발신번호 관리` 독립 메뉴 추가
- `승인 대기 / SOLAPI 등록중 / 승인 완료 / 보완 요청 / 전체 신청` KPI
- 병원명·발신번호 검색
- 상태 필터 및 자동발송 ON/OFF 필터
- `SOLAPI 등록 시작` 단계 추가
- 등록이 끝난 뒤에만 `SOLAPI 등록 확인 · 승인`
- 승인 전/보완 상태에서는 자동발송 OFF 유지
- 병원 화면에도 `SOLAPI 등록 진행 중` 상태 표시
- 기존 승인 병원은 그대로 유지

## 권장 운영 흐름
1. 병원이 CARESTEP에서 대표번호 신청
2. HQ → 발신번호 관리에서 `SOLAPI 등록 시작`
3. 중앙 SOLAPI 관리자에서 번호 소유·사용 권한 확인 및 발신번호 등록
4. SOLAPI 등록 완료 후 HQ에서 `SOLAPI 등록 확인 · 승인`
5. 병원 자동발송 ON

## 배포
1. Worker 교체 후 Deploy
2. index.html 교체
3. hq.html 교체
4. Ctrl+Shift+R

새 Secret 및 D1 수동 SQL은 필요하지 않습니다. `clinic_sender_onboarding.status` 기존 TEXT 필드에 `registering` 상태를 사용합니다.
