# CARESTEP Clinic v8.1.3 · SOLAPI 예약취소 신뢰성 Hotfix

## 수정 목적
v8.1.2에서 `예약 취소`를 누를 때 SOLAPI 예약 취소가 실제로 먼저 성공했더라도,
이어지는 그룹 삭제(clean-up)가 즉시 성공하지 않으면 CARESTEP이 전체 취소를 실패로 표시할 수 있었습니다.

SOLAPI의 예약 취소 API는 `DELETE /messages/v4/groups/:groupId/schedule`이며,
성공 시 예약이 해제되고 그룹은 `PENDING` 상태로 돌아갑니다.
그룹 삭제는 별도의 정리 작업이며 예약 취소 자체의 성공 조건이 아닙니다.

## v8.1.3 변경
- `SCHEDULED` 그룹:
  1. SOLAPI 예약 취소 API 실행
  2. 예약 취소 성공을 즉시 인정
  3. 그룹 삭제는 best-effort 정리로 처리
  4. 그룹 삭제가 늦거나 실패해도 CARESTEP에서는 `취소됨`으로 정상 반영
- `PENDING` 그룹:
  - 이전 버전에서 예약 취소는 성공했지만 로컬 상태 갱신이 실패한 건으로 간주 가능
  - 다시 `예약 취소`를 누르면 취소 완료 상태로 복구
- `SENDING / COMPLETE / FAILED` 등 이미 발송 단계에 들어간 그룹만 취소 불가
- 취소 성공 후 `last_error` 초기화
- 취소/교체/롤백의 provider status를 실제 결과(`PENDING` 또는 `DELETED`)로 저장
- 취소 버튼 클릭 중 `취소 중…` 표시 및 중복 클릭 방지
- 병원/HQ 표시 버전 `v8.1.3`으로 통일

## 기존 데이터
D1 스키마 변경은 없습니다.
기존 예약/실패/발송 기록은 그대로 유지됩니다.

v8.1.2에서 취소 시도 후 SOLAPI 쪽 그룹이 이미 `PENDING`으로 변했는데 CARESTEP에는 `예약접수`로 남은 건도,
v8.1.3 배포 후 같은 `예약 취소` 버튼을 다시 누르면 정상적으로 `취소됨`으로 복구됩니다.

## 배포
1. Cloudflare Worker를 `worker-paste-ready.txt` 전체 내용으로 교체
2. Deploy
3. 병원용 `index.html` 교체
4. HQ 사용 시 `hq.html` 교체
5. `Ctrl+Shift+R` 강력 새로고침
6. 화면 버전 `v8.1.3` 확인

새 Secret 및 D1 SQL 작업은 없습니다.

## 권장 테스트
1. 미래 시각 D+ 메시지 1건 예약
2. 최근 자동발송 예약에서 `SOLAPI SCHEDULED` 확인
3. `예약 취소`
4. CARESTEP 목록에서 해당 건이 취소 상태로 변경되고 취소 버튼이 사라지는지 확인
5. SOLAPI 콘솔에서 해당 그룹이 더 이상 예약 발송 대기 상태가 아닌지 확인
