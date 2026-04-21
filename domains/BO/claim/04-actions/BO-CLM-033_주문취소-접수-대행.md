---
id: "BO-CLM-033"
title: "주문취소 접수(대행)"
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
tags: [클레임, 주문취소, 접수, 대행, 고객센터, API]
trigger: submit
method: POST
endpoint: "/api/v1/claims/cancel/proxy"
---

# 주문취소 접수(대행)

## 트리거 조건

- 주문관리 모듈의 주문상세 화면에서 **"주문취소 대행" 버튼** 클릭
- 고객센터(CS)를 통해 접수된 구매자의 주문취소 요청을 BO/PO 관리자가 대행 접수하는 플로우
- 대상 주문상태: **결제완료** 또는 **상품준비중**

---

## 사전 호출

- 주문상세 조회 API(주문관리 모듈): 취소 가능 상품주문번호·옵션·수량 확인
- 취소 대상 상품별 중복 클레임 진행 여부 검증(동일 상품 부분수량 진행 중일 경우 불가)

---

## 요청 (Request)

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| order_id | number | Y | 원주문 ID |
| shipment_id | number | Y | 출고지 ID(출고지 단위 발번) |
| items | array | Y | 취소 대상 상품 목록 |
| items[].order_item_id | number | Y | 원주문 상품주문번호 |
| items[].quantity | number | Y | 취소 수량(≥1, 원주문 수량 이하) |
| reason_code | string | Y | 클레임 사유 코드 ([BO-CLM-010#ClaimReasonCode](../02-data/BO-CLM-010_클레임-엔티티.md#ClaimReasonCode)) |
| reason_detail | string | C | 사유 상세 (reason_code=`OTHER` 시 필수, 10~500자) |
| buyer_refund_bank | object | C | 실시간 계좌이체 환불 계좌(해당 결제수단 사용 시 필수) |
| buyer_refund_bank.bank_code | string | C | 은행코드 |
| buyer_refund_bank.account_number | string | C | 계좌번호 |
| buyer_refund_bank.account_holder | string | C | 예금주명 |
| memo | string | N | 접수 메모(0~500자) — CS 접수 경로/내용 요약 |

---

## 응답 (Response)

### 성공 (201 Created)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 생성된 클레임 ID |
| claim_code | string | 발번된 클레임 코드 |
| claim_sub_type | string | `ORDER_CANCEL_BY_USER` |
| claim_status_bo | string | 결제완료 건: `REFUND_REQUESTED`(즉시 환불요청). 상품준비중 건: `CANCEL_REQUESTED`(승인 대기) |
| approval_required | boolean | 상품준비중 건=true, 결제완료 건=false |
| process_channel | string | `BO_PROXY` |
| applied_channel | string | `BO_PROXY` |
| refund_total_amount | number | 환불 예정 총액 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 파라미터 누락/잘못된 값 |
| 400 | REASON_DETAIL_REQUIRED | `reason_code=OTHER`인데 `reason_detail` 누락 |
| 400 | REFUND_BANK_REQUIRED | 계좌이체 결제인데 환불 계좌 미입력 |
| 403 | FORBIDDEN | 대상 주문 고객사가 관리자 권한 범위 외 |
| 404 | ORDER_NOT_FOUND | 존재하지 않는 주문 |
| 409 | ORDER_STATUS_INVALID | 결제완료·상품준비중 외 상태 |
| 409 | CLAIM_IN_PROGRESS | 동일 상품 부분수량 클레임 진행 중 |
| 409 | QUANTITY_OVER | 취소 수량이 주문 수량을 초과 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "주문취소가 접수되었습니다. (클레임코드: {claim_code})"
2. 주문상세 화면에서 해당 상품주문의 상태가 `주문취소 신청`(또는 `환불요청 완료`)으로 갱신
3. `approval_required = false`(결제완료 건)인 경우 시스템이 즉시 PG 환불/포인트 복원 요청을 실행(비동기)
4. `approval_required = true`(상품준비중 건)인 경우 [BO-CLM-021 주문취소 목록](../03-components/BO-CLM-021_주문취소-목록-테이블.md)에 긴급 처리 대기 건으로 노출
5. ClaimHistory에 `(NULL → CANCEL_REQUESTED)` 이력 자동 기록

### 실패 시

1. 오류 토스트
2. 409 충돌 시 주문상세 자동 새로고침 안내

---

## 유효성 검사

### 프론트엔드

- `items` 1건 이상 필수, 각 `quantity ≥ 1`
- `reason_code=OTHER` 선택 시 `reason_detail` 10자 이상 필수
- 결제수단이 실시간 계좌이체 포함인 경우 환불 계좌 입력 필수
- 수량 단위 취소는 쿠폰 사용 등으로 수량별 상품주문번호가 구분된 경우에만 허용 ([BO-CLM-001 §2-(2)](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md#2-클레임-기본-정책))

### 백엔드

- 주문 상태(`PAID`/`PREPARING`) 검증
- 동일 상품 진행 중 클레임 존재 여부 검증
- 원주문 수량과의 정합성(`quantity ≤ OrderItem.quantity - 기존 클레임 수량`)
- 쿠폰 조건 재계산 및 원복 필요성 판정
- 취소 금액 계산(`item_refund_amount` = 단가×수량 - 할인분배)
- 다수 출고지 포함 시 `shipment_id` 기준 분리 발번(다른 출고지는 별도 API 호출 필요)
- 결제완료 건의 경우 환불 프로세스 자동 트리거(RefundRecord 생성)
