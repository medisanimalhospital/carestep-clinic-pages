# CARESTEP Clinic v8.7 — Monthly Operations Report

## 목적
병원이 CARESTEP의 운영 가치를 월별 숫자로 확인할 수 있도록 **퇴원 패키지 → 후속관리 → 자동발송 → 보호자 설문 → 위험신호**를 하나의 비식별 월간 리포트로 묶은 버전입니다.

## 병원 화면
`관리 · 설정 → 운영 리포트`에서 월을 선택하면 병원 전체 서버 데이터를 집계합니다.

주요 지표:
- 퇴원 패키지 생성 건수 및 평균 생성시간
- 후속관리 비식별 케이스 수
- 후속관리 등록 업무 대비 결과입력 완료율
- 보호자 문자 발송 성공률
- 설문 응답률 / 평균 만족도
- 주의·악화 기반 위험 케이스 발견 건수
- 문자 + 설문응답 + 결과입력을 합친 보호자 관리 접점
- 지난달 대비 변화
- 많이 사용한 교육자료
- Journey별 완료·호전·악화·수의사 확인
- 직원별 패키지/후속관리 활용 현황

`월간 리포트 PDF`, `CSV 저장`을 지원합니다.

## HQ
`사용량 · 정산`의 기존 월 선택값과 동일한 기준으로 **병원별 월간 CARESTEP 성과** 표를 추가했습니다. HQ는 병원 단위 집계만 보고 개별 case key나 보호자 응답 원문은 보지 않습니다.

## 데이터 기준
- 월 경계는 **Asia/Seoul (KST)** 기준입니다.
- 퇴원 패키지 수는 고유 환자 수가 아니라 **패키지 생성 이벤트 수**입니다.
- 후속관리 케이스는 메시지/설문/CRM에 존재하는 비식별 `case_key`의 월별 distinct 수입니다.
- 후속관리 완료율은 해당 월 Google Calendar 직접등록 업무 수 대비 CRM 결과입력 완료 업무 수입니다. 재동기화나 월 경계가 있는 경우 운영 참고값으로 해석합니다.
- v8.7부터 후속관리 가이드 최종 완료 시 `followup_setup_complete` 사용 이벤트를 추가해 향후 분석 품질을 높입니다.

## 개인정보
월간 리포트 API와 HQ 월간 성과에는 환자명, 보호자명, 보호자 전화번호, 복약정보를 반환하지 않습니다. 기존 CARESTEP 개인정보 최소화 원칙을 유지합니다.

## 기존 기능 유지
- v8.6.1 Toss 정기결제 및 TEST ONLY 결제 실패/복구 테스트
- Founding Clinic 가격보장
- SOLAPI 자동발송 / 발신번호 onboarding
- Google Calendar / Outlook
- Patient Journey Auto Sync
- 보호자 설문 / Follow-up CRM
- Action Center / HQ 승인센터

## 배포
1. `worker-paste-ready.txt` 전체로 Cloudflare Worker 교체 → Deploy
2. 병원용 `index.html` 교체
3. HQ용 `hq.html` 교체
4. 브라우저 `Ctrl + Shift + R`
5. `/hq/health`가 `8.7`인지 확인
6. 병원 → 운영 리포트에서 현재 월 조회
7. HQ → 사용량 · 정산에서 동일 월의 병원별 성과 확인

새 Secret 없음 / 새 D1 SQL 직접 실행 없음 / 기존 Cron `15 0 * * *` 유지.
