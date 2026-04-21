---
id: "BO-CLM-030"
title: "클레임 목록 조회"
type: action
domain: claim
status: draft
version: "1.0"
created: 2026-04-21
updated: 2026-04-21
author: "기획자"
refs:
  - "BO-CLM-001"
  - "BO-CLM-010"
  - "BO-CLM-020"
tags: [클레임, 통합, 목록, 조회, 검색, API]
trigger: load
method: GET
endpoint: "/api/v1/claims"
---

# 클레임 목록 조회

## 트리거 조건

- [BO-CLM-020 배송상품 클레임 목록 테이블](../03-components/BO-CLM-020_배송상품-클레임-목록-테이블.md) 컴포넌트 최초 진입 시 자동 호출
- 검색 필터 변경 후 `검색` 버튼 클릭 또는 Enter 키 입력 시
- 탭 전환(주문취소/반품/교환) 시 (활성 탭에 한함)
- 페이지네이션 이동, 출력 건수 변경, 정렬 변경 시

---

## 요청 (Request)

### Parameters (Query String)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_type | string | Y | 클레임 유형 (`CANCEL` \| `RETURN` \| `EXCHANGE`) — 탭 기준 자동 지정 |
| claim_sub_type | string[] | N | 하위 유형 필터 (`ORDER_CANCEL_BY_USER`, `ORDER_CANCEL_BY_ADMIN` 등) |
| claim_status_fo | string[] | N | FO 상태 필터 |
| claim_status_bo | string[] | N | BO 상태 필터 |
| process_channel | string[] | N | 처리 채널 (`FO`, `BO_PROXY`, `SYSTEM`) |
| applied_channel | string[] | N | 접수 채널 (`FO`, `BO_PROXY`, `ADMIN`) |
| liability_party | string[] | N | 귀책 (`BUYER`, `SELLER`) |
| reason_code | string | N | 클레임 사유 |
| vendor_id | number | N | 판매사 ID |
| client_id | number | N | 고객사 ID (멀티태넌트) |
| buyer_id | number | N | 구매자 ID |
| refund_method | string[] | N | 환불 수단 |
| approval_required | boolean | N | 관리자 수기 승인 필요 여부 |
| date_type | string | N | 기간 유형 (`applied_at` \| `status_changed_at` \| `completed_at`). 기본 `applied_at` |
| date_from | date | N | 기간 시작(`YYYY-MM-DD`) |
| date_to | date | N | 기간 종료(`YYYY-MM-DD`) |
| keyword_type | string | N | 검색어 유형 (`claim_code` \| `order_id` \| `product_name` \| `buyer_name`) |
| keyword | string | N | 검색어(콤마 복수 허용: claim_code/order_id/product_name 한정) |
| sort_by | string | N | 정렬 컬럼 (기본 `applied_at`) |
| sort_order | string | N | `asc` \| `desc` (기본 `desc`) |
| page | number | N | 페이지 번호 (기본 1) |
| page_size | number | N | 페이지당 건수 (기본 20, 허용 20/50/100) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| total_count | number | 전체 검색결과 건수 |
| page | number | 현재 페이지 |
| page_size | number | 페이지당 건수 |
| items | array | 클레임 목록 |
| items[].claim_id | number | 클레임 ID |
| items[].claim_code | string | 클레임 코드 |
| items[].claim_type | string | 클레임 유형 |
| items[].claim_sub_type | string | 하위 유형 |
| items[].claim_status_fo | string | FO 상태 |
| items[].claim_status_bo | string | BO 상태 |
| items[].process_channel | string | 처리 채널 |
| items[].applied_channel | string | 접수 채널 |
| items[].order_id | number | 원주문 ID |
| items[].shipment_name | string | 출고지명 |
| items[].client_name | string | 고객사명 |
| items[].buyer_id | number | 구매자 ID |
| items[].buyer_name | string | 구매자명 |
| items[].vendor_id | number | 판매사 ID |
| items[].vendor_name | string | 판매사명 |
| items[].representative_product_name | string | 대표 상품명(+N건) |
| items[].total_item_quantity | number | 대상 수량 합계 |
| items[].liability_party | string | 귀책 |
| items[].reason_code | string | 사유 코드 |
| items[].refund_total_amount | number | 환불 총액 |
| items[].refund_pg_amount | number | PG 환불액 |
| items[].refund_point_amount | number | 포인트 복원액 |
| items[].shipping_fee_charged | number | 클레임 배송비 |
| items[].approval_required | boolean | 수기 승인 필요 여부 |
| items[].applied_at | datetime | 신청일시 |
| items[].status_changed_at | datetime | 상태변경일 |
| items[].completed_at | datetime | 환불완료일 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 필수 파라미터 누락 또는 잘못된 값 |
| 401 | UNAUTHORIZED | 인증 실패 |
| 403 | FORBIDDEN | 클레임 관리 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 테이블에 `items` 데이터 렌더링(badge, 금액 포매팅 포함)
2. 툴바에 `total_count` 표시 ("조회결과 N건")
3. 페이지네이션 UI 갱신
4. 긴급 행(`approval_required = true` + `claim_status_bo = CANCEL_REQUESTED`)은 강조 배경

### 실패 시

1. 오류 토스트 메시지 표시
2. 이전 데이터 유지

### 데이터 없음 (`total_count = 0`)

1. 테이블 영역에 "조회 결과가 없습니다. 필터를 조정해 주세요." 메시지 표시
2. 페이지네이션 숨김

---

## 유효성 검사

### 프론트엔드

- `claim_type`은 탭 선택에 따라 자동 지정(비활성 탭 API 호출 금지)
- 검색어 select가 `사용안함`이 아닌 경우 다른 필터 파라미터 미전송
- `date_from ≤ date_to` 검증
- `page_size` 허용값 검증

### 백엔드

- 관리자 권한 범위 내 `client_id`만 조회 가능 (권한 외 요청 시 403)
- `claim_type` 필수
- 날짜 형식 검증
- 목록은 `client_id` 기준 자동 필터(멀티태넌트 격리)
