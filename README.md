# CARESTEP Clinic v6.6.1 · DIRECT CALENDAR SYNC

## 목적
v6.6의 `.ics` 후속관리 자동알림을 유지하면서, 병원 사용자가 CARESTEP에서 선택한 D+1 / D+3 / D+7 / D+14 일정을 Google Calendar 또는 Outlook Calendar의 기본 캘린더에 직접 생성할 수 있도록 확장한 버전입니다.

## 변경 파일
- `index.html` : 병원용 직접 캘린더 연결/등록 UI 및 OAuth 브라우저 연동
- `hq.html` : HQ → `연동 설정` 메뉴 추가
- `worker.js` : OAuth 공개 Client ID 중앙 설정 API 추가
- `schema_v661.sql` : 새 D1 설정 테이블 참고용

## 개인정보 원칙
- Google/Microsoft access token은 CARESTEP Worker/D1에 저장하지 않습니다.
- Google access token은 브라우저 메모리에만 유지합니다.
- Outlook/MSAL 인증 캐시는 브라우저 `sessionStorage`를 사용합니다.
- 환자명 포함은 기본 OFF입니다.
- 환자명 포함을 직접 켠 경우에도 환자명은 CARESTEP Worker로 전송하지 않고 브라우저에서 연결된 Google/Microsoft 캘린더 API로 직접 전송합니다.
- CARESTEP D1/localStorage에는 환자별 일정 큐를 만들지 않습니다.

## Google Calendar
1. Google Cloud 프로젝트 생성/선택
2. Google Calendar API 활성화
3. OAuth consent screen 설정
4. OAuth Client → Web application 생성
5. Authorized JavaScript origins에 `https://medisanimalhospital.github.io` 추가
6. 생성된 Client ID를 CARESTEP HQ → `연동 설정` → Google OAuth Client ID에 입력
7. 병원 페이지 → 후속관리 → `Google 연결`
8. `선택 일정 바로 등록`

요청 scope: `https://www.googleapis.com/auth/calendar.events`
대상: 로그인한 Google 계정의 `primary` calendar

## Outlook / Microsoft 365
1. Microsoft Entra admin center → App registrations → New registration
2. Supported account types를 운영 정책에 맞게 선택
3. Authentication → Add a platform → Single-page application
4. Redirect URI에 `https://medisanimalhospital.github.io/carestep-clinic-pages/` 추가
5. API permissions → Microsoft Graph → Delegated permissions → `Calendars.ReadWrite`
6. Application (client) ID를 CARESTEP HQ → `연동 설정`에 입력
7. Tenant는 여러 조직/개인 Microsoft 계정 지원 시 기본 `common`
8. 병원 페이지 → 후속관리 → `Outlook 연결`
9. `선택 일정 바로 등록`

Client Secret은 SPA에 넣지 않습니다.

## 중복 등록 방지
- Google: 현재 CARESTEP 케이스 세션별 비식별 event id를 사용하여 같은 탭/세션에서 재등록 시 갱신합니다.
- Outlook: `transactionId` + 현재 세션의 event id를 사용해 재등록 시 기존 event를 갱신합니다.
- 새 환자 시작 시 CARESTEP case nonce가 새로 생성됩니다. nonce에는 환자정보가 들어가지 않습니다.

## HQ 설정
새 메뉴: `연동 설정`
- Google OAuth Client ID
- Microsoft Application (client) ID
- Microsoft Tenant / Audience

이 값들은 공개 OAuth 식별자이며 D1 `integration_settings`에 저장됩니다. 새 Secret은 필요하지 않습니다.

## 배포
1. GitHub Pages 루트의 `index.html` 교체
2. `hq.html` 교체
3. Cloudflare Worker 전체 코드를 `worker.js` 내용으로 교체 후 Deploy
4. 기존 D1 `DB` binding 그대로 유지
5. HQ 접속 → `연동 설정` → Client ID 저장
6. 병원 페이지 Ctrl+F5

## 주의
현재 Outlook 브라우저 인증은 GitHub Pages 단일 HTML 구조를 유지하기 위해 Microsoft-hosted MSAL Browser v2 CDN을 사용합니다. 상용 출시 전 빌드/번들 구조로 전환할 때 최신 `@azure/msal-browser`를 패키지로 포함하는 것을 권장합니다.
