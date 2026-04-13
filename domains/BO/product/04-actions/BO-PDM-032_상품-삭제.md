---
id: "BO-PDM-032"
title: "상품 삭제"
type: action
domain: product
status: draft
version: "1.0"
created: 2026-04-13
updated: 2026-04-13
author: "기획자"
refs:
  - "BO-PDM-001"
  - "BO-PDM-020"
  - "BO-PDM-021"
tags: [상품, 삭제, API]
trigger: click
method: DELETE
endpoint: "/api/v1/products/{product_id}"
---

# 상품 삭제

## 트리거 조건

- 배송상품 등록/수정 폼([BO-PDM-020](../03-components/BO-PDM-020_배송상품-등록수정-폼.md)) 또는 티켓상품 등록/수정 폼([BO-PDM-021](../03-components/BO-PDM-021_티켓상품-등록수정-폼.md))에서 **수정 모드** 상태
- 승인상태가 `REGISTERING` 또는 `WITHDRAWN`일 때만 삭제 가능
- `삭제` 버튼 클릭
- 확인 모달(Danger)에서 `확인` 클릭 시 API 호출 실행

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_id | number | Y | 삭제 대상 상품 ID |

### Body

없음

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 삭제된 상품 ID |
| deleted | boolean | `true` |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 권한 없음 |
| 404 | PRODUCT_NOT_FOUND | 존재하지 않는 상품 |
| 409 | DELETE_NOT_ALLOWED | 현재 상태에서 삭제 불가 (REGISTERING/WITHDRAWN 외 상태) |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "상품이 삭제되었습니다."
2. 상품 목록 페이지로 이동:
   - 배송형: 배송상품 목록([BO-PDM-024](../03-components/BO-PDM-024_배송상품-목록-테이블.md))
   - 티켓형: 티켓상품 목록([BO-PDM-025](../03-components/BO-PDM-025_티켓상품-목록-테이블.md))

### 실패 시

1. 오류 토스트 메시지 표시
2. 현재 폼 유지

---

## 유효성 검사

### 프론트엔드

- `삭제` 버튼은 `REGISTERING` / `WITHDRAWN` 상태에서만 표시
- 확인 모달 필수: "이 상품을 삭제하시겠습니까? 삭제된 상품은 복구할 수 없습니다."

### 백엔드

- `approval_status`가 `REGISTERING` 또는 `WITHDRAWN`인지 검증
- 그 외 상태에서 삭제 시도 시 409 반환
- 삭제는 soft delete (논리 삭제) 처리
