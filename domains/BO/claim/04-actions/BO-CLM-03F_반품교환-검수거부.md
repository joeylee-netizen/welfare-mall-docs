---
id: "BO-CLM-03F"
title: "반품/교환 검수거부"
type: action
domain: claim
status: draft
version: "1.0"
created: 2026-04-22
updated: 2026-04-22
author: "기획자"
refs:
  - "BO-CLM-003"
  - "BO-CLM-004"
  - "BO-CLM-010"
  - "BO-CLM-023"
  - "BO-CLM-024"
  - "BO-CLM-025"
  - "BO-CLM-026"
tags: [클레임, 반품, 교환, 검수, 검수거부, 재배송, 철회, API]
trigger: submit
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/inspection/reject"
---

# 반품/교환 검수거부

## 트리거 조건

- 반품/교환 상세 모달의 `검수거부` 버튼 → 거부 사유/메모 입력 모달
- 대상 상태: `RETURN_COLLECTED` 또는 `EXCHANGE_COLLECTED`

반품·교환 공용. **거부 시 원상품 재배송** 및 **교환의 경우 재고 복원** 부수 효과 발생.

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| inspection_result | string | Y | 거부 사유 enum (`FAILED_DEFECT` / `FAILED_MISSING` / `FAILED_OTHER`) |
| inspection_memo | string | C | 검수 메모 — `FAILED_OTHER`인 경우 필수(10~1000자) |
| redelivery_carrier_code | string | N | 재배송 택배사 코드(원상품 반송용, 선택) |

> `PASSED`는 본 액션으로 처리하지 않음. 반품은 [BO-CLM-03D](./BO-CLM-03D_반품-환불승인.md), 교환은 [BO-CLM-03E](./BO-CLM-03E_교환-최종승인.md) 호출.

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_type | string | `RETURN` \| `EXCHANGE` |
| claim_status_bo | string | `RETURN_WITHDRAWN` 또는 `EXCHANGE_WITHDRAWN` |
| claim_status_fo | string | 동일 |
| inspection_result | string | 거부 사유 enum |
| inspection_by_id | number | 검수자 ID |
| inspection_at | datetime | 검수 일시 |
| redelivery_triggered | boolean | 원상품 재배송 트리거 여부 |
| inventory_restored | boolean | **교환 전용.** 재고 복원 실행 여부 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_INSPECTION_RESULT | `PASSED` 또는 허용 범위 외 값 |
| 400 | MEMO_REQUIRED | `FAILED_OTHER`인데 메모 누락 |
| 400 | MEMO_LENGTH | 메모 길이(10~1000자) 위반 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `RETURN_COLLECTED` / `EXCHANGE_COLLECTED` 외 상태 |
| 500 | INVENTORY_RESTORE_FAILED | 교환 재고 복원 실패 (롤백) |
| 500 | REDELIVERY_FAILED | 재배송 예약 실패 (Claim 상태는 전환, 재배송 재시도 큐 등록) |
| 500 | INTERNAL_ERROR | 서버 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "검수가 거부 처리되었습니다. 원상품이 재배송됩니다."
2. 해당 행 상태 `RETURN_WITHDRAWN` / `EXCHANGE_WITHDRAWN`으로 갱신 + 검수 결과 badge 갱신
3. 상세 재조회 → 섹션 4(검수 정보)에 거부 사유·메모 반영, 종결 상태로 버튼 영역 비활성
4. **원상품 재배송 트리거:** 판매사 창고에서 구매자 배송지로 재배송 예약(배송비는 구매자 귀책 처리)
5. **교환 전용:** 선차감된 재고 자동 복원
6. ClaimHistory에 `(COLLECTED → WITHDRAWN)` + 검수자·사유·메모 기록
7. FO 구매자에게 검수거부·재배송 안내 알림톡·메일 발송(사유 포함)

### 실패 시

1. 오류 토스트
2. **500 INVENTORY_RESTORE_FAILED (교환):** 전체 트랜잭션 롤백, CS 알림
3. **500 REDELIVERY_FAILED:** Claim 상태는 `WITHDRAWN`으로 전환되나 재배송 실패 플래그 + 재시도 큐 등록. 관리자에게 경고 토스트.
4. 409 상태 충돌 시 상세 새로고침

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = RETURN_COLLECTED` 또는 `EXCHANGE_COLLECTED`
- 거부 사유 Radio 필수
- `FAILED_OTHER` 선택 시 메모 10~1000자 필수
- 확인 모달: "검수거부 시 반품/교환이 철회되며 원상품이 구매자에게 재배송됩니다."

### 백엔드

- 상태 검증
- `inspection_result` 화이트리스트 검증(`FAILED_*`만 허용, `PASSED`는 거부)
- 메모 조건부 필수 검증
- 트랜잭션 처리:
  - Claim 상태 업데이트 + 검수 결과 기록 + ClaimHistory 기록
  - **교환:** 재고 복원 포함 원자 처리
  - 재배송 트리거(비동기, 실패 시 재시도 큐)
- 관리자 권한 범위 내 `client_id` 검증
- 알림 발송(비동기)
