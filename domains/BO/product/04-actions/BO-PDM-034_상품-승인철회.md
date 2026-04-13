---
id: "BO-PDM-034"
title: "상품 승인철회"
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
tags: [상품, 승인철회, API]
trigger: click
method: PATCH
endpoint: "/api/v1/products/{product_id}/withdraw-approval"
---

# 상품 승인철회

## 트리거 조건

- 배송상품 등록/수정 폼([BO-PDM-020](../03-components/BO-PDM-020_배송상품-등록수정-폼.md)) 또는 티켓상품 등록/수정 폼([BO-PDM-021](../03-components/BO-PDM-021_티켓상품-등록수정-폼.md))에서 **조회 모드(PENDING)** 상태
- `승인철회` 버튼 클릭
- 확인 모달에서 `확인` 클릭 시 API 호출 실행

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_id | number | Y | 승인철회 대상 상품 ID |

### Body

없음 (상태 전환만 수행)

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 상품 ID |
| approval_status | string | `WITHDRAWN` (승인철회) |
| withdrawn_at | datetime | 승인철회 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 권한 없음 |
| 404 | PRODUCT_NOT_FOUND | 존재하지 않는 상품 |
| 409 | INVALID_STATUS_TRANSITION | 현재 상태에서 승인철회 불가 (PENDING 외) |
| 409 | ALREADY_PROCESSED | 이미 승인/반려 처리된 상품 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "승인요청이 철회되었습니다."
2. 조회 모드(PENDING) → 수정 모드로 전환 (WITHDRAWN은 REGISTERING과 동일 편집 범위)
3. 전체 필드 편집 가능 상태로 전환
4. 하단 버튼을 수정(REGISTERING) 모드 구성으로 변경: `삭제`, `저장`, `승인요청`
5. 승인이력 테이블에 새 이력 추가 (WITHDRAW 행위)

### 실패 시

1. 오류 토스트 메시지 표시
2. 현재 조회 모드 유지

---

## 유효성 검사

### 프론트엔드

- `승인철회` 버튼은 `PENDING` 상태에서만 표시
- 확인 모달: "승인요청을 철회하시겠습니까? 철회 후 수정 및 삭제가 가능합니다."

### 백엔드

- `approval_status`가 `PENDING`인지 검증
- 이미 승인/반려 처리된 경우 409 반환
- 승인 이력 자동 기록 (ProductApprovalHistory)
