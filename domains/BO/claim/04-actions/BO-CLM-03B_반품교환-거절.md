---
id: "BO-CLM-03B"
title: "반품/교환 거절"
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
tags: [클레임, 반품, 교환, 거절, 철회, 반려사유, API, 재고복원]
trigger: submit
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/withdraw"
---

# 반품/교환 거절

## 트리거 조건

- 반품/교환 목록의 개별/일괄 `거절` 버튼 → 반려사유 입력 모달 → `거절 확정`
- 반품/교환 상세 모달의 `거절` 버튼
- 대상 상태: `RETURN_REQUESTED` 또는 `EXCHANGE_REQUESTED`

반품·교환 공용. **교환 거절 시 재고 복원 부수 효과 발생.**

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| reject_reason | string | Y | 반려 사유 (10~200자) |
| fo_notice_template | string | N | FO 안내 문구 템플릿 (반품 불가 사유 중 선택 또는 `OTHER`) |
| fo_notice_message | string | N | FO 추가 안내 메시지(템플릿 뒤 덧붙임, 0~500자) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_type | string | `RETURN` \| `EXCHANGE` |
| claim_status_bo | string | `RETURN_WITHDRAWN` 또는 `EXCHANGE_WITHDRAWN` |
| claim_status_fo | string | `RETURN_WITHDRAWN` 또는 `EXCHANGE_WITHDRAWN` |
| rejected_by_id | number | 반려자 관리자 ID |
| rejected_at | datetime | 반려 일시 |
| reject_reason | string | 저장된 반려 사유 |
| inventory_restored | boolean | **교환 전용.** 재고 복원 실행 여부 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | REJECT_REASON_REQUIRED | 반려 사유 누락 |
| 400 | REJECT_REASON_LENGTH | 길이(10~200자) 위반 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `RETURN_REQUESTED` / `EXCHANGE_REQUESTED` 외 상태 |
| 500 | INVENTORY_RESTORE_FAILED | 교환 재고 복원 실패 (트랜잭션 롤백) |
| 500 | INTERNAL_ERROR | 서버 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "{반품|교환}이 거절되었습니다. (사유: {reject_reason 앞 20자}...)"
2. 해당 행의 상태를 `RETURN_WITHDRAWN` / `EXCHANGE_WITHDRAWN`으로 갱신
3. 상세 팝업에서 호출 시: 재조회 + 종결 상태로 버튼 영역 비활성
4. ClaimHistory에 `(REQUESTED → WITHDRAWN)` + 반려자/사유 기록
5. **원주문 상태 복원:** 클레임 신청 전 주문상태(`DELIVERED` 또는 `EXCHANGE_DELIVERED`)로 복원
6. **교환 전용:** 선차감된 재고 자동 복원(`exchange_inventory_reserved = false` 기록, 실제 재고 +`quantity`)
7. FO 구매자에게 거절 안내 알림톡·메일 발송:
   - 기본 본문: "{반품|교환} 요청이 거절되었습니다. (사유: ...)"
   - `fo_notice_template`/`fo_notice_message` 포함

### 실패 시

1. 오류 토스트
2. **500 INVENTORY_RESTORE_FAILED:** 전체 트랜잭션 롤백. 관리자에게 CS 연락 유도 메시지
3. 409 상태 충돌 시 자동 새로고침

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = RETURN_REQUESTED` 또는 `EXCHANGE_REQUESTED`
- 반려사유 길이 10~200자, 공백만 입력 시 무효
- 템플릿 선택 시 기본 안내 문구를 본문에 자동 프리필

### 백엔드

- 상태 검증
- `reject_reason` 필수/길이 검증
- 관리자 권한 범위 내 `client_id` 검증
- 원주문 상태 원복 트랜잭션 처리
- **교환 전용:** `exchange_inventory_reserved = true`인 경우 재고 복원을 동일 트랜잭션 내에서 처리. 실패 시 전체 롤백.
- 알림 발송(비동기)
