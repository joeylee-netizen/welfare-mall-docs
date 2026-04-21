---
id: "BO-CLM-039"
title: "반품/교환 접수"
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
  - "BO-CLM-025"
tags: [클레임, 반품, 교환, 접수, 신청, API]
trigger: submit
method: POST
endpoint: "/api/v1/claims/{claim_type}"
---

# 반품/교환 접수

## 트리거 조건

- 주문관리 모듈의 주문상세 화면에서 **"반품 신청 대행" / "교환 신청 대행" 버튼** 클릭 시 BO/PO 대행 접수
- FO 구매자가 마이페이지에서 직접 반품/교환 신청 시 (FO 호출)
- 대상 주문상태: **배송완료** (또는 교환배송완료) ~ 구매확정 이전, 8일 이내

**엔드포인트 분기:**

- 반품: `POST /api/v1/claims/return`
- 교환: `POST /api/v1/claims/exchange`

`{claim_type}` path parameter로 구분. 내부 로직은 공용이되 교환 고유 필드(`exchange_target_option_id`) 검증 로직이 추가된다.

---

## 사전 호출

- 주문상세 조회: 대상 상품주문번호·옵션·수량·가격 확인
- 중복 클레임 진행 여부 검증(동일 옵션 진행 중 불가)
- 교환의 경우: 교환 대상 옵션 조회(동일 상품 ID 내 옵션 리스트), 재고 확인

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_type | string | Y | `return` \| `exchange` |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| order_id | number | Y | 원주문 ID |
| shipment_id | number | Y | 출고지 ID |
| items | array | Y | 대상 상품 목록 |
| items[].order_item_id | number | Y | 원주문 상품주문번호 |
| items[].quantity | number | Y | 대상 수량(≥1, 원주문 수량 이하) |
| reason_code | string | Y | 클레임 사유 ([ClaimReasonCode](../02-data/BO-CLM-010_클레임-엔티티.md#ClaimReasonCode)) |
| reason_detail | string | C | 사유 상세(`OTHER` 시 필수, 10~500자) |
| applied_channel | string | Y | `FO` (구매자 직접) \| `BO_PROXY` (BO/PO 대행) |
| memo | string | N | 접수 메모(0~500자) — CS 경로 요약 등 |
| exchange_target_option_id | number | C | **교환 전용 필수**. 교환 대상 옵션 ID |
| buyer_shipping_fee_payment | object | C | 구매자 귀책 배송비 결제 정보(카드 결제 시 필수) |
| buyer_shipping_fee_payment.card_token | string | C | 카드 결제 토큰 |
| buyer_shipping_fee_payment.amount | number | C | 결제 금액 |

---

## 응답 (Response)

### 성공 (201 Created)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 생성된 클레임 ID |
| claim_code | string | 발번된 클레임 코드 (`CLM{YYYYMMDD}{6자리}`) |
| claim_type | string | `RETURN` \| `EXCHANGE` |
| claim_sub_type | string | `RETURN_BY_USER`/`RETURN_BY_ADMIN`/`EXCHANGE_BY_USER`/`EXCHANGE_BY_ADMIN` |
| claim_status_bo | string | `RETURN_REQUESTED` 또는 `EXCHANGE_REQUESTED` |
| claim_status_fo | string | `RETURN_REQUESTED` 또는 `EXCHANGE_REQUESTED` |
| process_channel | string | `FO` 또는 `BO_PROXY` |
| liability_party | string | `BUYER` 또는 `SELLER` (사유 코드 기반 자동 결정) |
| refund_total_amount | number | 환불 예정 총액(반품) / 0(교환, 동일 가격) |
| shipping_fee_charged | number | 구매자 부담 배송비 |
| exchange_target_option_name | string | 교환 전용. 교환 대상 옵션명 |
| exchange_inventory_reserved | boolean | 교환 전용. 재고 선차감 여부 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 파라미터 누락/형식 오류 |
| 400 | INVALID_REASON | 클레임 유형에 허용되지 않는 사유 코드 |
| 400 | REASON_DETAIL_REQUIRED | `reason_code=OTHER`인데 상세 누락 |
| 400 | EXCHANGE_TARGET_REQUIRED | 교환인데 `exchange_target_option_id` 누락 |
| 400 | EXCHANGE_DIFFERENT_PRODUCT | 교환 대상 옵션의 상품 ID가 원주문 상품과 다름 |
| 400 | EXCHANGE_DIFFERENT_PRICE | 교환 대상 옵션의 가격이 원주문 옵션과 다름 |
| 400 | SHIPPING_FEE_PAYMENT_REQUIRED | 구매자 귀책인데 배송비 결제 정보 누락 |
| 403 | FORBIDDEN | 권한 없음(타 고객사 주문) |
| 404 | ORDER_NOT_FOUND | 존재하지 않는 주문 |
| 409 | ORDER_STATUS_INVALID | 배송완료·교환배송완료 외 상태 |
| 409 | PERIOD_EXPIRED | 가능 기간(8일) 경과 |
| 409 | CLAIM_IN_PROGRESS | 동일 옵션 진행 중 클레임 존재 |
| 409 | QUANTITY_OVER | 대상 수량이 주문 수량 초과 |
| 409 | EXCHANGE_OUT_OF_STOCK | 교환 대상 옵션 재고 부족 |
| 500 | INTERNAL_ERROR | 서버 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "{반품|교환} 신청이 접수되었습니다. (클레임코드: {claim_code})"
2. 주문상세 화면에서 해당 상품주문의 상태 표시 갱신
3. **교환의 경우:** 교환 대상 옵션의 재고 선차감(`exchange_inventory_reserved = true` 기록)
4. ClaimHistory에 `(NULL → 반품신청|교환신청)` 이력 자동 기록
5. FO 구매자에게 신청 접수 알림톡·메일 발송
6. 관리자 모니터링: 대상 [BO-CLM-023](../03-components/BO-CLM-023_반품-목록-테이블.md) / [BO-CLM-025](../03-components/BO-CLM-025_교환-목록-테이블.md) 목록에 승인 대기 건으로 노출

### 실패 시

1. 오류 토스트
2. 409 충돌(상태/기간) 시 주문상세 새로고침 안내

---

## 유효성 검사

### 프론트엔드

- `items` 1건 이상 필수, 각 `quantity ≥ 1`
- `reason_code`는 클레임 유형별 허용 사유 화이트리스트 내에서만 선택 가능
- 교환 신청 시 **교환 대상 옵션 선택 UI**에서 동일 상품 내 옵션만 노출
- 구매자 귀책 배송비 발생 시 카드 결제 UI 필수 완료

### 백엔드

- 주문 상태(`DELIVERED` / `EXCHANGE_DELIVERED`) 및 8일 이내 검증
- 중복 클레임 검증
- 원주문 수량과 정합성(`quantity ≤ OrderItem.quantity - 진행 중 클레임 수량`)
- 사유 코드 ↔ 클레임 유형 매트릭스 검증([BO-CLM-001 §4-(2)](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md#4-클레임-사유-및-귀책))
- 사유 코드 기반 `liability_party` 자동 결정
- **교환 전용:**
  - `exchange_target_option_id` 필수
  - 대상 옵션의 `product_id`가 원주문 `OrderItem.product_id`와 일치 검증
  - 대상 옵션의 `price`가 원주문 `OrderItem.unit_price`와 일치 검증
  - 대상 옵션 재고 확인 및 선차감(`exchange_inventory_reserved = true`, `exchange_target_option_id` 저장)
- 다수 출고지 포함 시 `shipment_id` 기준 분리 발번
