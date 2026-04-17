---
id: "BO-PDM-010"
title: "상품 엔티티"
type: data
domain: product
status: draft
version: "2.0"
created: 2026-04-13
updated: 2026-04-17
author: "기획자"
refs:
  - "BO-PDM-001"
tags: [상품, 엔티티, 데이터]
entity_name: "Product"
source_system: "welfare-mall-api"
---

# 상품 엔티티

## 1. Product (상품)

### (1) 필드 정의

#### 기본정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| product_id | BIGINT | Y | 상품 고유 ID | PK, Auto Increment |
| request_code | VARCHAR(30) | Y | 요청코드 | 저장(상품등록중) 시 자동 발번 |
| product_code | VARCHAR(30) | N | 상품코드 | 승인완료 시 부여, UNIQUE |
| product_type | Enum | Y | 상품 유형 | ProductType 참조 |
| product_category | Enum | Y | 상품 구분 | ProductCategory 참조 |
| brand_id | BIGINT | C | 브랜드 ID | FK → Brand, 배송형 상품 필수 |
| vendor_id | BIGINT | C | 판매사 ID | FK → Vendor, 티켓형 상품 필수 |
| product_name | VARCHAR(200) | Y | 상품명 | |
| sales_status | Enum | Y | 판매 상태 | SalesStatus 참조, 기본값: SELLING |
| approval_status | Enum | Y | 승인 상태 | ApprovalStatus 참조, 기본값: REGISTERING |

#### 판매기간

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| sales_start_date | DATE | N | 판매 시작일 | 승인완료 후 설정 |
| sales_end_date | DATE | N | 판매 종료일 | 종료 시 판매종료로 자동 전환 |

#### 확장정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| tax_type | Enum | Y | 과세구분 | TaxType 참조 (TAXABLE/TAX_FREE/ZERO_RATED) |
| model_name | VARCHAR(100) | N | 모델명 | |
| registration_source | Enum | Y | 등록방식 | RegistrationSource 참조, 기본값: DIRECT |
| is_temp_saved | BOOLEAN | Y | 임시저장 여부 | 기본값: false |
| request_type | Enum | Y | 요청구분 | RequestType 참조 |
| display_channel | Enum | Y | 노출채널 | DisplayChannel 참조, 기본값: ALL |
| is_displayed | BOOLEAN | Y | 전시여부 | 기본값: false |
| md_id | BIGINT | N | 담당 MD ID | FK → Admin |
| client_id | BIGINT | N | 고객사 ID | FK → Client |

#### 티켓상품 전용

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| validity_period_start | DATE | C | 유효기간 시작일 | 티켓형 필수 |
| validity_period_end | DATE | C | 유효기간 종료일 | 티켓형 필수 |
| usage_info | TEXT | C | 이용 안내 | 티켓형 필수 |
| issue_method | Enum | C | 발급 수단 | IssueMethod 참조, 티켓형 필수 |
| reservation_info | TEXT | N | 예약 안내 | TICKET_RESERVATION만 필수 |

#### 등록/수정 이력

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| created_at | DATETIME | Y | 등록일시 | 시스템 자동 |
| updated_at | DATETIME | Y | 수정일시 | 시스템 자동 |

### (2) Enum 정의

#### ProductType (상품 유형)

| 값 | 설명 | 비고 |
|----|------|------|
| GENERAL_SHIPPING | 일반 배송 상품 | 배송형 |
| INSTALLATION | 배송형 설치 상품 | 배송형 |
| TICKET_COUPON | 일반 티켓 상품(e쿠폰) | 티켓형 |
| TICKET_RESERVATION | 예약 티켓 상품(예약) | 티켓형 |

#### ProductCategory (상품 구분)

| 값 | 설명 |
|----|------|
| SHIPPING | 배송형 |
| TICKET | 티켓형 |

> **매핑 규칙:** `GENERAL_SHIPPING`, `INSTALLATION` → `SHIPPING` / `TICKET_COUPON`, `TICKET_RESERVATION` → `TICKET`

#### SalesStatus (판매 상태)

| 값 | 설명 | FO 전시 | FO 주문 |
|----|------|:-------:|:-------:|
| SELLING | 판매중 | O | O |
| SALES_PAUSED | 판매일시중지 | O | X |
| TEMP_OUT_OF_STOCK | 일시품절 | O | X |
| OUT_OF_STOCK | 품절 | O | X |
| SALES_ENDED | 판매종료 | X | X |

