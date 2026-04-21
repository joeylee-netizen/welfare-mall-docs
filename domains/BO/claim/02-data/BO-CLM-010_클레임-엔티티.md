---
id: "BO-CLM-010"
title: "클레임 엔티티"
type: data
domain: claim
status: draft
version: "2.0"
created: 2026-04-21
updated: 2026-04-22
author: "기획자"
refs:
  - "BO-CLM-001"
  - "BO-CLM-002"
  - "BO-CLM-003"
  - "BO-CLM-004"
tags: [클레임, 엔티티, 데이터, 주문취소, 반품, 교환, 수거, 검수, 환불]
entity_name: "Claim"
source_system: "welfare-mall-api"
---

# 클레임 엔티티

## 1. Claim (클레임 헤더)

클레임 1건을 표현하는 최상위 엔티티. 사용자 또는 관리자가 신청한 클레임 단위로 1:1 발번된다. 다수 출고지 주문에서 발생한 클레임은 **출고지 단위**로 분리되어 각각의 Claim 레코드로 저장된다.

### (1) 필드 정의

#### 기본정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| claim_id | BIGINT | Y | 클레임 고유 ID | PK, Auto Increment |
| claim_code | VARCHAR(24) | Y | 클레임 코드 | `CLM{YYYYMMDD}{6자리}`, UNIQUE |
| claim_type | Enum | Y | 클레임 유형 | `ClaimType` 참조 |
| claim_sub_type | Enum | Y | 클레임 하위 유형 | `ClaimSubType` 참조 (주문취소/판매취소 등 구분) |
| order_id | BIGINT | Y | 원주문 ID | FK → Order |
| shipment_id | BIGINT | Y | 출고지 ID | FK → Shipment. 출고지 단위 발번 키 |
| buyer_id | BIGINT | Y | 구매자(FO 회원) ID | FK → Member |
| client_id | BIGINT | Y | 고객사 ID | FK → Client (멀티태넌트 필수) |

#### 상태 정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| claim_status_fo | Enum | Y | FO 노출 클레임 상태 | `ClaimStatusFO` 참조 |
| claim_status_bo | Enum | Y | BO/PO 클레임 상태 | `ClaimStatusBO` 참조 |
| process_channel | Enum | Y | 처리 채널 | `ProcessChannel` 참조 (FO/BO/SYSTEM) |
| approval_required | BOOLEAN | Y | 관리자 수기 승인 필요 여부 | 상품준비중 단계=true, 결제완료 단계=false |
| status_changed_at | DATETIME | Y | 최종 상태 변경 일시 | 시스템 자동 |

#### 신청 정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| applied_channel | Enum | Y | 접수 채널 | `AppliedChannel` 참조 (FO/BO_PROXY/ADMIN) |
| applied_by_id | BIGINT | Y | 신청자 ID | FO 구매자 ID 또는 BO 관리자 ID |
| applied_by_name | VARCHAR(100) | Y | 신청자 표시명 | 스냅샷(이력 보존) |
| applied_at | DATETIME | Y | 신청 일시 | 시스템 자동 |

#### 사유/귀책

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| reason_code | Enum | Y | 클레임 사유 코드 | `ClaimReasonCode` 참조 |
| reason_detail | VARCHAR(500) | N | 사유 상세 | 사유코드=OTHER 시 필수 |
| liability_party | Enum | Y | 귀책 주체 | `LiabilityParty` 참조 (BUYER/SELLER) |

#### 환불/정산 정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| refund_total_amount | DECIMAL(12,2) | Y | 총 환불 예정 금액 | 부분취소 시 취소 대상 금액만 |
| refund_pg_amount | DECIMAL(12,2) | Y | PG 환불 금액 | 카드/계좌이체 |
| refund_point_amount | DECIMAL(12,2) | Y | 복지포인트 복원 금액 | |
| shipping_fee_charged | DECIMAL(10,2) | Y | 클레임 배송비 부과 금액 | 구매자 귀책 시 |
| shipping_fee_liable_to | Enum | Y | 클레임 배송비 부담 주체 | `LiabilityParty` 참조 |
| additional_shipping_fee | DECIMAL(10,2) | Y | 추가배송비(무료배송 조건 미달 시) | 기본 0 |

