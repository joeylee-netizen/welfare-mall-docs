---
id: "BO-CLM-031"
title: "주문취소 목록 조회"
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
tags: [클레임, 주문취소, 목록, 조회, 검색, API]
trigger: load
method: GET
endpoint: "/api/v1/claims/cancel"
---

# 주문취소 목록 조회

## 트리거 조건

- [BO-CLM-021 주문취소 목록 테이블](../03-components/BO-CLM-021_주문취소-목록-테이블.md) 최초 진입 시 자동 호출
- 검색 필터 변경 후 `검색` 또는 Enter 입력 시
- 페이지네이션/정렬/출력건수 변경 시
- 상세 팝업에서 처리(승인/반려/환불완료/재전송) 후 자동 새로고침 시

---

## 요청 (Request)

### Parameters (Query String)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_sub_type | string[] | N | `ORDER_CANCEL_BY_USER`, `ORDER_CANCEL_BY_ADMIN` |
| claim_status_fo | string[] | N | FO 상태 필터 |
| claim_status_bo | string[] | N | BO 상태 필터 |
| process_channel | string[] | N | 처리 채널 |
| applied_channel | string[] | N | 접수 채널 |
| approval_required | boolean | N | 수기 승인 필요 여부 |
| liability_party | string[] | N | 귀책 |
| reason_code | string | N | 사유 |
| vendor_id | number | N | 판매사 ID |
| client_id | number | N | 고객사 ID |
| refund_method | string[] | N | 환불 수단 |
| date_type | string | N | 기간 유형 (기본 `applied_at`) |
| date_from | date | N | 기간 시작 |
| date_to | date | N | 기간 종료 |
| keyword_type | string | N | `claim_code` \| `order_id` \| `product_name` \| `buyer_name` |
| keyword | string | N | 검색어 |
| sort_by | string | N | 기본 `applied_at` |
| sort_order | string | N | `desc`(기본) |
| page | number | N | 기본 1 |
| page_size | number | N | 기본 20 (20/50/100) |

> `claim_type`은 서버에서 `CANCEL` 고정 처리(쿼리 전송 불필요).

---

## 응답 (Response)

### 성공 (200 OK)

[BO-CLM-030](./BO-CLM-030_클레임-목록-조회.md)의 응답 구조와 동일. 단, `items[].claim_type`은 항상 `CANCEL`.

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 파라미터 오류 |
| 401 | UNAUTHORIZED | 인증 실패 |
| 403 | FORBIDDEN | 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 테이블 데이터 렌더링
2. **긴급 행 강조:** `approval_required = true` + `claim_status_bo = CANCEL_REQUESTED` 행 `#fff7ed` 배경
3. 툴바 건수 표시
4. 일괄 처리 버튼 활성화 상태 재계산(체크박스 선택 초기화)

### 실패 시

- 에러 토스트 + 이전 데이터 유지

### 데이터 없음

- "조회 결과가 없습니다." 표시

---

## 유효성 검사

### 프론트엔드

- 검색어 select 외 필터 비활성 시 파라미터 미전송
- 날짜 범위 검증
- `page_size` 허용값 검증

### 백엔드

- 관리자 권한 범위 내 `client_id`만 조회
- 목록 정렬 허용 컬럼 화이트리스트(`applied_at`, `status_changed_at`, `completed_at`, `refund_total_amount`)