**상태 전환 규칙:**

- `SELLING` → `SALES_PAUSED`: 관리자 수동 전환
- `SELLING` → `TEMP_OUT_OF_STOCK`: 재고 0 발생 후 6일 이내 시스템 자동 전환
- `TEMP_OUT_OF_STOCK` → `OUT_OF_STOCK`: 재고 0 상태 7일 이상 지속 시 시스템 자동 전환
- `SELLING` → `SALES_ENDED`: 판매기간 만료 시 시스템 자동 전환 또는 관리자 수동 설정
- `SALES_PAUSED` / `TEMP_OUT_OF_STOCK` / `OUT_OF_STOCK` → `SELLING`: 관리자 수동 전환 (재고 보충 필요)

#### TaxType (과세구분)

| 값 | 설명 |
|----|------|
| TAXABLE | 과세 |
| TAX_FREE | 면세 |
| ZERO_RATED | 영세 |

#### RegistrationSource (등록방식)

| 값 | 설명 |
|----|------|
| DIRECT | PO/BO 직접 등록 |
| API | 외부 API 연동 등록 |

#### RequestType (요청구분)

| 값 | 설명 |
|----|------|
| NEW | 신규등록 |
| MODIFY | 상품수정 |

#### DisplayChannel (노출채널)

| 값 | 설명 |
|----|------|
| ALL | 전체 (PC+모바일) |
| PC | PC만 |
| MOBILE | 모바일만 |

#### IssueMethod (발급 수단)

| 값 | 설명 | 비고 |
|----|------|------|
| MMS | 문자(MMS) 발급 | |
| ALIMTALK | 알림톡 발급 | |
| APP_PUSH | 앱 푸시 발급 | |

#### ApprovalStatus (승인 상태)

| 값 | 설명 | 수정 가능 | 비고 |
|----|------|:---------:|------|
| REGISTERING | 상품등록중 | O | PO 저장 시 기본값 |
| PENDING | 승인대기 | X | PO 승인요청 시 |
| WITHDRAWN | 승인철회 | O | PO 승인요청 철회 시 |
| APPROVED | 승인완료 | O | BO 관리자 승인 시, 상품코드 부여 |
| REJECTED | 승인반려 | O | BO 관리자 반려 시, 반려사유 필수 |

**상태 전환 규칙:**

- `REGISTERING` → `PENDING`: PO 승인요청
- `PENDING` → `APPROVED`: BO 관리자 승인 (상품코드 자동 부여)
- `PENDING` → `REJECTED`: BO 관리자 반려 (반려사유 필수 입력)
- `PENDING` → `WITHDRAWN`: PO 승인요청 철회
- `REJECTED` → `PENDING`: PO 수정 후 재요청
- `WITHDRAWN` → `PENDING`: PO 재요청

### (3) 관계

| 관계 | 대상 엔티티 | 카디널리티 | 설명 |
|------|-------------|------------|------|
| 브랜드 | Brand | N:1 | 배송형 상품의 브랜드 |
| 판매사 | Vendor | N:1 | 상품 공급 판매사 |
| 담당 MD | Admin | N:1 | 상품 담당 MD |
| 고객사 | Client | N:1 | 서비스 이용 기업 고객 |
| 승인 이력 | ProductApprovalHistory | 1:N | 상품 승인 상태 변경 이력 |
| 판매 상태 로그 | ProductSalesStatusLog | 1:N | 판매 상태 변경 로그 |

### (4) 인덱스

| 인덱스명 | 대상 필드 | 용도 |
|----------|-----------|------|
| idx_product_sales_status | sales_status | 판매상태별 상품 목록 조회 |
| idx_product_approval_status | approval_status | 승인상태별 상품 목록 조회 |
| idx_product_brand | brand_id | 브랜드별 상품 조회 |
| idx_product_vendor | vendor_id | 판매사별 상품 조회 |
| idx_product_type | product_type | 상품 유형별 조회 |
| idx_product_tax_type | tax_type | 과세구분별 조회 |
| idx_product_md | md_id | 담당 MD별 조회 |
| idx_product_client | client_id | 고객사별 조회 |
| uq_product_code | product_code | 상품코드 유니크 제약 |

