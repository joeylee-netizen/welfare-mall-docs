---
id: "BO-CLM-034"
title: "판매취소 접수"
type: action
domain: claim
status: draft
version: "1.0"
created: 2026-04-21
updated: 2026-04-21
author: "기획자"
refs:
  - "BO-CLM-001"
  - "BO-CLM-002"
  - "BO-CLM-010"
tags: [클레임, 판매취소, 접수, 관리자, 품절, API]
trigger: submit
method: POST
endpoint: "/api/v1/claims/cancel/admin"
---

# 판매취소 접수

## 트리거 조건

- 주문관리 모듈의 주문상세 화면에서 **"판매취소" 버튼** 클릭
- 상품 품절·발송 불가 등 **판매사 귀책**으로 BO/PO 관리자가 직접 주문취소를 신청하는 플로우
- 대상 주문상태: **결제완료** 또는 **상품준비중**

---

## 요청 (Request)

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| order_id | number | Y | 원주문 ID |
| shipment_id | number | Y | 출고지 ID |
| items | array | Y | 취소 대상 상품 목록 |
| items[].order_item_id | number | Y | 원주문 상품주문번호 |
| items[].quantity | number | Y | 취소 수량 |
| reason_code | string | Y | 클레임 사유 (주로 `SELLER_OUT_OF_STOCK`, `SELLER_DELIVERY_DELAY`, `OTHER`) |
| reason_detail | string | C | 사유 상세 (`OTHER` 시 필수, 10~500자) |
| memo | string | N | 관리자 처리 메모 |

> 판매취소는 **판매사 귀책 사유만 허용**. 구매자 귀책 사유(`BUYER_*`) 입력 시 400 반환.

---

## 응답 (Response)

### 성공 (201 Created)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_code | string | 클레임 코드 |
| claim_sub_type | string | `ORDER_CANCEL_BY_ADMIN` |
| claim_status_bo | string | `REFUND_REQUESTED` — 즉시 환불요청 단계로 전환 |
| approval_required | boolean | 항상 `false` (관리자 직접 접수이므로 승인 불필요) |
| process_channel | string | `BO_PROXY` |
| applied_channel | string | `ADMIN` |
| liability_party | string | `SELLER` (자동 설정) |
| refund_total_amount | number | 환불 예정 총액 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_REASON | 판매취소에 허용되지 않은 사유 코드 |
| 400 | INVALID_PARAM | 파라미터 누락/잘못된 값 |
| 400 | REASON_DETAIL_REQUIRED | `OTHER` 사유인데 상세 누락 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | ORDER_NOT_FOUND | 존재하지 않는 주문 |
| 409 | ORDER_STATUS_INVALID | 결제완료·상품준비중 외 상태 |
| 409 | CLAIM_IN_PROGRESS | 동일 상품 부분수량 클레임 진행 중 |
| 409 | QUANTITY_OVER | 취소 수량 초과 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "판매취소가 접수되었습니다. 환불 처리가 진행됩니다. (클레임코드: {claim_code})"
2. 시스템이 즉시 환불 프로세스 시작(RefundRecord 생성 → PG/포인트 API 호출)
3. FO 구매자에게 판매사 사정으로 인한 주문취소 안내 알림톡·메일 발송(알림 모듈 연계)
4. ClaimHistory에 `(NULL → SALE_CANCEL_REQUESTED → REFUND_REQUESTED)` 이력 기록
5. 원주문 상태를 판매취소 상태로 갱신

### 실패 시

1. 오류 토스트
2. 재시도 가능

---

## 유효성 검사

### 프론트엔드

- `reason_code`는 판매사 귀책 사유 리스트에서만 선택 가능(UI에서 구매자 귀책 사유 숨김)
- `items` 1건 이상 필수
- 수량/원주문 수량 범위 검증

### 백엔드

- `reason_code`는 `LiabilityParty = SELLER`인 사유인지 검증
- `liability_party`는 서버에서 자동으로 `SELLER` 설정
- 주문 상태·중복 클레임·수량 검증([BO-CLM-033](./BO-CLM-033_주문취소-접수-대행.md#유효성-검사) 참고)
- 즉시 환불 요청 트리거(비동기)
- 알림 발송 트리거
