---
id: "BO-PDM-03E"
title: "할인프로모션 중지·삭제"
type: action
domain: product
status: draft
version: "1.0"
created: 2026-04-20
updated: 2026-04-20
author: "기획자"
refs:
  - "BO-PDM-002"
  - "BO-PDM-011"
  - "BO-PDM-026"
tags: [할인프로모션, 중지, 삭제, API]
trigger: click
method: PATCH
endpoint: "/api/v1/promotions/{promotion_code}/status"
---

# 할인프로모션 중지·삭제

## 트리거 조건

- 할인프로모션 목록 테이블([BO-PDM-026](../03-components/BO-PDM-026_할인프로모션-목록-테이블.md))에서 체크박스 선택 후 `중지` 또는 `삭제` 버튼 클릭
- 확인 모달에서 `확인` 클릭 시 API 호출 실행

---

## 1. 중지 (ACTIVE → PAUSED)

### 요청 (Request)

```
PATCH /api/v1/promotions/{promotion_code}/status
```

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| action | string | Y | `PAUSE` |

### 전제 조건

- 대상 프로모션 상태가 `ACTIVE`여야 한다.

### 처리

1. 프로모션 상태를 `PAUSED`로 변경
2. FO 할인 즉시 해제 (원래 판매가로 복원)

### 일괄 처리

- 체크박스로 복수 프로모션 선택 후 `중지` 버튼 클릭
- `POST /api/v1/promotions/batch-status` 엔드포인트 사용

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| promotion_codes | string[] | Y | 대상 프로모션코드 배열 |
| action | string | Y | `PAUSE` |

---

## 2. 삭제 (DRAFT/PAUSED/ENDED → DELETED)

### 요청 (Request)

```
PATCH /api/v1/promotions/{promotion_code}/status
```

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| action | string | Y | `DELETE` |

### 전제 조건

- 대상 프로모션 상태가 `DRAFT`, `PAUSED`, `ENDED` 중 하나여야 한다.
- `ACTIVE` 상태의 프로모션은 삭제 불가 (중지 후 삭제).

### 처리

1. 프로모션 상태를 `DELETED`로 변경 (soft delete)
2. 프로모션 대상 상품 매핑 데이터 유지 (이력 보존)

### 일괄 처리

- 체크박스로 복수 프로모션 선택 후 `삭제` 버튼 클릭
- `POST /api/v1/promotions/batch-status` 엔드포인트 사용

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| promotion_codes | string[] | Y | 대상 프로모션코드 배열 |
| action | string | Y | `DELETE` |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| success_count | number | 처리 성공 건수 |
| fail_count | number | 처리 실패 건수 |
| results | array | 개별 처리 결과 |
| results[].promotion_code | string | 프로모션코드 |
| results[].status | string | 변경된 상태 |
| results[].success | boolean | 처리 성공 여부 |
| results[].error | string | 실패 시 오류 메시지 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_ACTION | 유효하지 않은 액션 |
| 400 | INVALID_STATUS_TRANSITION | 현재 상태에서 불가한 전환 (예: ACTIVE → DELETE) |
| 404 | NOT_FOUND | 존재하지 않는 프로모션코드 |
| 401 | UNAUTHORIZED | 인증 실패 |
| 403 | FORBIDDEN | 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 중지 성공 시

1. 성공 토스트: "N건의 프로모션이 중지되었습니다."
2. 목록 자동 갱신 (중지된 프로모션 상태 badge 변경)
3. 체크박스 선택 해제

### 삭제 성공 시

1. 성공 토스트: "N건의 프로모션이 삭제되었습니다."
2. 목록 자동 갱신 (삭제된 프로모션 목록에서 제거)
3. 체크박스 선택 해제

### 실패 시

1. 오류 토스트 메시지 표시
2. 일괄 처리 시 부분 실패: "N건 성공, M건 실패" 토스트 + 실패 건 상세 표시
