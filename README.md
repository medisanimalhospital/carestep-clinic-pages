# CARESTEP Clinic v6.1 · Content Quality & Governance

## 목적
v6.0 AI Content Studio의 Draft → Review → Published 흐름에 의료 콘텐츠 품질관리와 검수 이력을 추가한 버전입니다.

## 주요 기능
- Quality Gate 100점 점검
  - 구조 완성도 25
  - 안전표현 25
  - 보호자 실행가능성 25
  - 후속관리/전달정보 25
- 위험표현 자동 탐지
  - 약 임의 중단 지시
  - 용량 증량/감량 직접 지시
  - 진단 단정 표현 등
- 모호표현 경고
- 수의사 검수 체크리스트 6종
- 검수자 기록
- Published 시 재검토 주기 6/12/18/24개월 설정
- REVIEW DUE / 재검토 임박 표시
- 버전 snapshot 및 JSON field 단위 변경점 비교
- 수정/Review/Publish/Archive 감사 이력
- 기존 Published 자료 수정 시 자동으로 Review 상태로 전환하여 재검수 전 공개 차단
- Content Studio JSON 백업/복원에 governance 정보 포함

## Publish 조건
다음을 모두 만족해야 Published 가능합니다.
1. CARESTEP Schema 필수 구조 통과
2. Quality Gate 80점 이상
3. Critical 위험표현 0건
4. 수의사 검수 체크리스트 6항목 모두 완료
5. 최종 수의사 검수 완료 체크

> Quality Gate는 의학적 정확성을 자동 판정하지 않습니다. 문서 구조, 위험표현, 누락을 찾는 보조 도구이며 최종 의학적 검수는 수의사가 수행해야 합니다.

## GitHub Pages 업데이트
기존 저장소의 최상위 `index.html`을 이 버전으로 교체하고 Commit합니다.
GitHub Pages가 갱신된 뒤 브라우저에서 강력 새로고침(Ctrl+F5)을 권장합니다.

정상 버전 표시:
`CARESTEP Clinic MVP · v6.1 · CONTENT QUALITY & GOVERNANCE · VET GATE`

## 데이터 저장
Content Studio의 Draft/버전/검수 이력은 현재 브라우저 localStorage에 저장됩니다.
환자명, 보호자명, 처방약, 퇴원일 등 환자별 개인정보 비저장 원칙은 기존과 동일합니다.
정식 SaaS 단계에서는 중앙 DB, 사용자 권한, 서버 감사로그로 이전하는 것을 권장합니다.