#### 쿠폰 처리

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| coupon_restore_status | Enum | Y | 쿠폰 원복 상태 | `CouponRestoreStatus` 참조 (NONE/REQUIRED/COMPLETED/MANUAL_REQUIRED) |
| coupon_restore_note | VARCHAR(500) | N | 쿠폰 원복 처리 메모 | CS 수기 발급 사유 등 |

#### 반려 정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| reject_reason | VARCHAR(200) | N | 반려 사유 | BO/PO 반려 시 필수(10~200자) |
| rejected_by_id | BIGINT | N | 반려자 ID | FK → Admin |
| rejected_at | DATETIME | N | 반려 일시 | |

#### 반품/교환 전용 (NULL 허용, `claim_type ∈ {RETURN, EXCHANGE}` 시 사용)

##### 수거 정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| pickup_carrier_code | VARCHAR(20) | N | 반송 택배사 코드 | CJ/로젠/한진 등 |
| pickup_carrier_name | VARCHAR(50) | N | 반송 택배사명 스냅샷 | 이력 보존 |
| pickup_tracking_number | VARCHAR(50) | N | 반송장 번호 | 수거 요청 시 등록 |
| pickup_requested_at | DATETIME | N | 수거 요청 일시 | 관리자가 반송장 등록한 시점 |
| pickup_picked_up_at | DATETIME | N | 택배사 상품 수령 일시 | 시스템 자동 |
| pickup_returned_at | DATETIME | N | 판매사 회수 완료 일시 | 시스템 자동 |

##### 검수 정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| inspection_result | Enum | N | 검수 결과 | `InspectionResult` 참조. 회수완료 이후 기록 |
| inspection_memo | VARCHAR(1000) | N | 검수 메모 | 거부 사유 상세 등 |
| inspection_at | DATETIME | N | 검수 일시 | |
| inspection_by_id | BIGINT | N | 검수자 관리자 ID | FK → Admin |

##### 교환 전용 (EXCHANGE만 사용)

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| exchange_target_product_id | BIGINT | N | 교환 대상 상품 ID | 동일 상품 제약(원주문 상품과 일치) |
| exchange_target_option_id | BIGINT | N | 교환 대상 옵션 ID | FK → ProductOption. 동일 가격 옵션만 허용 |
| exchange_target_option_name | VARCHAR(200) | N | 교환 대상 옵션명 스냅샷 | |
| exchange_inventory_reserved | BOOLEAN | N | 교환 재고 선차감 여부 | 신청 시 true, 종결(철회/거부)·최종승인 시 처리 확정 |
| exchange_order_id | BIGINT | N | 교환 주문번호 | 최종승인 시 발급. FK → Order |
| exchange_redelivery_tracking | VARCHAR(50) | N | 교환 재배송 송장 | 주문 모듈에서 동기화 |
| exchange_final_approved_at | DATETIME | N | 교환 최종승인 일시 | |

#### 이력

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| created_at | DATETIME | Y | 생성일시 | 시스템 자동 |
| updated_at | DATETIME | Y | 수정일시 | 시스템 자동 |
| completed_at | DATETIME | N | 종결일시 | 환불완료·교환최종승인·철회 시 기록 |

### (2) Enum 정의

#### ClaimType (클레임 유형)

| 값 | 설명 | 비고 |
|----|------|------|
| `CANCEL` | 주문취소 | [BO-CLM-002](../01-policies/BO-CLM-002_주문취소-정책.md) |
| `RETURN` | 반품 | [BO-CLM-003](../01-policies/BO-CLM-003_반품-정책.md) |
| `EXCHANGE` | 교환 | [BO-CLM-004](../01-policies/BO-CLM-004_교환-정책.md) |

#### ClaimSubType (클레임 하위 유형)

| 값 | 설명 | 상위 유형 | 비고 |
|----|------|-----------|------|
| `ORDER_CANCEL_BY_USER` | 주문취소(사용자) | CANCEL | FO 또는 BO 대행 접수 |
| `ORDER_CANCEL_BY_ADMIN` | 판매취소(관리자) | CANCEL | BO/PO 관리자 직접 접수 |
| `RETURN_BY_USER` | 반품(사용자) | RETURN | FO 직접 신청 |
| `RETURN_BY_ADMIN` | 반품(관리자 대행) | RETURN | BO/PO 대행 접수 (CS 요청) |
| `EXCHANGE_BY_USER` | 교환(사용자) | EXCHANGE | FO 직접 신청 |
| `EXCHANGE_BY_ADMIN` | 교환(관리자 대행) | EXCHANGE | BO/PO 대행 접수 (CS 요청) |

