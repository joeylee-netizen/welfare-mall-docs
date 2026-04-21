---
id: "BO-CLM-03A"
title: "반품/교환 승인"
type: action
domain: claim
status: draft
version: "1.0"
created: 2026-04-22
updated: 2026-04-22
author: "기획자"
refs:
  - "BO-CLM-003"
  - "BO-CLM-004"
  - "BO-CLM-010"
  - "BO-CLM-023"
  - "BO-CLM-024"
  - "BO-CLM-025"
  - "BO-CLM-026"
tags: [클레임, 반품, 교환, 승인, 접수, API]
trigger: click
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/receive"
---

# 반품/교환 승인

## 트리거 조건

- [BO-CLM-023 반품 목록](../03-components/BO-CLM-023_반품-목록-테이블.md) / [BO-CLM-025 교환 목록](../03-components/BO-CLM-025_교환-목록-테이블.md)에서 개별 행 `승인` 버튼 클릭
- 목록의 `일괄 승인` 버튼 클릭 → 선택 행에 대해 반복 호출
- [BO-CLM-024 반품 상세](../03-components/BO-CLM-024_반품-상세-폼.md) / [BO-CLM-026 교환 상세](../03-components/BO-CLM-026_교환-상세-폼.md)의 `승인` 버튼
- 대상 상태: `RETURN_REQUESTED` 또는 `EXCHANGE_REQUESTED`

반품·교환 공용 Action. `claim_id`의 `claim_type`에 따라 내부에서 상태 전이 대상이 결정된다.

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body

없음 (상태 전환만 수행).

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_type | string | `RETURN` \| `EXCHANGE` |
| claim_status_bo | string | `RETURN_RECEIVED` 또는 `EXCHANGE_RECEIVED` |
| claim_status_fo | string | `RETURN_RECEIVED` 또는 `EXCHANGE_RECEIVED` |
| received_by_id | number | 승인자 관리자 ID |
| received_at | datetime | 승인 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 승인 권한 없음 또는 타 고객사 건 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `RETURN_REQUESTED` / `EXCHANGE_REQUESTED` 외 상태 |
| 500 | INTERNAL_ERROR | 서버 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "{반품|교환}이 접수되었습니다. 수거 요청을 진행해 주세요."
2. 해당 행의 상태를 `RETURN_RECEIVED` / `EXCHANGE_RECEIVED`로 갱신
3. 상세 팝업에서 호출 시: 상세 재조회 + 하단 버튼 영역 리렌더링(수거요청 버튼 노출)
4. ClaimHistory에 `(REQUESTED → RECEIVED)` + 처리자 기록
5. FO 구매자에게 접수 완료 알림톡 발송

### 실패 시

1. 오류 토스트
2. 409 충돌 시 목록/상세 자동 새로고침

### 일괄 처리 시

- 각 건 개별 호출. 응답 집계 후 "N건 성공, M건 실패" 토스트.
- 실패 건 claim_code 목록을 펼침 영역에 표시.

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = RETURN_REQUESTED` 또는 `EXCHANGE_REQUESTED`
- 확인 모달 필수 통과

### 백엔드

- `claim_status_bo`가 `RETURN_REQUESTED`/`EXCHANGE_REQUESTED`인지 검증
- 관리자 권한 범위 내 `client_id` 검증
- ClaimHistory 자동 기록, `processor_id = 승인자`
- **교환의 경우:** `exchange_inventory_reserved` 상태 유지(재고 확정은 최종승인 시)
