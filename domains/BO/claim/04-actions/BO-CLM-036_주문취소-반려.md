---
id: "BO-CLM-036"
title: "주문취소 반려"
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
tags: [클레임, 주문취소, 반려, 반려사유, 반품교환전환, API]
trigger: submit
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/reject"
---

# 주문취소 반려

## 트리거 조건

- [BO-CLM-021 주문취소 목록](../03-components/BO-CLM-021_주문취소-목록-테이블.md)에서 개별 행 `반려` 버튼 → 반려사유 입력 모달 → `반려 확정`
- 목록의 `일괄 반려` 버튼 → 반려사유 입력 모달 → `반려 확정` (각 선택 행에 대해 반복 호출)
- [BO-CLM-022 주문취소 상세 폼](../03-components/BO-CLM-022_주문취소-상세-폼.md)의 `반려` 버튼 클릭 → 반려사유 입력 모달
- 대상: `claim_status_bo = CANCEL_REQUESTED` + `approval_required = true` (상품준비중 단계)

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
| fo_notice_template | string | N | FO 안내 문구 템플릿 (`OUTBOUND_READY` \| `DELIVERY_IMMINENT` \| `OTHER`) |
| fo_notice_message | string | N | FO에게 노출할 추가 안내 메시지(템플릿에 덧붙임, 0~500자) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_status_fo | string | `CLAIM_CANCELED` |
| claim_status_bo | string | `CANCEL_WITHDRAWN` |
| rejected_by_id | number | 반려자 관리자 ID |
| rejected_at | datetime | 반려 일시 |
| reject_reason | string | 저장된 반려 사유 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | REJECT_REASON_REQUIRED | 반려 사유 누락 |
| 400 | REJECT_REASON_LENGTH | 반려 사유 길이(10~200자) 위반 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `CANCEL_REQUESTED` + `approval_required = true` 아님 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "주문취소가 반려되었습니다. (사유: {reject_reason 앞 20자}...)"
2. 해당 행의 상태를 `CANCEL_WITHDRAWN`(BO) / `CLAIM_CANCELED`(FO)로 갱신
3. 상세 팝업에서 호출 시: 상세 재조회 + 하단 버튼 영역 리렌더링(종결 상태로 조회만 가능)
4. ClaimHistory에 `(CANCEL_REQUESTED → CANCEL_WITHDRAWN)` + 반려자/사유 기록
5. **원주문 상태 복원:** 주문취소 신청 전 상태(`PAID` 또는 `PREPARING`)로 원주문 상태 원복
6. FO 구매자에게 반려 안내 알림톡·메일 발송:
   - 기본 본문: "주문취소 요청이 반려되었습니다. (사유: ...)"
   - 템플릿별 추가 안내:
     - `OUTBOUND_READY`: "출고 준비가 완료되어 취소가 불가합니다. 상품 수령 후 반품으로 진행해 주세요."
     - `DELIVERY_IMMINENT`: "배송이 곧 시작됩니다. 배송 중 수령 거부로 반품이 가능합니다."
     - `OTHER`: 사용자 입력 메시지 사용

### 실패 시

1. 오류 토스트 유지, 모달은 유지되어 재시도 가능
2. 상태 충돌(409) 시 목록·상세 자동 새로고침

### 일괄 처리 시

- 각 건에 대해 개별 호출. `성공 N건 / 실패 M건` 토스트 요약 표시.
- 반려사유는 공통 적용(일괄 반려 모달에서 1회 입력).

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = CANCEL_REQUESTED` + `approval_required = true`
- 반려사유 길이 10~200자, 공백만 입력 시 무효
- 템플릿 선택 시 기본 안내 문구를 본문에 자동 프리필(읽기 전용)

### 백엔드

- `claim_status_bo = CANCEL_REQUESTED` + `approval_required = true` 검증
- `reject_reason` 필수/길이 검증
- 관리자 권한 범위 내 `client_id` 검증
- 원주문 상태 원복 트랜잭션 처리(실패 시 전체 롤백)
- 알림 발송(비동기)