### (5) 제약 조건

- `product_code`는 승인완료(`APPROVED`) 시에만 부여되며, UNIQUE 제약을 갖는다.
- `brand_id`는 배송형 상품(`SHIPPING`)일 때 필수이다.
- `vendor_id`는 티켓형 상품(`TICKET`)일 때 필수이다.
- 승인대기(`PENDING`) 상태에서는 상품 정보 수정이 불가하다.
- 승인반려(`REJECTED`) 시 [ProductApprovalHistory](#2-productapprovalhistory-상품-승인-이력)에 `rejection_reason`이 필수 입력되어야 한다.
- `sales_end_date`는 `sales_start_date`보다 미래 시점이어야 한다.
- 승인완료(`APPROVED`) 상태의 상품만 FO에 전시된다.
- 판매종료(`SALES_ENDED`) 상품은 FO에서 비노출된다.
- `product_category`는 `product_type`에 의해 자동 결정된다.
- `product_name`은 최소 1자, 최대 200자이다.
- `rejection_reason`은 최소 1자, 최대 500자이다.
- 티켓형 상품(`TICKET`)은 `validity_period_start`, `validity_period_end`, `usage_info`, `issue_method`가 필수이다.
- `validity_period_end`는 `validity_period_start`보다 미래 시점이어야 한다.
- `reservation_info`는 `product_type`이 `TICKET_RESERVATION`일 때 필수이다.

---

## 2. ProductApprovalHistory (상품 승인 이력)

### (1) 필드 정의

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| history_id | BIGINT | Y | 이력 고유 ID | PK, Auto Increment |
| product_id | BIGINT | Y | 상품 ID | FK → Product |
| action | Enum | Y | 수행 행위 | ApprovalAction 참조 |
| previous_status | Enum | Y | 이전 승인 상태 | ApprovalStatus 참조 |
| new_status | Enum | Y | 변경 승인 상태 | ApprovalStatus 참조 |
| rejection_reason | VARCHAR(500) | C | 반려 사유 | action이 REJECT일 때 필수 |
| changed_by | VARCHAR(50) | Y | 변경자 | 관리자명 (관리자아이디) |
| changed_at | DATETIME | Y | 변경일시 | 시스템 자동 |

#### ApprovalAction (승인 행위)

| 값 | 설명 |
|----|------|
| REQUEST | 승인요청 |
| APPROVE | 승인완료 |
| REJECT | 승인반려 |
| WITHDRAW | 승인철회 |

### (2) 관계

| 관계 | 대상 엔티티 | 카디널리티 | 설명 |
|------|-------------|------------|------|
| 상품 | Product | N:1 | 승인 이력이 속한 상품 |

### (3) 제약 조건

- `rejection_reason`은 `action`이 `REJECT`일 때 필수이며, 비어 있을 수 없다.
- `new_status`는 Product의 승인 상태 전환 규칙을 따라야 한다.

---

## 3. ProductSalesStatusLog (판매 상태 변경 로그)

### (1) 필드 정의

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| log_id | BIGINT | Y | 로그 고유 ID | PK, Auto Increment |
| product_id | BIGINT | Y | 상품 ID | FK → Product |
| previous_status | Enum | Y | 이전 판매 상태 | SalesStatus 참조 |
| new_status | Enum | Y | 변경 판매 상태 | SalesStatus 참조 |
| change_type | Enum | Y | 변경 유형 | ChangeType 참조 |
| change_reason | VARCHAR(500) | N | 변경 사유 | 수동 변경 시 입력 |
| changed_by | VARCHAR(50) | C | 변경자 | MANUAL일 때 필수 |
| changed_at | DATETIME | Y | 변경일시 | 시스템 자동 |

#### ChangeType (변경 유형)

| 값 | 설명 |
|----|------|
| MANUAL | 관리자 수동 변경 |
| AUTO | 시스템 자동 변경 |

### (2) 관계

| 관계 | 대상 엔티티 | 카디널리티 | 설명 |
|------|-------------|------------|------|
| 상품 | Product | N:1 | 판매 상태 로그가 속한 상품 |

### (3) 제약 조건

- `changed_by`는 `change_type`이 `MANUAL`일 때 필수이다.
- `new_status`는 Product의 판매 상태 전환 규칙을 따라야 한다.
