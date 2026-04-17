---
id: "BO-PDM-03A"
title: "상품 임시저장"
type: action
domain: product
status: draft
version: "1.0"
created: 2026-04-17
updated: 2026-04-17
author: "기획자"
refs:
  - "BO-PDM-001"
  - "BO-PDM-010"
  - "BO-PDM-020"
  - "BO-PDM-021"
  - "BO-PDM-030"
tags: [상품, 임시저장, API]
trigger: click
method: "POST / PATCH"
endpoint: "POST /api/v1/products/temp-save | PATCH /api/v1/products/{product_id}/temp-save"
---

# 상품 임시저장

## 트리거 조건

- 배송상품 등록/수정 폼([BO-PDM-020](../03-components/BO-PDM-020_배송상품-등록수정-폼.md)) 또는 티켓상품 등록/수정 폼([BO-PDM-021](../03-components/BO-PDM-021_티켓상품-등록수정-폼.md))에서 `임시저장` 버튼 클릭
- 승인상태가 `REGISTERING`인 경우에만 버튼 노출
- 승인완료(`APPROVED`) 상태에서는 임시저장 버튼 비노출

---

## 요청 (Request)

### 신규 임시저장 (POST)

**POST** `/api/v1/products/temp-save`

#### Body (JSON)

[BO-PDM-030](./BO-PDM-030_상품-등록.md)과 동일한 필드 구조이나, **필수 필드 검증을 생략**한다.

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_type | string | N | 상품 유형 |
| product_category | string | N | 상품 구분 (SHIPPING/TICKET) |
| product_name | string | N | 상품명 |
| ... | ... | N | 기타 상품 정보 필드 (모두 선택) |

### 수정 임시저장 (PATCH)

**PATCH** `/api/v1/products/{product_id}/temp-save`

#### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_id | number | Y | 임시저장 대상 상품 ID |

#### Body (JSON)

변경된 필드만 전송 (Partial Update). 필수 필드 검증 생략.

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 상품 ID |
| request_code | string | 요청코드 (신규 시 자동 발번) |
| is_temp_saved | boolean | `true` |
| saved_at | datetime | 임시저장 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 권한 없음 |
| 404 | PRODUCT_NOT_FOUND | 존재하지 않는 상품 (수정 시) |
| 409 | INVALID_STATUS_TRANSITION | REGISTERING 상태가 아닌 상품에서 임시저장 시도 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "임시 저장되었습니다."
2. 현재 화면 유지 (등록 모드 → product_id 유지)
3. `is_temp_saved = true` 상태 반영
4. 신규 임시저장 시: URL에 product_id 파라미터 추가 (이후 PATCH 모드 전환)

### 실패 시

1. 오류 토스트 메시지 표시
2. 현재 화면 유지

---

## 유효성 검사

### 프론트엔드

- `임시저장` 버튼은 `REGISTERING` 상태에서만 표시
- `APPROVED` 상태에서는 비노출
- 필수 필드 미입력 경고 없이 저장 가능

### 백엔드

- `approval_status`가 `REGISTERING`인지 검증
- 필수 필드 검증 **생략** (임시저장 특성)
- `is_temp_saved = true`로 설정
- 요청코드 자동 발번 (신규 시)
