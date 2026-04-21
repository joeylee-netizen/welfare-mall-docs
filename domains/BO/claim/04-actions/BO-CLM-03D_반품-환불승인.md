---
id: "BO-CLM-03D"
title: "반품 환불승인"
type: action
domain: claim
status: draft
version: "1.0"
created: 2026-04-22
updated: 2026-04-22
author: "기획자"
refs:
  - "BO-CLM-003"
  - "BO-CLM-010"
  - "BO-CLM-023"
  - "BO-CLM-024"
tags: [클레임, 반품, 검수, 환불승인, 환불요청, API]
trigger: submit
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/refund/approve"
---

# 반품 환불승인

## 트리거 조건

- [BO-CLM-023 반품 목록](../03-components/BO-CLM-023_반품-목록-테이블.md) 개별/일괄 `환불승인` 버튼 → 환불승인 확인 모달
- [BO-CLM-024 반품 상세](../03-components/BO-CLM-024_반품-상세-폼.md)의 `환불승인` 버튼
- 대상 상태: `RETURN_COLLECTED` (반품회수완료)
- 대상 유형: **반품 전용** (`claim_type = RETURN`)

검수 결과를 `PASSED`로 기록하고 환불 프로세스를 자동 트리거.

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| inspection_memo | string | N | 검수 메모(0~1000자, 선택) |

> `inspection_result`는 서버에서 자동 `PASSED`로 설정.

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_status_bo | string | `REFUND_REQUESTED` |
| claim_status_fo | string | `RETURN_COMPLETED` |
| inspection_result | string | `PASSED` |
| inspection_by_id | number | 검수자 관리자 ID |
| inspection_at | datetime | 검수 일시 |
| refund_record_ids | number[] | 생성된 RefundRecord ID 배열(수단별) |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 403 | FORBIDDEN | 권한 없음 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `RETURN_COLLECTED` 아님 |
| 409 | INVALID_CLAIM_TYPE | `claim_type` ≠ `RETURN` (교환에는 본 API 사용 불가) |
| 500 | INTERNAL_ERROR | 서버 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "검수가 통과 처리되었으며 환불 요청이 진행됩니다."
2. 해당 행 상태 `REFUND_REQUESTED`(BO) / `RETURN_COMPLETED`(FO)로 갱신
3. 상세 팝업 재조회 → 섹션 4(검수 정보) 반영 + 섹션 5(환불 상세)에 RefundRecord 추가
4. **환불 프로세스 자동 트리거:**
   - PG 환불 API 호출(복합결제인 경우 PG → 포인트 순)
   - RefundRecord 생성(`refund_status = PENDING`, `triggered_by = SYSTEM`)
   - 이후 성공 시 `REFUNDED`, 실패 시 `REFUND_FAILED` 전이
5. 쿠폰 원복 정책 재검토 및 처리
6. ClaimHistory에 `(RETURN_COLLECTED → REFUND_REQUESTED)` + 검수자·메모 기록
7. FO 구매자에게 반품 완료·환불 진행 알림톡

### 실패 시

1. 오류 토스트
2. 409 상태 충돌 시 상세 새로고침

### 일괄 처리 시

- 각 건 개별 호출, 응답 집계 토스트.

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = RETURN_COLLECTED` 및 `claim_type = RETURN`
- 확인 모달 필수

### 백엔드

- 상태·유형 검증
- 관리자 권한 범위 내 `client_id` 검증
- 트랜잭션 처리: Claim 상태 업데이트 + 검수 정보 기록 + RefundRecord 생성 원자적으로
- 쿠폰 원복·무료배송 조건 재검토 및 원주문 조정
