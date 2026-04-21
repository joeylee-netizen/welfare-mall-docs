---
id: "BO-CLM-03E"
title: "교환 최종승인"
type: action
domain: claim
status: draft
version: "1.0"
created: 2026-04-22
updated: 2026-04-22
author: "기획자"
refs:
  - "BO-CLM-004"
  - "BO-CLM-010"
  - "BO-CLM-025"
  - "BO-CLM-026"
tags: [클레임, 교환, 검수, 최종승인, 교환주문, 재배송, API]
trigger: submit
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/exchange/finalize"
---

# 교환 최종승인

## 트리거 조건

- [BO-CLM-025 교환 목록](../03-components/BO-CLM-025_교환-목록-테이블.md) 개별/일괄 `최종승인` 버튼
- [BO-CLM-026 교환 상세](../03-components/BO-CLM-026_교환-상세-폼.md)의 `최종승인` 버튼
- 대상 상태: `EXCHANGE_COLLECTED` (교환회수완료)
- 대상 유형: **교환 전용** (`claim_type = EXCHANGE`)

검수 결과를 `PASSED`로 기록하고 **교환주문을 신규 발급**하여 재배송을 트리거한다.

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| inspection_memo | string | N | 검수 메모(0~1000자) |

> `inspection_result`는 서버에서 `PASSED`로 자동 설정.

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_status_bo | string | `EXCHANGE_FINAL_APPROVED` |
| claim_status_fo | string | `EXCHANGE_FINAL_APPROVED` |
| inspection_result | string | `PASSED` |
| inspection_by_id | number | 검수자 ID |
| inspection_at | datetime | 검수 일시 |
| exchange_order_id | number | **발급된 교환주문 ID** |
| exchange_order_code | string | 교환주문번호 |
| exchange_final_approved_at | datetime | 최종승인 일시 |
| exchange_inventory_confirmed | boolean | 재고 선차감 확정 여부(true) |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 권한 없음 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `EXCHANGE_COLLECTED` 아님 |
| 409 | INVALID_CLAIM_TYPE | `claim_type` ≠ `EXCHANGE` (반품에는 본 API 사용 불가) |
| 500 | EXCHANGE_ORDER_CREATION_FAILED | 교환주문 발급 실패 (롤백 후 재시도 가능) |
| 500 | INTERNAL_ERROR | 서버 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "교환이 최종 승인되었습니다. 교환주문번호: {exchange_order_code}"
2. 해당 행 상태 `EXCHANGE_FINAL_APPROVED`로 갱신 + 교환주문번호 표시
3. 상세 팝업 재조회 → 섹션 6(교환 최종승인 정보)에 교환주문 정보 표시
4. **교환주문 발급 프로세스(트랜잭션):**
   - 신규 Order 레코드 생성(`exchange_order_id`) — 주문 모듈 연동
   - 교환 대상 옵션 수량만큼 주문 아이템 생성
   - **재고 선차감 확정**(`exchange_inventory_reserved`를 실제 재고 차감으로 확정)
   - 재배송 트리거(주문 모듈에서 배송 프로세스 시작)
5. Claim에 `exchange_order_id` 업데이트
6. ClaimHistory에 `(EXCHANGE_COLLECTED → EXCHANGE_FINAL_APPROVED)` + 교환주문번호·검수자·메모 기록
7. 쿠폰/배송비 정산은 원주문 기준 유지(교환은 동일 가격이므로 환불 발생 없음)
8. FO 구매자에게 교환 승인·재배송 안내 알림톡 발송

### 실패 시

1. 오류 토스트
2. **500 EXCHANGE_ORDER_CREATION_FAILED:** 전체 트랜잭션 롤백(Claim 상태 유지), 버튼 활성 유지(재시도 가능), CS 알림
3. 409 상태 충돌 시 상세 새로고침

### 일괄 처리 시

- 각 건 개별 호출. 교환주문 발급은 건별 독립 트랜잭션.

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = EXCHANGE_COLLECTED` 및 `claim_type = EXCHANGE`
- 확인 모달 필수

### 백엔드

- 상태·유형 검증
- 관리자 권한 범위 내 `client_id` 검증
- 트랜잭션 처리(Claim 업데이트 + Order 생성 + 재고 확정 + Shipment 생성 원자적)
- 교환주문 발급 실패 시 전체 롤백(재고 상태 유지, Claim 상태 유지)
- 교환주문의 배송지는 원주문 배송지 복사
- 교환주문의 결제 정보는 원주문 결제 참조(재결제 없음 — 동일 가격)
