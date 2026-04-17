---
id: "BO-PDM-03B"
title: "상품 일괄반려"
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
  - "BO-PDM-022"
  - "BO-PDM-023"
  - "BO-PDM-037"
tags: [상품, 일괄반려, 배치, API]
trigger: click
method: PATCH
endpoint: "/api/v1/products/batch-reject"
---

# 상품 일괄반려

## 트리거 조건

- 배송상품 승인 목록([BO-PDM-022](../03-components/BO-PDM-022_배송상품-승인-목록-테이블.md)) 또는 티켓상품 승인 목록([BO-PDM-023](../03-components/BO-PDM-023_티켓상품-승인-목록-테이블.md))에서 체크박스로 1건 이상 선택
- `일괄 반려` 버튼 클릭
- 반려 사유 입력 모달에서 사유 입력 후 `반려` 클릭 시 API 호출 실행

---

## 요청 (Request)

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_ids | number[] | Y | 반려 대상 상품 ID 배열 (1건 이상) |
| rejection_reason | string | Y | 반려사유 (최소 1자, 최대 500자) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| total_count | number | 요청 건수 |
| success_count | number | 반려 성공 건수 |
| fail_count | number | 반려 실패 건수 |
| results | array | 개별 처리 결과 |
| results[].product_id | number | 상품 ID |
| results[].status | string | `success` \| `fail` |
| results[].error_code | string | 실패 사유 코드 (실패 시) |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | EMPTY_PRODUCT_IDS | 상품 ID 배열이 비어 있음 |
| 400 | EMPTY_REJECTION_REASON | 반려사유 미입력 |
| 400 | REJECTION_REASON_TOO_LONG | 반려사유 500자 초과 |
| 403 | FORBIDDEN | 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 전체 성공 시

1. 성공 토스트: "N건의 상품이 반려되었습니다."
2. 반려사유 입력 모달 닫힘
3. 목록 재조회 (현재 필터 조건 유지)
4. 체크박스 선택 상태 초기화
5. 검색결과 건수 갱신

### 부분 성공 시

1. 경고 토스트: "N건 반려, M건 실패. 실패 건은 상태를 확인해 주세요."
2. 모달 닫힘
3. 목록 재조회

### 전체 실패 시

1. 오류 토스트: "일괄 반려에 실패했습니다."
2. 모달 유지 (재시도 가능)

---

## 유효성 검사

### 프론트엔드

- `일괄 반려` 버튼은 1건 이상 체크 시에만 활성화
- 체크박스는 전체 행에서 선택 가능하나, PENDING 상태만 반려 대상으로 처리
- 반려사유 입력 모달:
  - "선택한 N건의 {상품유형}상품을 반려하시겠습니까?"
  - 반려사유 Textarea 필수 입력
  - 미입력 시: "반려사유를 입력해 주세요." 인라인 오류
  - 최대 500자 제한

### 백엔드

- `product_ids` 배열이 1건 이상인지 검증
- `rejection_reason` 필수 및 1자~500자 검증
- 각 상품의 `approval_status`가 `PENDING`인지 개별 검증
- PENDING이 아닌 상품은 개별 실패 처리 (나머지는 정상 반려)
- 승인 이력 자동 기록 (ProductApprovalHistory, action: REJECT, rejection_reason 포함, 건별)
