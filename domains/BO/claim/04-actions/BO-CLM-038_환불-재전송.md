---
id: "BO-CLM-038"
title: "환불 재전송"
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
tags: [클레임, 주문취소, 환불, 재전송, 실패, API]
trigger: click
method: POST
endpoint: "/api/v1/claims/{claim_id}/refund/retry"
---

# 환불 재전송

## 트리거 조건

- [BO-CLM-021 주문취소 목록](../03-components/BO-CLM-021_주문취소-목록-테이블.md) 개별 행 `환불재전송` 버튼 클릭 → 확인 모달 → `재전송`
- 목록의 `일괄 환불재전송` 버튼 클릭 → 확인 모달 → `재전송` (선택 행에 대해 반복 호출)
- [BO-CLM-022 주문취소 상세 폼](../03-components/BO-CLM-022_주문취소-상세-폼.md)의 `환불 재전송` 버튼 클릭 → 확인 모달
- 대상: `claim_status_bo = REFUND_FAILED`

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| memo | string | N | 재전송 사유 메모(0~500자) — CS 판단 등 |

---

## 응답 (Response)

### 성공 (202 Accepted)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_status_bo | string | `REFUND_REQUESTED` (재전송 요청 접수됨) |
| refund_record_ids | number[] | 새로 생성된 RefundRecord ID 배열(수단별) |
| retry_count | number | 누적 재시도 횟수(배치 + 수동) |
| triggered_by | string | `ADMIN_MANUAL` |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 권한 없음 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `REFUND_FAILED` 상태 아님 |
| 409 | DAILY_RETRY_LIMIT_EXCEEDED | 당일 수동 재시도 1회 제한 초과 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "환불 재전송 요청되었습니다. 처리 결과는 환불 완료 또는 실패로 반영됩니다."
2. 상세/목록 자동 새로고침
3. 환불 수단 상세 테이블에 새 RefundRecord(`PENDING`) 추가
4. ClaimHistory에 `(REFUND_FAILED → REFUND_REQUESTED)` + 관리자 ID + 메모 기록
5. PG사 비동기 응답 수신 시:
   - 성공 → `REFUNDED`로 상태 전환 및 RefundRecord 업데이트
   - 실패 → `REFUND_FAILED` 재진입(재전송 카운터 +1)

### 실패 시

1. 오류 토스트
2. `DAILY_RETRY_LIMIT_EXCEEDED` 시 버튼 비활성 + 툴팁("내일 0시 이후 재시도 가능")

### 일괄 처리 시

- 각 건에 대해 개별 호출. `성공 N건 / 실패 M건` 토스트 요약.
- 일일 한도 초과 건은 실패 집계에 포함하되 별도 메시지 표시.

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = REFUND_FAILED`
- 확인 모달: "환불 요청을 재전송합니다. 당일 수동 재시도는 1회로 제한됩니다."

### 백엔드

- `claim_status_bo = REFUND_FAILED` 검증
- **일일 수동 재시도 한도(1회) 체크** — 동일 claim에 `triggered_by = ADMIN_MANUAL`인 RefundRecord가 당일 존재하면 409 반환
- 시스템 배치(BATCH) 재전송은 별도로 1일 1회 실행(관리자 수동 한도와 분리)
- 재전송 시 기존 `FAILED` RefundRecord는 그대로 보존, 신규 `PENDING` 레코드 생성
- PG API 호출은 비동기 큐잉(실시간 응답 대기하지 않음)
