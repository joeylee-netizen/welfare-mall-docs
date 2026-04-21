---
id: "BO-CLM-037"
title: "환불 수기 완료"
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
tags: [클레임, 주문취소, 환불, 수기완료, API]
trigger: submit
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/refund/complete"
---

# 환불 수기 완료

## 트리거 조건

- [BO-CLM-022 주문취소 상세 폼](../03-components/BO-CLM-022_주문취소-상세-폼.md)의 `환불 수기 완료` 버튼 클릭 → 확인 모달 → `완료 처리`
- 대상: `claim_status_bo = REFUND_REQUESTED` 또는 `REFUND_FAILED`
- 사용 목적: 외부(PG사 관리 콘솔 등)에서 환불을 직접 처리한 경우 상태를 `환불완료`로 확정하거나, 반복 실패 건을 관리자가 최종 마감 처리할 때 사용

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| memo | string | N | 수기 완료 처리 메모(0~500자) — 외부 처리 근거, PG 거래 ID 등 |
| external_reference | string | N | 외부 환불 처리 참조값(PG사 관리자 화면 거래 ID 등) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_status_fo | string | `CLAIM_COMPLETED` |
| claim_status_bo | string | `REFUNDED` |
| completed_at | datetime | 환불 완료 일시 |
| refund_record_id | number | 수기 완료로 기록된 RefundRecord ID (`refund_status = MANUAL_COMPLETED`) |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 권한 없음 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `REFUND_REQUESTED` / `REFUND_FAILED` 외 상태 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "환불 완료 처리되었습니다."
2. 상세 재조회 + 하단 버튼 영역 리렌더링(종결 상태로 조회만 가능)
3. 목록 자동 새로고침
4. ClaimHistory에 `(REFUND_REQUESTED/REFUND_FAILED → REFUNDED)` + 메모 기록
5. 수기 완료 트리거의 RefundRecord 생성(`triggered_by = ADMIN_MANUAL`, `refund_status = MANUAL_COMPLETED`)
6. 기존 진행 중 RefundRecord(PENDING) 존재 시 `FAILED`로 마감 후 수기 완료 레코드 추가
7. 쿠폰 원복 정책 재검토 및 원복 처리(필요 시)

### 실패 시

1. 오류 토스트
2. 상태 충돌(409) 시 상세 자동 새로고침

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo ∈ {REFUND_REQUESTED, REFUND_FAILED}`
- 확인 모달 필수 통과
- `memo` 500자 초과 불가

### 백엔드

- 대상 상태 검증
- 관리자 권한 검증(수기 완료는 권한 레벨 상위 — 예: 클레임 매니저 이상)
- 트랜잭션 처리: Claim 상태 업데이트 + RefundRecord 기록 + ClaimHistory 기록을 원자적으로 수행
- 수기 완료 감사 로그 별도 기록(감사 추적용)
