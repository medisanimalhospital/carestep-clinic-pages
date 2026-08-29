# CARESTEP Clinic v8.8 — Clinic Onboarding & Go-Live Center

v8.8은 신규 병원이 가입 후 별도 설명 없이 CARESTEP을 실제 진료 흐름에서 사용할 수 있도록 **셀프 온보딩 + HQ Go-Live Center**를 추가한 버전입니다.

## 병원용 시작 설정
Owner/Admin 첫 로그인 시 아직 개통이 완료되지 않았다면 `관리 · 설정 → 시작 설정`을 자동으로 안내합니다. 이후에는 홈에서도 진행률 배너를 계속 확인할 수 있습니다.

Go-Live 7단계:
1. 병원 정보 — 병원명 + 대표전화 저장
2. 직원 계정 — 활성 직원 2명 이상 또는 `나중에` 선택
3. 발신번호 — HQ 승인 완료
4. Google Calendar — OAuth 연결 후 임시 테스트 일정 생성/삭제로 쓰기 권한 확인
5. 문자 발송 — SOLAPI 테스트 문자 1건 성공
6. 첫 퇴원자료 — `package` 사용 이벤트 1건 이상
7. 후속관리 자동화 — `followup_setup_complete` 또는 Calendar 직접 동기화 1건 이상

결제카드 등록은 PILOT 동안 선택사항으로 별도 표시하고 유료 전환 전에 확인하도록 안내합니다.

## 기존 병원 자동 인정
v8.8 이전에 완료한 작업도 가능한 범위에서 그대로 인정합니다.
- 클라우드 병원 설정에 병원명/대표전화가 있으면 병원정보 완료
- 활성 직원 수를 직원 단계에 반영
- 기존 발신번호 승인 상태 반영
- 기존 Calendar 직접 동기화 이력 반영
- 기존 SOLAPI 테스트 발송 audit 반영
- 기존 퇴원 패키지 생성/후속관리 설정 사용이력 반영
- 기존 Toss 등록 카드 반영

## Google Calendar 연결 확인
온보딩의 `Google 연결 확인`은 단순 로그인 여부만 보지 않습니다. 선택한 Calendar에 약 10분 뒤의 `CARESTEP 연결 테스트` 임시 일정을 1건 만든 뒤 즉시 삭제하여 **실제 쓰기 권한**까지 확인합니다. 성공한 뒤에는 비식별 완료 플래그만 D1에 저장합니다.

## HQ Go-Live Center
HQ 메뉴에 `개통 센터`가 추가됩니다.
- 병원별 0~7 진행률
- 7단계 개별 완료 상태
- 발신번호 상태
- 카드 등록 여부
- 무료체험 남은 일수
- 최근 사용일
- Go-Live / 설정 중 / 확인 필요

자동 확인 사유:
- 가입 후 3일 이상 + 발신번호 미완료
- 가입 후 3일 이상 + Calendar 미연결
- 가입 후 3일 이상 + 첫 퇴원자료 미생성
- 가입 후 3일 이상 + 첫 후속관리 미완료
- 무료체험 7일 이하 + 카드 미등록
- 무료체험 3일 이하 + 카드 미등록
- 14일 이상 사용 없음

HQ Dashboard `승인 · 처리 센터`에도 `개통 지연` 카드가 추가되어 바로 Go-Live Center로 이동합니다.

## 개인정보
온보딩 상태에는 환자명, 보호자명, 보호자 전화번호, 복약정보를 저장하지 않습니다. 단계 완료 여부와 기존 비식별 사용 이벤트, 연동 상태만 사용합니다.

## 배포
1. `worker-paste-ready.txt` 전체를 Cloudflare Worker에 교체 후 Deploy
2. `/hq/health`에서 `version = 8.8` 확인
3. 병원 `index.html` 교체
4. HQ `hq.html` 교체
5. GitHub Pages 반영 후 `Ctrl + Shift + R`
6. 병원 Owner/Admin 로그인 → `시작 설정` 확인
7. HQ → `개통 센터` 확인

## 인프라 변경
- 새 Worker Secret: 없음
- 수동 D1 SQL: 없음 — Worker가 `clinic_onboarding_state`를 자동 생성
- Cron 변경: 없음 — 기존 `15 0 * * *` 유지
- Toss / SOLAPI / Google OAuth 기존 설정 그대로 유지

## 유지되는 기능
v8.7 월간 운영 리포트, v8.6.1 Toss 결제 실패/복구 테스트, 자동결제, SOLAPI 발신번호/예약문자, Google Calendar 직접연동, Patient Journey, 보호자 설문, Follow-up CRM, Action Center 기능을 그대로 유지합니다.
