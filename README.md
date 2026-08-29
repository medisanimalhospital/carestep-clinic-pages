# CARESTEP Clinic v8.4 · Navigation & Information Architecture Refresh

v8.4는 기능을 추가하기보다 병원 직원이 자주 쓰는 화면을 더 빠르게 찾도록 좌측 메뉴 구조를 정돈한 UX 버전입니다.

## 핵심 변경

### 1. 주요 업무 3개를 최상위로 고정
- 자료실 · 교육자료 찾기
- 자료 생성 · 환자 맞춤 출력
- 후속관리 · Patient Journey / 문자 / 일정

각 메뉴에 아이콘과 짧은 설명을 추가해 처음 사용하는 직원도 기능 목적을 바로 이해하도록 했습니다.

### 2. HOME → 홈
기존 영문 `HOME`을 `홈`으로 통일하고 `오늘의 현황` 설명을 추가했습니다.

### 3. 저빈도 메뉴는 `관리 · 설정`으로 묶음
기본 상태에서는 접혀 있어 주요 업무에 시선이 집중됩니다.
- 병원 설정
- 계정 · 직원
- 문의 · 요청
- 운영 분석

관리 메뉴 안의 화면으로 직접 이동한 경우에는 그룹이 자동으로 펼쳐집니다.

### 4. 운영자 기능 분리
Operator unlock 시에만 기존처럼 아래 운영자 영역이 나타납니다.
- AI Content Studio
- 운영자 도구

### 5. 상단 정리
좌측 메뉴에 `병원 설정`이 명확해졌으므로 상단의 중복 `병원정보 수정` 버튼은 제거했습니다.
현재 케이스, 새 환자/초기화, 병원명, 계정 역할 표시는 유지합니다.

### 6. 기존 기능/권한 유지
v8.3의 Smart Case Flow, 자료생성→후속관리 자동 인계, 새 환자 초기화, Journey, SOLAPI, 설문, CRM, Google Calendar, 발신번호 관리 로직은 변경하지 않았습니다.
기존 Staff / Vet / Admin / Owner 권한도 그대로 유지합니다.

## 배포
1. Cloudflare Worker를 `worker-paste-ready.txt`로 교체 후 Deploy
2. 병원용 `index.html` 교체
3. HQ `hq.html` 교체
4. 브라우저 `Ctrl + Shift + R`

새 Secret이나 D1 SQL 작업은 없습니다.
