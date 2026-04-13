---
id: "BO-PDM-036"
title: "상품 일괄승인"
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
tags: [상품, 일괄승인, 배치, API]
trigger: click
method: PATCH
endpoint: "/api/v1/products/batch-approve"
---

# 상품 일괄승인

## 트리거 조건

- 배송상품 승인 목록([BO-PDM-022](../03-components/BO-PDM-022_배송상품-승인-목록-테이블.md)) 또는 티켓상품 승인 목록([BO-PDM-023](../03-components/BO-PDM-023_티켓상품-승인-목록-테이블.md))에서 체크박스로 1건 이상 선택
- `일괄 승인` 버튼 클릭
- 승인 확인 모달에서 `승인` 클릭 시 API 호출 실행

---

## 요청 (Request)

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_ids | number[] | Y | 승인 대상 상품 ID 배열 (1건 이상) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| total_count | number | 요청 건수 |
| success_count | number | 승인 성공 건수 |
| fail_count | number | 승인 실패 건수 |
| results | array | 개별 처리 결과 |
| results[].product_id | number | 상품 ID |
| results[].product_code | string | 부여된 상품코드 (성공 시) |
| results[].status | string | `success` \| `fail` |
| results[].error_code | string | 실패 사유 코드 (실패 시) |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | EMPTY_PRODUCT_IDS | 상품 ID 배열이 비어 있음 |
| 403 | FORBIDDEN | 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 전체 성공 시

1. 성공 토스트: "N건의 상품이 승인되었습니다."
2. 승인 확인 모달 닫힘
3. 목록 재조회 (현재 필터 조건 유지)
4. 체크박스 선택 상태 초기화

### 부분 성공 시

1. 경고 토스트: "N건 승인, M건 실패. 실패 건은 상태를 확인해 주세요."
2. 모달 닫힘
3. 목록 재조회

### 전체 실패 시

1. 오류 토스트: "일괄 승인에 실패했습니다."
2. 모달 유지 (재시도 가능)

---

## 유효성 검사

### 프론트엔드

- `일괄 승인` 버튼은 1건 이상 체크 시에만 활성화
- 체크박스는 `PENDING` 상태 행에서만 활성화
- 승인 확인 모달: "선택한 N건의 {상품유형}상품을 승인하시겠습니까? 승인 시 상품코드가 부여됩니다."

### 백엔드

- `product_ids` 배열이 1건 이상인지 검증
- 각 상품의 `approval_status`가 `PENDING`인지 개별 검증
- PENDING이 아닌 상품은 개별 실패 처리 (나머지는 정상 승인)
- 상품코드 자동 채번 및 부여 (성공 건별)
- 승인 이력 자동 기록 (ProductApprovalHistory, 건별)
- 판매상태 초기값 `SELLING`(판매중) 설정 (건별)
