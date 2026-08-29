# CARESTEP Clinic v6.9 · Sender Number Onboarding

## 핵심 변경

v6.9은 v6.8의 SOLAPI 자동발송/실패관리 위에 **동물병원 발신번호 온보딩 워크플로**를 추가합니다.

- 각 동물병원은 SOLAPI에 별도 가입하지 않음
- CARESTEP 중앙 SOLAPI 계정의 API Key/Secret을 Worker Secret으로 공용 사용
- 병원 Owner/Admin이 CARESTEP에서 대표 발신번호 인증 신청
- 신청 즉시 해당 병원의 자동발송은 승인 전까지 안전하게 OFF
- HQ에서 `SOLAPI 등록 확인 · 승인` 처리
- 승인 즉시 해당 발신번호가 병원 발송 설정에 고정되고 자동발송 ON
- 보완 요청/승인 해제 시 자동발송 OFF
- 승인 번호와 실제 발송 설정 번호가 불일치하면 발송 차단
- v6.8에서 이미 정상 사용 중이던 `enabled=1` 병원은 최초 마이그레이션 시 자동으로 `approved` 승계

## 병원 온보딩 흐름

1. 병원 가입 및 자동발송 가능한 요금제(PILOT/PRO/ENTERPRISE) 사용
2. 후속관리 → 병원 발송 연동 → 대표전화 입력
3. `발신번호 인증 신청`
4. 상태가 `승인 대기`로 변경되고 자동발송은 임시 OFF
5. CARESTEP 운영자가 SOLAPI 관리자에서 해당 발신번호 등록/인증
6. HQ → 외부 연동 설정 → 동물병원 발신번호 온보딩
7. 해당 병원에서 `SOLAPI 등록 확인 · 승인`
8. 병원 자동발송이 즉시 ON
9. 병원은 테스트 문자 후 D+ 자동예약 사용

## 중요 운영 원칙

HQ의 승인 버튼은 단순 내부 승인이 아니라 **해당 번호가 SOLAPI 측에서 실제 발신번호 등록/인증 완료됐음을 확인했다는 의미**입니다. 번호 사용 권한을 확인하기 전에 승인하지 마세요.

## 기존 병원 보호

v6.9 배포 직후 기존 v6.8 사용 병원이 갑자기 차단되지 않도록 최초 D1 마이그레이션에서:

- `clinic_messaging_settings.enabled = 1`
- `sender_number`가 존재하는 병원

은 `clinic_sender_onboarding.status = approved`로 자동 승계합니다.

## 신규 D1 테이블

`clinic_sender_onboarding`

저장 항목은 병원 발신번호, 신청/검토 상태와 시각, 운영 메모입니다. 보호자 전화번호나 메시지 본문은 저장하지 않습니다.

## 신규 API

병원:
- `POST /saas/messaging/sender-onboarding`

HQ:
- `GET /hq/messaging/sender-onboarding`
- `PATCH /hq/messaging/sender-onboarding/:clinicId`

## 배포 순서

1. Cloudflare Worker를 `worker-paste-ready.txt` 전체 내용으로 교체 후 Deploy
2. 별도 D1 SQL 실행 불필요 — Worker가 `CREATE TABLE IF NOT EXISTS` 및 승계 마이그레이션 수행
3. 새 Worker Secret 없음
4. 기존 `SOLAPI_API_KEY`, `SOLAPI_API_SECRET`, `CARESTEP_HQ_KEY` 유지
5. 병원용 `index.html` 교체
6. HQ용 `hq.html` 교체
7. 기존 병원에서 자동발송 상태가 계속 `승인 완료 / 준비됨`인지 확인
8. 신규 테스트 병원에서 발신번호 신청 → HQ 승인 → 자동발송 ON 흐름 확인

## 개인정보 구조

보호자 휴대전화와 발송문구는 기존과 동일하게 CARESTEP D1에 저장하지 않습니다. 예약 시 Worker를 거쳐 SOLAPI로 전달하고 CARESTEP에는 비식별 운영 메타데이터만 남깁니다.
