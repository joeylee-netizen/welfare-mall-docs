---
id: "BO-PDM-030"
title: "상품 등록"
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
tags: [상품, 등록, 저장, API]
trigger: submit
method: POST
endpoint: "/api/v1/products"
---

# 상품 등록

## 트리거 조건

- 배송상품 등록/수정 폼([BO-PDM-020](../03-components/BO-PDM-020_배송상품-등록수정-폼.md)) 또는 티켓상품 등록/수정 폼([BO-PDM-021](../03-components/BO-PDM-021_티켓상품-등록수정-폼.md))에서 **등록 모드** 상태
- 모든 필수 필드 입력 후 `저장` 버튼 클릭
- 확인 모달에서 `확인` 클릭 시 API 호출 실행

---

## 사전 호출

없음 (등록 모드는 빈 폼으로 진입)

---

## 요청 (Request)

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_category | string | Y | 상품 카테고리 (`SHIPPING` \| `TICKET`) |
| product_type | string | Y | 상품유형 (`GENERAL_SHIPPING`, `INSTALLATION`, `TICKET_COUPON`, `TICKET_RESERVATION`) |
| product_name | string | Y | 상품명 (최대 200자) |
| brand_id | number | C | 브랜드 ID (배송형 필수) |
| vendor_id | number | C | 판매사 ID (티켓형 필수) |
| shipping_info | string | C | 배송 안내 (배송형 필수) |
| installation_info | string | C | 설치 안내 (INSTALLATION 유형 필수) |
| validity_period_start | date | C | 유효기간 시작일 (티켓형 필수) |
| validity_period_end | date | C | 유효기간 종료일 (티켓형 필수) |
| usage_info | string | C | 이용 안내 (티켓형 필수) |
| issue_method | string | C | 발급 수단 (티켓형 필수): `MMS`, `ALIMTALK`, `APP_PUSH` |
| reservation_info | string | C | 예약 정보 (TICKET_RESERVATION 유형 필수) |

> **C** = 조건부 필수. `product_category`와 `product_type`에 따라 결정.

---

## 응답 (Response)

### 성공 (201 Created)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 생성된 상품 ID |
| request_code | string | 자동 발번된 요청코드 |
| approval_status | string | `REGISTERING` (상품등록중) |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 필수 파라미터 누락 또는 유효성 검증 실패 |
| 400 | PRODUCT_NAME_TOO_LONG | 상품명 200자 초과 |
| 400 | INVALID_BRAND | 존재하지 않는 브랜드 ID |
| 400 | INVALID_VENDOR | 존재하지 않는 판매사 ID |
| 401 | UNAUTHORIZED | 인증 실패 |
| 403 | FORBIDDEN | 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트 표시: "상품이 등록되었습니다."
2. 등록 모드 → 수정 모드로 전환 (반환된 `product_id` 기반)
3. 요청코드(`request_code`) 필드 표시
4. 승인상태 `REGISTERING` badge 표시
5. 하단 버튼을 수정(REGISTERING) 모드 구성으로 변경: `삭제`, `저장`, `승인요청`

### 실패 시

1. 오류 토스트 메시지 표시
2. 폼 데이터 유지 (입력값 손실 방지)
3. 모달 닫힘

---

## 유효성 검사

### 프론트엔드 (모달 표시 전)

**공통:**
- `product_type`: 미선택 불가
- `product_name`: 빈 값 불가, 200자 이내

**배송형 (`product_category = SHIPPING`):**
- `brand_id`: 미선택 불가
- `shipping_info`: 빈 값 불가
- `installation_info`: `product_type = INSTALLATION`일 때 빈 값 불가

**티켓형 (`product_category = TICKET`):**
- `vendor_id`: 미선택 불가
- `validity_period_start`, `validity_period_end`: 모두 필수, 종료일 ≥ 시작일
- `usage_info`: 빈 값 불가
- `issue_method`: 미선택 불가
- `reservation_info`: `product_type = TICKET_RESERVATION`일 때 빈 값 불가

### 백엔드

- 프론트엔드 검증 항목 동일 적용
- `brand_id` / `vendor_id` 존재 여부 검증
- `product_type`과 `product_category` 조합 정합성 검증
