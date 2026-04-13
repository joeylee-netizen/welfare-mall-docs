---
id: "BO-PDM-035"
title: "상품 승인"
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
tags: [상품, 승인, 개별승인, API]
trigger: click
method: PATCH
endpoint: "/api/v1/products/{product_id}/approve"
---

# 상품 승인

## 트리거 조건

- 배송상품 승인 목록([BO-PDM-022](../03-components/BO-PDM-022_배송상품-승인-목록-테이블.md)) 또는 티켓상품 승인 목록([BO-PDM-023](../03-components/BO-PDM-023_티켓상품-승인-목록-테이블.md))에서 개별 행의 `승인` 버튼 클릭
- 승인 확인 모달에서 `승인` 클릭 시 API 호출 실행

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_id | number | Y | 승인 대상 상품 ID |

### Body

없음 (상태 전환만 수행)

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 상품 ID |
| product_code | string | 자동 부여된 상품코드 |
| approval_status | string | `APPROVED` (승인완료) |
| approved_at | datetime | 승인 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 권한 없음 |
| 404 | PRODUCT_NOT_FOUND | 존재하지 않는 상품 |
| 409 | INVALID_STATUS_TRANSITION | PENDING 상태가 아닌 상품 승인 시도 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "상품이 승인되었습니다."
2. 승인 확인 모달 닫힘
3. 해당 행의 승인상태 badge를 `APPROVED`(승인완료)로 갱신
4. 상품코드가 부여됨을 사용자에게 안내

### 실패 시

1. 오류 토스트 메시지 표시
2. 모달 유지 (재시도 가능)

---

## 유효성 검사

### 프론트엔드

- `승인` 버튼은 `PENDING` 상태 행에서만 표시/활성화
- 승인 확인 모달: "선택한 1건의 {상품유형}상품을 승인하시겠습니까? 승인 시 상품코드가 부여됩니다."

### 백엔드

- `approval_status`가 `PENDING`인지 검증
- 상품코드 자동 채번 및 부여
- 승인 이력 자동 기록 (ProductApprovalHistory)
- 판매상태 초기값 `SELLING`(판매중) 설정
