# CARESTEP Clinic v9.0.2 — Daily Workflow & UX Polish

v9.0.2 is a usability release on top of v9.0.1. It does not add a new business domain or permission model. Instead it applies seven UX improvements across the clinic and HQ experiences while preserving billing, messaging, Customer Success, Production Launch, legal/privacy, backup/deletion, and security behavior.

## 1. Clinic home — today-first workflow

The clinic home now emphasizes the work that should be handled now:
- 오늘 후속관리
- 확인 필요
- 문자 실패
- 새 환자 자료 만들기

The cards link directly to the relevant workflow. Existing performance/statistics remain available as secondary information rather than competing with the daily action area.

## 2. Role-aware account simplification

The existing Owner/Admin/Vet/Staff model is retained. No additional permission granularity was introduced.
- Owner/Admin: all account tabs, including 직원 관리 and 요금제 · 결제
- Vet/Staff: the management-only 직원 관리 and 요금제 · 결제 tabs are hidden, while 내 계정, 보안 · 데이터, 약관 · 탈퇴 remain available
- Clinic data deletion remains Owner-only

## 3. Unified user-facing status language

The hospital-facing UI uses a smaller, consistent Korean status vocabulary such as:
- 정상
- 확인 필요
- 처리 중
- 완료
- 조치 필요
- 중지

Messaging, sender onboarding, and billing states are mapped into the same mental model while preserving backend status values.

## 4. Actionable empty states

Empty screens now suggest the next useful action instead of only reporting that data is absent. Examples include:
- first employee invite
- first configuration backup
- card registration
- filter reset
- follow-up setup

HQ empty results also include reset/recovery actions where appropriate.

## 5. Current case context

A compact current-case bar remains visible while the user moves between discharge material creation and follow-up work. It shows the current browser-session case context and offers:
- 케이스 계속
- 새 환자

No new patient-context D1 endpoint or patient PII persistence was added.

## 6. HQ priority workflow

Customer Success now supports one-click operational filters:
- 오늘 처리
- 위험 병원
- 체험 종료 임박
- 결제 문제
- 14일 미사용

Default priority sorting emphasizes payment failure, trial expiry, long inactivity, onboarding delay, and then risk score.

## 7. Clear action feedback

Important actions now provide in-place feedback such as:
- 저장 중… → ✓ 저장됨
- 백업 중… → ✓ 백업 완료
- 결제 처리 중… → ✓ 결제 완료

This is applied to key clinic and HQ operations while keeping the existing toast/error flow.

## Regression / backend impact

- Existing v9.0.1 static control IDs preserved
- Existing HQ 5-group navigation and quick clinic search preserved
- Enterprise billing preserved
- message overage auto-settlement preserved
- Founding policy preserved
- real-time message test preserved
- Customer Success preserved
- Production Launch / legal / backup / deletion preserved
- No new D1 table or column
- No new Secret
- No Cron change
- Worker business logic is unchanged from v9.0.1 except version metadata alignment (`9.0.2`)

## Validation

Static/structural regression suite: **120/120 PASS**
- Worker JavaScript syntax PASS
- Clinic inline JavaScript syntax PASS
- HQ inline JavaScript syntax PASS
- no duplicate static HTML IDs
- all v9.0.1 static IDs preserved
- all seven UX improvement groups verified
- critical billing/messaging/governance regression markers verified

External provider/runtime transactions are not executed by this offline static validation and should be smoke-tested after deployment.

## Deployment

1. Deploy `worker-paste-ready.txt` to Cloudflare Worker.
2. Confirm `/hq/health` reports `9.0.2`.
3. Replace GitHub Pages `index.html`.
4. Replace `hq.html`.
5. Wait for Pages deployment and hard refresh (`Ctrl+Shift+R`).
6. Follow `DEPLOY_CHECKLIST.txt`.
