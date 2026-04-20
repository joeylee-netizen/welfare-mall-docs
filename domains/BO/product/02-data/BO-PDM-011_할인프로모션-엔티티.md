---
id: "BO-PDM-011"
title: "할인프로모션 엔티티"
type: data
domain: product
status: draft
version: "1.0"
created: 2026-04-20
updated: 2026-04-20
author: "기획자"
refs:
  - "BO-PDM-002"
tags: [할인프로모션, 엔티티, 데이터]
entity_name: "Promotion"
source_system: "welfare-mall-api"
---

# 할인프로모션 엔티티

## 1. Promotion (할인프로모션)

### (1) 필드 정의

#### 기본정보

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| promotion_id | BIGINT | Y | 프로모션 고유 ID | PK, Auto Increment |
| promotion_code | VARCHAR(20) | Y | 프로모션코드 | 등록 완료 시 자동 발급, UNIQUE |
| promotion_name | VARCHAR(100) | Y | 프로모션명 | 관리용 명칭 |
| promotion_type | Enum | Y | 프로모션구분 | PromotionType 참조 |
| discount_amount | DECIMAL(10,0) | Y | 할인금액 | 원 단위, 0 초과 |
| start_date | DATETIME | Y | 시작일시 | 프로모션 시작 |
| end_date | DATETIME | Y | 종료일시 | 프로모션 종료, start_date 이후 |
| coupon_applicable | BOOLEAN | Y | 쿠폰적용여부 | 기본값: true |
| status | Enum | Y | 상태 | PromotionStatus 참조, 기본값: DRAFT |
| memo | TEXT | N | 비고 | 관리자 메모 |

#### 등록/수정 이력

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| created_by | VARCHAR(50) | Y | 등록자 | 관리자명 |
| created_at | DATETIME | Y | 등록일시 | 시스템 자동 |
| updated_by | VARCHAR(50) | Y | 수정자 | 관리자명 |
| updated_at | DATETIME | Y | 수정일시 | 시스템 자동 |

### (2) Enum 정의

#### PromotionType (프로모션구분)

| 값 | 설명 | 비고 |
|----|------|------|
| MZ_VENDOR | 메가존 할인 (판매사 부담) | 할인 비용 판매사 100% |
| MZ_PLATFORM | 메가존 할인 (복지몰 부담) | 할인 비용 복지몰 100% |
| VENDOR | 판매사 할인 | 판매사 자체 프로모션 |

#### PromotionStatus (프로모션상태)

| 값 | 설명 | 비고 |
|----|------|------|
| DRAFT | 임시저장 | 등록 정보 임시 저장 |
| ACTIVE | 진행중 | 프로모션 기간 내 적용 중 |
| PAUSED | 중지 | 관리자 수동 중지 |
| ENDED | 종료 | 기간 만료 시 시스템 자동 전환 |
| DELETED | 삭제 | 소프트 삭제 |

**상태 전환 규칙:**

- `DRAFT` → `ACTIVE`: 등록 완료 (기간 도래 시)
- `ACTIVE` → `PAUSED`: 관리자 수동 중지
- `ACTIVE` → `ENDED`: 기간 만료 시 시스템 자동 전환
- `PAUSED` → `ACTIVE`: 관리자 재활성화 (기간 내)
- `DRAFT` / `PAUSED` / `ENDED` → `DELETED`: 관리자 삭제

### (3) 관계

| 관계 | 대상 엔티티 | 카디널리티 | 설명 |
|------|-------------|------------|------|
| 대상 상품 | PromotionProduct | 1:N | 프로모션에 포함된 상품 목록 |

### (4) 인덱스

| 인덱스명 | 대상 필드 | 용도 |
|----------|-----------|------|
| idx_promotion_status | status | 상태별 프로모션 목록 조회 |
| idx_promotion_type | promotion_type | 구분별 프로모션 조회 |
| idx_promotion_date | start_date, end_date | 기간별 프로모션 조회 |
| uq_promotion_code | promotion_code | 프로모션코드 유니크 제약 |

### (5) 제약 조건

- `promotion_code`는 등록 완료 시 자동 발급되며, UNIQUE 제약을 갖는다.
- `discount_amount`는 0보다 커야 한다.
- `end_date`는 `start_date`보다 미래 시점이어야 한다.
- `promotion_name`은 최소 1자, 최대 100자이다.
- 동일 상품에 대해 기간이 중복되는 프로모션은 등록할 수 없다.
- 대상 상품은 승인완료(APPROVED) 상태여야 한다.

---

## 2. PromotionProduct (프로모션 대상 상품)

### (1) 필드 정의

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| promotion_product_id | BIGINT | Y | 고유 ID | PK, Auto Increment |
| promotion_id | BIGINT | Y | 프로모션 ID | FK → Promotion |
| product_id | BIGINT | Y | 상품 ID | FK → Product |

### (2) 관계

| 관계 | 대상 엔티티 | 카디널리티 | 설명 |
|------|-------------|------------|------|
| 프로모션 | Promotion | N:1 | 프로모션에 속한 대상 상품 |
| 상품 | Product | N:1 | 프로모션 대상 상품 |

### (3) 인덱스

| 인덱스명 | 대상 필드 | 용도 |
|----------|-----------|------|
| idx_pp_promotion | promotion_id | 프로모션별 대상 상품 조회 |
| idx_pp_product | product_id | 상품별 프로모션 조회 |
| uq_pp_unique | promotion_id, product_id | 중복 등록 방지 |

### (4) 제약 조건

- `(promotion_id, product_id)` 조합은 UNIQUE 제약을 갖는다.
- `product_id`는 승인완료(APPROVED) 상태인 상품만 등록 가능하다.
