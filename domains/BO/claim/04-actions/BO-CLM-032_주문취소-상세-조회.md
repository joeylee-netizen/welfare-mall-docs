---
id: "BO-CLM-032"
title: "주문취소 상세 조회"
type: action
domain: claim
status: draft
version: "1.0"
created: 2026-04-21
updated: 2026-04-21
author: "기획자"
refs:
  - "BO-CLM-010"
  - "BO-CLM-022"
tags: [클레임, 주문취소, 상세, 조회, 모달, API]
trigger: load
method: GET
endpoint: "/api/v1/claims/{claim_id}"
---

# 주문취소 상세 조회

## 트리거 조건

- [BO-CLM-020](../03-components/BO-CLM-020_배송상품-클레임-목록-테이블.md) 또는 [BO-CLM-021](../03-components/BO-CLM-021_주문취소-목록-테이블.md) 목록에서 클레임코드 클릭 시
- [BO-CLM-022](../03-components/BO-CLM-022_주문취소-상세-폼.md) 모달 오픈 시 자동 호출
- 모달 내 처리(승인/반려/완료/재전송) 후 재조회 시

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Query Parameters

없음.

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim | object | 클레임 헤더 정보 |
| claim.claim_id | number | 클레임 ID |
| claim.claim_code | string | 클레임 코드 |
| claim.claim_type | string | 클레임 유형 |
| claim.claim_sub_type | string | 하위 유형 |
| claim.claim_status_fo | string | FO 상태 |
| claim.claim_status_bo | string | BO 상태 |
| claim.process_channel | string | 처리 채널 |
| claim.applied_channel | string | 접수 채널 |
| claim.applied_by_id | number | 신청자 ID |
| claim.applied_by_name | string | 신청자 표시명 |
| claim.applied_at | datetime | 신청일시 |
| claim.reason_code | string | 사유 코드 |
| claim.reason_detail | string | 사유 상세 |
| claim.liability_party | string | 귀책 |
| claim.approval_required | boolean | 수기 승인 필요 |
| claim.reject_reason | string | 반려 사유(있을 경우) |
| claim.completed_at | datetime | 완료일시 |
| order | object | 원주문 요약 |
| order.order_id | number | 주문 ID |
| order.order_status | string | 원주문 상태 |
| order.order_created_at | datetime | 주문일시 |
| order.buyer_id | number | 구매자 ID |
| order.buyer_name | string | 구매자명 |
| order.client_name | string | 고객사명 |
| order.shipment_id | number | 출고지 ID |
| order.shipment_name | string | 출고지명 |
| order.shipping_address | string | 배송지 주소 |
| order.vendor_id | number | 판매사 ID |
| order.vendor_name | string | 판매사명 |
| order.payment_method | string | 결제수단 표기(복합결제 포함) |
| order.payment_total | number | 총 결제금액 |
| order.payment_pg_amount | number | PG 결제액 |
| order.payment_point_amount | number | 포인트 사용액 |
| order.used_coupons | array | 사용 쿠폰 목록 |
| order.used_coupons[].coupon_id | number | 쿠폰 ID |
| order.used_coupons[].coupon_name | string | 쿠폰명 |
| order.used_coupons[].discount_amount | number | 할인액 |
| items | array | 취소 대상 ClaimItem 목록 |
| items[].claim_item_id | number | 클레임 아이템 ID |
| items[].product_id | number | 상품 ID |
| items[].product_name | string | 상품명 |
| items[].option_name | string | 옵션명 |
| items[].quantity | number | 수량 |
| items[].unit_price | number | 단가 |
| items[].discount_allocated | number | 할인 분배 |
| items[].item_refund_amount | number | 환불 예정 금액 |
| items[].coupon_id | number | 적용 쿠폰 ID |
| items[].coupon_name | string | 적용 쿠폰명 |
| refund | object | 환불 요약 |
| refund.refund_total_amount | number | 총 환불액 |
| refund.refund_pg_amount | number | PG 환불 |
| refund.refund_point_amount | number | 포인트 복원 |
| refund.shipping_fee_charged | number | 클레임 배송비 |
| refund.shipping_fee_liable_to | string | 배송비 부담 주체 |
| refund.additional_shipping_fee | number | 추가 배송비 |
| refund.coupon_restore_status | string | 쿠폰 원복 상태 |
| refund.coupon_restore_note | string | 쿠폰 원복 메모 |
| refund.records | array | 환불 수단별 RefundRecord 목록 |
| refund.records[].refund_id | number | 환불 레코드 ID |
| refund.records[].refund_method | string | 환불 수단 |
| refund.records[].refund_amount | number | 요청 금액 |
| refund.records[].refund_status | string | 상태 |
| refund.records[].triggered_by | string | 트리거 주체 |
| refund.records[].retry_count | number | 시도 횟수 |
| refund.records[].pg_transaction_id | string | PG 거래 ID |
| refund.records[].pg_response_message | string | PG 응답 메시지 |
| refund.records[].requested_at | datetime | 요청일시 |
| refund.records[].responded_at | datetime | 응답일시 |
| history | array | 상태 전이 이력 (최신 순) |
| history[].history_id | number | 이력 ID |
| history[].from_status | string | 이전 상태 |
| history[].to_status | string | 이후 상태 |
| history[].process_channel | string | 처리 채널 |
| history[].processor_name | string | 처리자 |
| history[].memo | string | 메모 |
| history[].created_at | datetime | 기록 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 401 | UNAUTHORIZED | 인증 실패 |
| 403 | FORBIDDEN | 조회 권한 없음(타 고객사 클레임) |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 모달의 5개 섹션에 데이터 매핑
2. 하단 버튼 영역을 `claim_status_bo` × `approval_required` 조건으로 분기 렌더링([BO-CLM-022 §모드 제어](../03-components/BO-CLM-022_주문취소-상세-폼.md#모드-제어-하단-버튼-분기))
3. 타임라인(이력) 섹션 최신 순 렌더링

### 실패 시

1. 모달 중앙에 에러 메시지 + 재시도 버튼
2. 404 시 모달 자동 닫힘 + 상위에 토스트

---

## 유효성 검사

### 프론트엔드

- `claim_id`는 양의 정수

### 백엔드

- 관리자의 권한 `client_id` 범위 내 조회만 허용(타 고객사 403)
- 응답 시 FO 전용 민감 필드(예: 마스킹 대상 구매자 상세 정보) 필터링