#### ClaimStatusFO (FO 노출 클레임 상태)

##### 주문취소 계열 (CANCEL)

| 값 | 설명 | 비고 |
|----|------|------|
| `CLAIM_REQUESTED` | 주문취소 | FO 통합 표시 — 신청~환불요청 완료까지 포함 |
| `CLAIM_COMPLETED` | 주문취소 | 환불완료 상태에서도 동일 노출 |
| `CLAIM_CANCELED` | 주문취소 철회 | 구매자 철회 또는 관리자 반려 |

##### 반품 계열 (RETURN)

| 값 | 설명 | 비고 |
|----|------|------|
| `RETURN_REQUESTED` | 반품신청 | FO/BO 대행 접수 직후 |
| `RETURN_RECEIVED` | 반품접수 | 관리자 승인 후 |
| `RETURN_PICKUP_IN_PROGRESS` | 반품수거중 | 반송장 등록 후 |
| `RETURN_PICKED_UP` | 반품상품수거 | 택배사 수령 |
| `RETURN_COLLECTED` | 반품회수완료 | 판매사 창고 회수 |
| `RETURN_COMPLETED` | 반품완료 | 환불요청완료·환불완료·환불실패 FO 통합 표시 |
| `RETURN_WITHDRAWN` | 반품철회 | 구매자 철회 또는 관리자 거절 / 검수거부 |

##### 교환 계열 (EXCHANGE)

| 값 | 설명 | 비고 |
|----|------|------|
| `EXCHANGE_REQUESTED` | 교환신청 | FO/BO 대행 접수 직후 |
| `EXCHANGE_RECEIVED` | 교환접수 | 관리자 승인 후 |
| `EXCHANGE_PICKUP_IN_PROGRESS` | 교환수거중 | 반송장 등록 후 |
| `EXCHANGE_PICKED_UP` | 교환상품수거 | 택배사 수령 |
| `EXCHANGE_COLLECTED` | 교환회수완료 | 판매사 창고 회수 |
| `EXCHANGE_FINAL_APPROVED` | 교환최종승인 | 최종승인 완료. 교환주문 발급·재배송 트리거 |
| `EXCHANGE_WITHDRAWN` | 교환철회 | 구매자 철회 또는 관리자 거절 / 검수거부 |

> FO에는 세부 단계를 단순화하여 노출(예: "반품완료"는 환불요청완료·환불완료·환불실패를 통합 표기). BO에는 단계별 상세 상태 노출.

#### ClaimStatusBO (BO/PO 관리자용 클레임 상태)

##### 주문취소 계열 (CANCEL)

| 값 | 설명 | 처리 채널 | 상태 전이 규칙 |
|----|------|-----------|---------------|
| `CANCEL_REQUESTED` | 주문취소 신청 | FO / BO/PO 대행 | 결제완료 건은 자동→`REFUND_REQUESTED`, 상품준비중 건은 관리자 승인 대기 |
| `SALE_CANCEL_REQUESTED` | 판매취소 신청 | BO/PO | 즉시 시스템 전환 → `REFUND_REQUESTED` |
| `REFUND_REQUESTED` | 환불요청 완료 | system / BO/PO | PG 성공→`REFUNDED`, 실패→`REFUND_FAILED` |
| `REFUNDED` | 환불완료 | system / BO/PO | 종결 상태 |
| `REFUND_FAILED` | 환불실패 | system | 관리자 재전송 또는 수기 완료 필요 |
| `CANCEL_WITHDRAWN` | 주문취소 철회 | FO / BO/PO | 종결 상태(구매자 철회 또는 관리자 반려) |

##### 반품 계열 (RETURN)

| 값 | 설명 | 처리 채널 | 상태 전이 규칙 |
|----|------|-----------|---------------|
| `RETURN_REQUESTED` | 반품신청 | FO / BO/PO 대행 | 관리자 승인 대기. 거절 시 `RETURN_WITHDRAWN` |
| `RETURN_RECEIVED` | 반품접수 | BO/PO | 수거요청 대기(반송장 등록) |
| `RETURN_PICKUP_IN_PROGRESS` | 반품수거중 | BO/PO | 택배사 수령 대기 |
| `RETURN_PICKED_UP` | 반품상품수거 | system | 택배사 수령 완료. 판매사 회수 대기 |
| `RETURN_COLLECTED` | 반품회수완료 | system | 관리자 검수 대기. 검수 통과→`REFUND_REQUESTED`, 거부→`RETURN_WITHDRAWN` |
| `RETURN_WITHDRAWN` | 반품철회 | FO / BO/PO | 종결. FO 철회·BO 거절·검수거부 |

