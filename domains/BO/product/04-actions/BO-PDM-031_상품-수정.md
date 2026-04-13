---
id: "BO-PDM-031"
title: "상품 수정"
type: action
domain: product
status: draft
version: "1.0"
created: 2026-04-13
updated: 2026-04-13
author: "기획자"
refs:
  - "BO-PDM-001"
  - "BO-PDM-010"
  - "BO-PDM-020"
  - "BO-PDM-021"
tags: [상품, 수정, 저장, API]
trigger: submit
method: PUT
endpoint: "/api/v1/products/{product_id}"
---

# 상품 수정

## 트리거 조건

- 배송상품 등록/수정 폼([BO-PDM-020](../03-components/BO-PDM-020_배송상품-등록수정-폼.md)) 또는 티켓상품 등록/수정 폼([BO-PDM-021](../03-components/BO-PDM-021_티켓상품-등록수정-폼.md))에서 **수정 모드** 상태
- 승인상태가 `REGISTERING`, `REJECTED`, `APPROVED` 중 하나일 때
- 수정 후 `저장` 버튼 클릭
- 확인 모달에서 `확인` 클릭 시 API 호출 실행

---

## 사전 호출

폼 진입(수정 모드) 시 상품 상세 조회를 통해 기존 데이터를 로드한다.

**GET** `/api/v1/products/{product_id}`

### 상세 조회 응답

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 상품 ID |
| product_code | string | 상품코드 (APPROVED 시 존재) |
| request_code | string | 요청코드 |
| product_category | string | 상품 카테고리 |
| product_type | string | 상품유형 |
| product_name | string | 상품명 |
| brand_id | number | 브랜드 ID |
| vendor_id | number | 판매사 ID |
| approval_status | string | 승인상태 |
| sales_status | string | 판매상태 |
| sales_start_date | date | 판매시작일 |
| sales_end_date | date | 판매종료일 |
| shipping_info | string | 배송 안내 |
| installation_info | string | 설치 안내 |
| validity_period_start | date | 유효기간 시작일 |
| validity_period_end | date | 유효기간 종료일 |
| usage_info | string | 이용 안내 |
| issue_method | string | 발급 수단 |
| reservation_info | string | 예약 정보 |
| approval_histories | array | 승인 이력 목록 |

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_id | number | Y | 수정 대상 상품 ID |

### Body (JSON)

편집 범위는 승인상태에 따라 달라진다.

**REGISTERING / REJECTED 상태 (전체 편집):**

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_type | string | Y | 상품유형 |
| product_name | string | Y | 상품명 (최대 200자) |
| brand_id | number | C | 브랜드 ID (배송형 필수) |
| vendor_id | number | C | 판매사 ID (티켓형 필수) |
| shipping_info | string | C | 배송 안내 (배송형 필수) |
| installation_info | string | C | 설치 안내 (INSTALLATION 유형 필수) |
| validity_period_start | date | C | 유효기간 시작일 (티켓형 필수) |
| validity_period_end | date | C | 유효기간 종료일 (티켓형 필수) |
| usage_info | string | C | 이용 안내 (티켓형 필수) |
| issue_method | string | C | 발급 수단 (티켓형 필수) |
| reservation_info | string | C | 예약 정보 (TICKET_RESERVATION 필수) |

**APPROVED 상태 (제한 편집):**

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_name | string | Y | 상품명 (최대 200자) |
| sales_status | string | N | 판매상태 |
| sales_start_date | date | N | 판매시작일 |
| sales_end_date | date | N | 판매종료일 |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 수정된 상품 ID |
| approval_status | string | 현재 승인상태 (변경 없음) |
| updated_at | datetime | 수정 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 필수 파라미터 누락 또는 유효성 검증 실패 |
| 400 | INVALID_SALES_STATUS_TRANSITION | 허용되지 않는 판매상태 전환 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | PRODUCT_NOT_FOUND | 존재하지 않는 상품 |
| 409 | EDIT_NOT_ALLOWED | 현재 상태에서 수정 불가 (PENDING 상태) |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

**APPROVED 상태 수정:**
1. 성공 토스트: "수정 사항이 저장되었습니다. FO에 즉시 반영됩니다."
2. 폼 데이터 갱신 (응답 데이터 반영)

**그 외 상태 수정:**
1. 성공 토스트: "수정 사항이 저장되었습니다."
2. 폼 데이터 갱신

### 실패 시

1. 오류 토스트 메시지 표시
2. 폼 데이터 유지

---

## 유효성 검사

### 프론트엔드

- 등록 액션([BO-PDM-030](./BO-PDM-030_상품-등록.md))과 동일한 필드 검증
- APPROVED 상태: 판매상태 변경 규칙 검증 ([BO-PDM-001](../01-policies/BO-PDM-001_상품-관리-정책.md) 참조)
  - 판매중 → 판매일시중지, 판매종료만 허용
  - 판매일시중지 / 일시품절 / 품절 → 판매중만 허용
- `sales_end_date` ≥ `sales_start_date` 검증

### 백엔드

- 프론트엔드 검증 항목 동일 적용
- `approval_status = PENDING` 시 수정 거부 (409)
- APPROVED 상태에서 편집 범위 외 필드 변경 시 무시 또는 거부
- 판매상태 전환 규칙 서버 측 검증
