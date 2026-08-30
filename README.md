# CARESTEP Clinic v8.9 — Pilot & Customer Success

v8.9 adds an HQ Customer Success portfolio on top of v8.8.3.

## 핵심
- 병원별 **도입 점수 0~100** 자동 계산
- 병원별 **이탈 위험점수 0~100** 자동 계산
- 자동 판정: `도입 성공 / 양호 / 관찰 필요 / 위험 / 도입 초기`
- 무료체험 **7일 이하**, **3일 이하**, 체험 종료 + 카드 미등록 자동 탐지
- **7일/14일 미사용**, 실사용 이벤트 없음, 30일 퇴원자료 0건 자동 탐지
- Go-Live 완료율, 최근 30일 패키지·후속관리·활성일을 플랜 특성에 맞춰 반영
- `past_due / paused / canceled / 해지예약`을 이탈 위험에 반영
- 미확인 문자 실패도 운영 마찰 신호로 반영
- 병원별 **추천 Customer Success 액션** 자동 제안
- HQ Dashboard 승인·처리센터에 **고객 성공 위험** 카드 추가

## 도입 점수
BASIC처럼 후속관리 기능이 없는 플랜은 후속관리 미사용으로 감점하지 않습니다.
- 온보딩/Go-Live
- 최근 30일 퇴원자료 생성
- 최근 30일 활성일
- 후속관리 지원 플랜은 Patient Journey/후속관리 활용

## 이탈 위험
다음 신호를 조합합니다.
- 7일/14일 이상 미사용
- 체험 종료 임박 또는 종료 + 카드 미등록
- 가입 후 온보딩 지연
- 30일 퇴원자료 0건 / 실사용 이벤트 없음
- 결제 실패·일시중지·해지·해지 예약
- 미확인 문자 실패

규칙 기반 Customer Success 운영지표이며 의료 판단용 점수가 아닙니다.

## Privacy
HQ Customer Success에는 환자명, 보호자명, 전화번호, 복약정보, 메시지 본문을 사용하거나 표시하지 않습니다. 병원 단위 운영 메타데이터만 집계합니다.

## Deploy
1. `worker-paste-ready.txt` 전체로 Cloudflare Worker 교체 → Deploy
2. `index.html` 교체
3. `hq.html` 교체
4. Ctrl + Shift + R
5. HQ → `고객 성공` 메뉴 확인

새 Secret 없음 · 수동 D1 SQL 없음 · Cron 변경 없음 (`15 0 * * *` 유지).