> 반품의 환불 단계(`REFUND_REQUESTED`·`REFUNDED`·`REFUND_FAILED`)는 주문취소 계열과 enum 값을 공유한다.

##### 교환 계열 (EXCHANGE)

| 값 | 설명 | 처리 채널 | 상태 전이 규칙 |
|----|------|-----------|---------------|
| `EXCHANGE_REQUESTED` | 교환신청 | FO / BO/PO 대행 | 재고 선차감 상태. 거절 시 `EXCHANGE_WITHDRAWN` + 재고 복원 |
| `EXCHANGE_RECEIVED` | 교환접수 | BO/PO | 수거요청 대기 |
| `EXCHANGE_PICKUP_IN_PROGRESS` | 교환수거중 | BO/PO | 택배사 수령 대기 |
| `EXCHANGE_PICKED_UP` | 교환상품수거 | system | 택배사 수령 완료 |
| `EXCHANGE_COLLECTED` | 교환회수완료 | system | 관리자 검수 대기. 최종승인→`EXCHANGE_FINAL_APPROVED`, 거부→`EXCHANGE_WITHDRAWN`+재고 복원 |
| `EXCHANGE_FINAL_APPROVED` | 교환최종승인 | BO/PO | 종결. 교환주문번호 발급 + 재배송 트리거 |
| `EXCHANGE_WITHDRAWN` | 교환철회 | FO / BO/PO | 종결. 재고 복원 완료 |

#### ProcessChannel (처리 채널)

| 값 | 설명 |
|----|------|
| `FO` | FO 구매자 |
| `BO_PROXY` | BO/PO 관리자(대행 접수·수기 처리) |
| `SYSTEM` | 시스템 자동 |

#### AppliedChannel (접수 채널)

| 값 | 설명 |
|----|------|
| `FO` | FO 구매자 직접 신청 |
| `BO_PROXY` | BO/PO 관리자가 고객 요청 대행 접수 |
| `ADMIN` | BO/PO 관리자 직접 신청 (판매취소) |

#### ClaimReasonCode (클레임 사유 코드)

| 값 | 설명 | 허용 유형 | 귀책 |
|----|------|:---------:|:----:|
| `BUYER_CHANGE_OF_MIND` | 단순변심 | 취소/반품/교환 | 구매자 |
| `BUYER_OPTION_DISSATISFACTION` | 색상/사이즈 옵션 불만족 | 반품/교환 | 구매자 |
| `BUYER_PRICE_COMPARISON` | 가격/프로모션 비교 | 취소 | 구매자 |
| `SELLER_DELIVERY_DELAY` | 배송지연 | 취소 | 판매사 |
| `SELLER_OUT_OF_STOCK` | 상품 품절 | 취소 | 판매사 |
| `SELLER_DEFECT` | 상품불량/파손 | 반품/교환 | 판매사 |
| `SELLER_MISSING_ITEM` | 상품누락배송 | 반품/교환 | 판매사 |
| `SELLER_WRONG_ITEM` | 다른상품배송 | 반품/교환 | 판매사 |
| `OTHER` | 기타 | 취소 | 판매사 |

#### LiabilityParty (귀책 주체)

| 값 | 설명 |
|----|------|
| `BUYER` | 구매자 귀책 |
| `SELLER` | 판매사 귀책 |

#### CouponRestoreStatus (쿠폰 원복 상태)

| 값 | 설명 |
|----|------|
| `NONE` | 쿠폰 미사용 또는 원복 대상 아님 |
| `REQUIRED` | 원복 필요(대기 중) |
| `COMPLETED` | 원복 완료 |
| `MANUAL_REQUIRED` | 쿠폰 종료일 경과 등으로 **CS 수기 재발급** 필요 |

#### InspectionResult (검수 결과)

반품·교환 클레임의 `반품회수완료`·`교환회수완료` 단계에서 관리자가 기록하는 검수 결과.

