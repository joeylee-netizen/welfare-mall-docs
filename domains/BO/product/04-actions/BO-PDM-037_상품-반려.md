---
id: "BO-PDM-037"
title: "상품 반려"
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
  - "BO-PDM-022"
  - "BO-PDM-023"
tags: [상품, 반려, 승인반려, API]
trigger: click
method: PATCH
endpoint: "/api/v1/products/{product_id}/reject"
---

# 상품 반려

## 트리거 조건

- 배송상품 승인 목록([BO-PDM-022](../03-components/BO-PDM-022_배송상품-승인-목록-테이블.md)) 또는 티켓상품 승인 목록([BO-PDM-023](../03-components/BO-PDM-023_티켓상품-승인-목록-테이블.md))에서 개별 행의 `반려` 버튼 클릭
- 반려사유 입력 모달에서 사유 입력 후 `반려` 클릭 시 API 호출 실행

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_id | number | Y | 반려 대상 상품 ID |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| rejection_reason | string | Y | 반려사유 (최대 500자) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 상품 ID |
| approval_status | string | `REJECTED` (승인반려) |
| rejected_at | datetime | 반려 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | EMPTY_REJECTION_REASON | 반려사유 미입력 |
| 400 | REJECTION_REASON_TOO_LONG | 반려사유 500자 초과 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | PRODUCT_NOT_FOUND | 존재하지 않는 상품 |
| 409 | INVALID_STATUS_TRANSITION | PENDING 상태가 아닌 상품 반려 시도 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "상품이 반려되었습니다."
2. 반려사유 입력 모달 닫힘
3. 해당 행의 승인상태 badge를 `REJECTED`(승인반려)로 갱신

### 실패 시

1. 오류 토스트 메시지 표시
2. 모달 유지 (재시도 가능)

---

## 유효성 검사

### 프론트엔드

- `반려` 버튼은 `PENDING` 상태 행에서만 표시/활성화
- 반려사유 입력 모달:
  - 반려 대상 상품 정보 표시 (요청코드, 상품명, 브랜드/판매사)
  - 반려사유 Textarea 필수 입력
  - 미입력 시: "반려사유를 입력해 주세요." 인라인 오류
  - 최대 500자 제한

### 백엔드

- `approval_status`가 `PENDING`인지 검증
- `rejection_reason` 필수 및 500자 이내 검증
- 승인 이력 자동 기록 (ProductApprovalHistory, action: REJECT, rejection_reason 포함)
