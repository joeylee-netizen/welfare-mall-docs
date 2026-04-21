---
id: "BO-CLM-035"
title: "주문취소 승인"
type: action
domain: claim
status: draft
version: "1.0"
created: 2026-04-21
updated: 2026-04-21
author: "기획자"
refs:
  - "BO-CLM-002"
  - "BO-CLM-010"
  - "BO-CLM-021"
  - "BO-CLM-022"
tags: [클레임, 주문취소, 승인, 상품준비중, API]
trigger: click
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/approve"
---

# 주문취소 승인

## 트리거 조건

- [BO-CLM-021 주문취소 목록](../03-components/BO-CLM-021_주문취소-목록-테이블.md)에서 개별 행 `승인` 버튼 클릭 + 확인 모달 `승인`
- 목록의 `일괄 승인` 버튼 클릭 + 확인 모달 `승인` (각 선택 행에 대해 반복 호출)
- [BO-CLM-022 주문취소 상세 폼](../03-components/BO-CLM-022_주문취소-상세-폼.md)의 `승인` 버튼 클릭
- 대상: `claim_status_bo = CANCEL_REQUESTED` + `approval_required = true` (상품준비중 단계)

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body

없음(상태 전환만 수행).

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_status_bo | string | `REFUND_REQUESTED` (환불요청 완료로 전환) |
| approved_by_id | number | 승인자 관리자 ID |
| approved_at | datetime | 승인 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 승인 권한 없음 또는 타 고객사 건 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `CANCEL_REQUESTED` + `approval_required = true` 아님 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "주문취소가 승인되었습니다. 환불 처리가 진행됩니다."
2. 해당 행의 `claim_status_bo`를 `REFUND_REQUESTED`로 갱신, `approval_required` false로 갱신
3. 상세 팝업에서 호출 시: 상세 재조회(`[BO-CLM-032](./BO-CLM-032_주문취소-상세-조회.md)`) + 하단 버튼 영역 리렌더링
4. 시스템이 환불 프로세스 자동 시작(RefundRecord 생성 → PG/포인트 API 호출)
5. ClaimHistory에 `(CANCEL_REQUESTED → REFUND_REQUESTED)` + 처리자 기록

### 실패 시

1. 오류 토스트 표시
2. 상태 충돌(409) 시 목록·상세 자동 새로고침

### 일괄 처리 시

- 각 건에 대해 개별 호출. 응답을 모아 `성공 N건 / 실패 M건` 토스트 요약 표시.
- 실패 건 claim_code 목록을 별도 영역(펼침)에 표시.

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = CANCEL_REQUESTED` + `approval_required = true`
- 확인 모달 본문: "본 건을 승인하시겠습니까? 승인 시 즉시 환불요청이 진행됩니다."

### 백엔드

- `claim_status_bo = CANCEL_REQUESTED`인지 검증
- `approval_required = true`인지 검증
- 관리자 권한 범위 내 `client_id` 검증
- 승인 즉시 환불 프로세스 비동기 트리거
- 승인 이력(`ClaimHistory`) 기록, `processor_id` = 승인자
