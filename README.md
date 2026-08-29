# CARESTEP Clinic v8.1.1 · Unified Category Sync & Version Label Fix

v8.1.1은 v8.1의 Education → Patient Journey Auto Sync 위에 **자료실과 후속관리의 카테고리 체계를 완전히 통일**하고, 병원 화면에 남아 있던 v7.3 표기 오류를 수정한 패치입니다.

## 1. 카테고리 단일 기준
CARESTEP은 다음 12개 카테고리 코드를 자료실과 Patient Journey에 공통으로 사용합니다.

- 수술 → 수술·회복
- 내과 → 내과·만성질환
- 안과 → 안과
- 피부과 → 피부·알레르기
- 비뇨기 → 비뇨기·신장
- 치과 → 치과·구강
- 신경 → 신경
- 종양 → 종양·항암
- 응급 → 응급·증상
- 노령 → 노령·삶의질
- 예방 → 예방·건강관리
- 기타 → 기타

기존 Journey의 `정형외과`, `일반외과`, `신경외과`, `내과·소화기` 같은 별도 분류는 자료실 카테고리로 자동 변환됩니다.

## 2. 기존 20개 교육자료와 Journey 정렬
기본 Journey는 연결된 자료실 콘텐츠의 카테고리를 따라갑니다. 예를 들어:

- 슬개골 / 십자인대 / 디스크 / 고관절 / 중성화 / 비장절제 / 방광결석 / 담낭 / 이물 → 수술·회복
- 췌장염 / 만성신장병 / 심장병 / 당뇨 → 내과·만성질환
- 경련·발작 / 구토·설사 → 응급·증상
- 유선종양 / 항암 → 종양·항암
- 스케일링·발치 / 고양이 전발치 → 치과·구강
- 인지기능저하 → 노령·삶의질
- 고양이 요도폐색·FLUTD → 비뇨기·신장

## 3. 앞으로 카테고리를 바꾸면 Journey도 같이 변경
자료실 운영자 카테고리 관리에서 질환 카테고리를 변경하면:

`자료실 카테고리 변경 → 연결된 CARESTEP 기본 Journey 카테고리 변경 → Auto Sync 검토 초안 카테고리 변경`

으로 즉시 동기화됩니다. 카테고리를 `원래 분류`로 되돌리는 경우도 Journey가 함께 복원됩니다.

새 Published 교육자료에서 자동 생성되는 Journey도 자료실의 **유효 카테고리(운영자 override 우선)** 를 사용합니다.

## 4. 병원 전용 Journey
병원 Journey 편집기의 카테고리는 자유입력 대신 자료실과 같은 12개 카테고리 선택형으로 변경했습니다. 기존 복제본은 원본 Journey 카테고리를 승계하며, 이전 버전에서 비표준 카테고리를 가진 병원 Journey는 배포 후 자동 정리됩니다.

## 5. v7.3 표기 오류 수정
병원용 `index.html`에 남아 있던 다음 v7.3 표기를 모두 v8.1.1로 수정했습니다.

- 브라우저 title
- 로그인/상단 CARESTEP SAAS 버전
- 메인 화면 버전
- 자동발송 설명
- Patient Journey / Follow-up CRM / 운영분석 버전
- ICS / Follow-up ZIP 내부 버전

HQ도 v8.1.1로 표시됩니다.

## 6. 기존 데이터 자동 마이그레이션
새 migration key: `v8.1.1_unified_library_journey_categories`

Worker가 최초 요청 시 기존 시스템 Journey와 병원 복제 Journey 카테고리를 자동 정리합니다. D1 Console에서 SQL을 직접 실행할 필요가 없습니다.

## 배포 순서
1. Cloudflare Worker → `worker-paste-ready.txt` 전체 교체 → Deploy
2. 병원용 `index.html` 교체
3. HQ용 `hq.html` 교체
4. 브라우저 `Ctrl + Shift + R`
5. CARESTEP 상단/브라우저 탭이 `v8.1.1`인지 확인
6. 자료실과 후속관리에서 같은 질환의 카테고리명이 동일한지 확인
7. 운영자 카테고리 1건 변경 → Journey 새로고침 → 동시 반영 확인

새 Worker Secret은 없으며 D1 SQL 직접 실행도 필요 없습니다.