| 값 | 설명 | 후속 처리 |
|----|------|----------|
| `PASSED` | 검수 통과 | 반품: `REFUND_REQUESTED`로 전환 / 교환: `EXCHANGE_FINAL_APPROVED`로 전환 |
| `FAILED_DEFECT` | 구매자 귀책 훼손 | 검수거부 → 철회 + 원상품 재배송 (교환은 재고 복원) |
| `FAILED_MISSING` | 상품 누락 | 검수거부 → 철회 + 원상품 재배송 |
| `FAILED_OTHER` | 기타 사유 | 검수거부 → 철회. `inspection_memo` 필수 |

> 검수거부 시 재배송 비용은 구매자 귀책으로 처리([BO-CLM-003 §3-(4)](../01-policies/BO-CLM-003_반품-정책.md#3-반품-처리), [BO-CLM-004 §3-(4)](../01-policies/BO-CLM-004_교환-정책.md#3-교환-처리)).

### (3) 관계

| 관계 | 대상 엔티티 | 카디널리티 | 설명 |
|------|-------------|:----------:|------|
| 소속 주문 | Order | N:1 | 원주문 |
| 교환 주문 | Order | N:0..1 | `claim_type = EXCHANGE` + `EXCHANGE_FINAL_APPROVED` 시 발급되는 신규 주문 (`exchange_order_id`) |
| 출고지 | Shipment | N:1 | 출고지 단위 발번 |
| 구매자 | Member | N:1 | FO 회원 |
| 고객사 | Client | N:1 | 멀티태넌트 |
| 교환 대상 옵션 | ProductOption | N:0..1 | `claim_type = EXCHANGE` 시 교환 요청 옵션 |
| 클레임 상세 아이템 | ClaimItem | 1:N | 옵션×수량 단위 세부 레코드 |
| 환불 트랜잭션 | RefundRecord | 1:N | PG/포인트 환불 호출 이력 (반품·주문취소) |
| 클레임 이력 | ClaimHistory | 1:N | 상태 전이 이력 |
| 반려자 | Admin | N:0..1 | 반려 기록용 |
| 신청자(대행/관리자) | Admin | N:0..1 | AppliedChannel=BO_PROXY/ADMIN 시 |
| 검수자 | Admin | N:0..1 | `inspection_by_id` (반품·교환 검수) |

### (4) 인덱스

| 인덱스 | 컬럼 | 용도 |
|--------|------|------|
| PK | claim_id | 기본 키 |
| UQ_claim_code | claim_code | 유니크 제약 |
| IDX_order | order_id, shipment_id | 주문 단위 조회 |
| IDX_buyer | buyer_id, applied_at DESC | FO 마이페이지 목록 |
| IDX_bo_status | claim_status_bo, applied_at DESC | BO 목록 조회 |
| IDX_client_status | client_id, claim_status_bo | 멀티태넌트 필터 |
| IDX_approval | approval_required, claim_status_bo | 수기 승인 대상 필터 |
| IDX_claim_type | claim_type, claim_status_bo | 유형별 목록(반품/교환 전용 화면) |
| IDX_exchange_order | exchange_order_id | 교환주문 역조회 |
| IDX_pickup_tracking | pickup_tracking_number | 반송장 추적 |

### (5) 제약 조건

- `claim_code`는 UNIQUE.
- `(order_id, shipment_id, claim_type)` 조합으로 **진행 중(미종결)** 클레임이 중복 생성되지 않도록 애플리케이션 레벨에서 검증(부분 수량 클레임 동시 진행 금지, [BO-CLM-001 §2-(2)](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md#2-클레임-기본-정책)).
- `reject_reason`은 `CANCEL_WITHDRAWN`·`RETURN_WITHDRAWN`·`EXCHANGE_WITHDRAWN` 전이 중 **BO/PO 반려·검수거부**인 경우 필수(10~200자).
- `reason_detail`은 `reason_code = OTHER`인 경우 필수.
- `refund_total_amount = refund_pg_amount + refund_point_amount - shipping_fee_charged + additional_shipping_fee` 체크(반품·주문취소 한정).
- **반품/교환 전용 제약:**
  - `claim_type = RETURN`: 원주문의 주문상태가 `배송완료` 또는 `교환배송완료`이고 해당 시점 8일 이내여야 신규 생성 가능.
  - `claim_type = EXCHANGE`: `exchange_target_option_id`의 `product_id`가 **원주문 상품의 product_id와 일치**(동일 상품 제약).
  - `claim_type = EXCHANGE`: `exchange_target_option_id`의 `price`가 **원주문 옵션 price와 일치**(동일 가격 제약).
  - `claim_type = EXCHANGE`: `EXCHANGE_REQUESTED` 생성 시 `exchange_inventory_reserved = true`로 설정하고 교환 대상 옵션 재고 선차감. 종결(FINAL_APPROVED 또는 WITHDRAWN) 시 확정 또는 복원 처리.
  - `pickup_tracking_number`는 `RETURN_PICKUP_IN_PROGRESS` / `EXCHANGE_PICKUP_IN_PROGRESS` 전이 시 필수.
  - `inspection_result`는 `REFUND_REQUESTED`(반품 검수 통과) / `EXCHANGE_FINAL_APPROVED` / 검수거부 전이 시 필수.
  - `inspection_memo`는 `inspection_result = FAILED_OTHER`인 경우 필수.

---

## 2. ClaimItem (클레임 세부 아이템)

Claim 1건 내 **옵션 × 수량** 단위 세부 레코드. 부분 클레임 시 대상 상품/옵션 별로 1:N 생성.

### (1) 필드 정의

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| claim_item_id | BIGINT | Y | 클레임 아이템 ID | PK, Auto Increment |
| claim_id | BIGINT | Y | 상위 클레임 ID | FK → Claim |
| order_item_id | BIGINT | Y | 원주문 상품주문번호 | FK → OrderItem |
| product_id | BIGINT | Y | 상품 ID (스냅샷 참조) | FK → Product |
| product_code | VARCHAR(30) | Y | 상품코드 스냅샷 | 이력 보존 |
| product_name | VARCHAR(200) | Y | 상품명 스냅샷 | 이력 보존 |
| option_id | BIGINT | Y | 옵션 ID | FK → ProductOption |
| option_name | VARCHAR(200) | Y | 옵션명 스냅샷 | 이력 보존 |
| quantity | INT | Y | 클레임 대상 수량 | ≥ 1 |
| unit_price | DECIMAL(10,2) | Y | 단가 스냅샷 | 주문 시점 |
| item_refund_amount | DECIMAL(12,2) | Y | 아이템 환불 금액 | 할인 반영 후 |
| discount_allocated | DECIMAL(10,2) | Y | 아이템에 분배된 할인 금액 | 쿠폰·프로모션 |
| coupon_id | BIGINT | N | 적용된 쿠폰 ID | FK → Coupon |
| created_at | DATETIME | Y | 생성일시 | 시스템 자동 |

### (2) 제약 조건

- `quantity`는 원주문 `OrderItem.quantity` 이하.
- 동일 `order_item_id`에 대해 진행 중 ClaimItem이 존재하면 동일 수량 범위로 추가 생성 불가(동일 상품 부분수량 클레임 중복 금지).

---

## 3. RefundRecord (환불 트랜잭션 이력)

PG·복지포인트 환불 요청/응답 이력. 재전송·수기 완료·실패 로그를 모두 보존.

### (1) 필드 정의

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| refund_id | BIGINT | Y | 환불 레코드 ID | PK |
| claim_id | BIGINT | Y | 클레임 ID | FK → Claim |
| refund_method | Enum | Y | 환불 수단 | `RefundMethod` 참조 |
| refund_amount | DECIMAL(12,2) | Y | 환불 요청 금액 | |
| pg_transaction_id | VARCHAR(100) | N | PG 거래 ID | 카드/계좌이체 |
| pg_response_code | VARCHAR(20) | N | PG 응답 코드 | |
| pg_response_message | VARCHAR(500) | N | PG 응답 메시지 | |
| refund_status | Enum | Y | 환불 상태 | `RefundStatus` 참조 |
| requested_at | DATETIME | Y | 요청 일시 | |
| responded_at | DATETIME | N | 응답 일시 | |
| retry_count | INT | Y | 재시도 횟수 | 기본 0 |
| triggered_by | Enum | Y | 트리거 주체 | `RefundTrigger` 참조 (SYSTEM/BATCH/ADMIN_MANUAL) |
| triggered_by_admin_id | BIGINT | N | 수동 실행한 관리자 ID | FK → Admin |

### (2) Enum 정의

#### RefundMethod

| 값 | 설명 |
|----|------|
| `CARD_DOMESTIC` | 카드(국내) |
| `CARD_OVERSEAS` | 카드(해외) |
| `BANK_TRANSFER` | 실시간 계좌이체 |
| `WELFARE_POINT` | 복지포인트 |

#### RefundStatus

| 값 | 설명 |
|----|------|
| `PENDING` | 요청 대기/진행 중 |
| `SUCCESS` | 환불 성공 |
| `FAILED` | 환불 실패(PG 에러) |
| `MANUAL_COMPLETED` | 관리자 수기 완료 처리 |

#### RefundTrigger

| 값 | 설명 |
|----|------|
| `SYSTEM` | 클레임 상태 전이로 인한 자동 호출 |
| `BATCH` | 실패건 재전송 배치(1일 1회) |
| `ADMIN_MANUAL` | 관리자 수동 재전송/수기 완료 |

### (3) 제약 조건

- 동일 `claim_id` 내 `refund_method`별 `SUCCESS`/`MANUAL_COMPLETED`는 최대 1건.
- `retry_count`는 자동 배치 + 수동 실행 누적 값.

---

## 4. ClaimHistory (클레임 상태 이력)

상태 전이, 처리자, 사유 등 감사(audit) 로그. 모든 상태 변경 시 자동 기록.

### (1) 필드 정의

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| history_id | BIGINT | Y | 이력 ID | PK |
| claim_id | BIGINT | Y | 클레임 ID | FK → Claim |
| from_status | Enum | N | 이전 상태 | `ClaimStatusBO` (최초 생성 시 NULL) |
| to_status | Enum | Y | 이후 상태 | `ClaimStatusBO` |
| process_channel | Enum | Y | 처리 채널 | `ProcessChannel` |
| processor_id | BIGINT | N | 처리자 ID | FK → Admin (SYSTEM 시 NULL) |
| processor_name | VARCHAR(100) | N | 처리자 표시명 | 스냅샷 |
| memo | VARCHAR(500) | N | 이벤트 메모 | 반려사유·재전송 사유 등 |
| created_at | DATETIME | Y | 기록 일시 | 시스템 자동 |

### (2) 인덱스

| 인덱스 | 컬럼 | 용도 |
|--------|------|------|
| PK | history_id | 기본 키 |
| IDX_claim | claim_id, created_at DESC | 클레임 상세 타임라인 조회 |

---

## 5. 엔티티 관계 요약 (ERD)

```
Order ──1:N── Shipment ──1:N── OrderItem
                            │
                            ▼
                         Claim ──1:N── ClaimItem ──N:1── OrderItem
                            │
                            ├──1:N── RefundRecord
                            └──1:N── ClaimHistory

Member ──1:N── Claim
Client ──1:N── Claim (멀티태넌트)
Admin  ──N:0..1── Claim (applied_by / rejected_by)
```

---

## 관련 문서

- [BO-CLM-001](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md) — 공통 정책
- [BO-CLM-002](../01-policies/BO-CLM-002_주문취소-정책.md) — 주문취소 정책(상태값 원본)
- [BO-CLM-003](../01-policies/BO-CLM-003_반품-정책.md) — 반품 정책(상태값 원본)
- [BO-CLM-004](../01-policies/BO-CLM-004_교환-정책.md) — 교환 정책(상태값·교환주문 원본)
- [BO-CLM-020](../03-components/BO-CLM-020_배송상품-클레임-목록-테이블.md) — 통합 목록 테이블
- [BO-CLM-021](../03-components/BO-CLM-021_주문취소-목록-테이블.md) — 주문취소 목록 테이블
- [BO-CLM-022](../03-components/BO-CLM-022_주문취소-상세-폼.md) — 주문취소 상세 폼
- [BO-CLM-023](../03-components/BO-CLM-023_반품-목록-테이블.md) — 반품 목록 테이블
- [BO-CLM-024](../03-components/BO-CLM-024_반품-상세-폼.md) — 반품 상세 폼
- [BO-CLM-025](../03-components/BO-CLM-025_교환-목록-테이블.md) — 교환 목록 테이블
- [BO-CLM-026](../03-components/BO-CLM-026_교환-상세-폼.md) — 교환 상세 폼
