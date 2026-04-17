---
id: "BO-PDM-033"
title: "상품 승인요청"
type: action
domain: product
status: draft
version: "1.1"
created: 2026-04-13
updated: 2026-04-17
author: "기획자"
refs:
  - "BO-PDM-001"
  - "BO-PDM-010"
  - "BO-PDM-020"
  - "BO-PDM-021"
tags: [상품, 승인요청, 재승인요청, API]
trigger: click
method: PATCH
endpoint: "/api/v1/products/{product_id}/request-approval"
---

# 상품 승인요청

## 트리거 조건

- 배송상품 등록/수정 폼([BO-PDM-020](../03-components/BO-PDM-020_배송상품-등록수정-폼.md)) 또는 티켓상품 등록/수정 폼([BO-PDM-021](../03-components/BO-PDM-021_티켓상품-등록수정-폼.md))에서 **수정 모드** 상태
- **승인요청:** 승인상태가 `REGISTERING`일 때 `승인요청` 버튼 클릭
- **재승인요청:** 승인상태가 `REJECTED`일 때 `재승인요청` 버튼 클릭
- 확인 모달에서 `확인` 클릭 시 API 호출 실행

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_id | number | Y | 승인요청 대상 상품 ID |

### Body

없음 (상태 전환만 수행)

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 상품 ID |
| approval_status | string | `PENDING` (승인대기) |
| requested_at | datetime | 승인요청 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INCOMPLETE_PRODUCT | 필수 정보 미입력 상태에서 승인요청 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | PRODUCT_NOT_FOUND | 존재하지 않는 상품 |
| 409 | INVALID_STATUS_TRANSITION | 현재 상태에서 승인요청 불가 (REGISTERING/REJECTED 외) |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "승인 요청이 완료되었습니다."
2. 수정 모드 → 조회 모드(PENDING)로 전환
3. 모든 필드 읽기전용 전환
4. 하단 버튼을 조회(PENDING) 모드 구성으로 변경: `승인철회`
5. 승인이력 테이블에 새 이력 추가 (REQUEST 행위)

### 실패 시

1. 오류 토스트 메시지 표시
2. 현재 모드 유지

---

## 유효성 검사

### 프론트엔드

- `승인요청` 버튼은 `REGISTERING` 상태에서만 표시
- `재승인요청` 버튼은 `REJECTED` 상태에서만 표시
- 승인요청 모달:
  - REGISTERING: "상품 승인을 요청하시겠습니까? 승인대기 상태에서는 수정할 수 없습니다."
  - REJECTED: "수정된 상품의 승인을 재요청하시겠습니까?"

### 백엔드

- `approval_status`가 `REGISTERING` 또는 `REJECTED`인지 검증
- `is_temp_saved`가 `true`인 경우 승인요청 거부 (정식 저장 필수)
- 상품 필수 필드 완전성 검증 (미입력 필드가 있으면 `INCOMPLETE_PRODUCT` 오류 반환)
- 승인 이력 자동 기록 (ProductApprovalHistory)

### INCOMPLETE_PRODUCT 검증 필드 목록

승인요청 시 다음 필드가 모두 입력되어 있어야 한다. 하나라도 누락 시 `400 INCOMPLETE_PRODUCT` 오류를 반환한다.

#### 공통 필수 필드

| 필드명 | 설명 |
|--------|------|
| product_name | 상품명 (1자 이상) |
| product_type | 상품 유형 |
| sales_status | 판매 상태 |
| tax_type | 과세구분 |
| display_channel | 노출채널 |
| is_displayed | 전시여부 |

#### 배송형 추가 필수 필드

| 필드명 | 설명 | 조건 |
|--------|------|------|
| brand_id | 브랜드 | product_category = SHIPPING |

#### 티켓형 추가 필수 필드

| 필드명 | 설명 | 조건 |
|--------|------|------|
| vendor_id | 판매사 | product_category = TICKET |
| validity_period_start | 유효기간 시작일 | product_category = TICKET |
| validity_period_end | 유효기간 종료일 | product_category = TICKET |
| usage_info | 이용 안내 | product_category = TICKET |
| issue_method | 발급 수단 | product_category = TICKET |
