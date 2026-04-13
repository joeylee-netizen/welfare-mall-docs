---
id: "BO-PDM-038"
title: "상품 목록 조회"
type: action
domain: product
status: draft
version: "1.0"
created: 2026-04-13
updated: 2026-04-13
author: "기획자"
refs:
  - "BO-PDM-001"
  - "BO-PDM-024"
  - "BO-PDM-025"
tags: [상품, 목록, 조회, 검색, API]
trigger: load
method: GET
endpoint: "/api/v1/products"
---

# 상품 목록 조회

## 트리거 조건

- 배송상품 목록([BO-PDM-024](../03-components/BO-PDM-024_배송상품-목록-테이블.md)) 또는 티켓상품 목록([BO-PDM-025](../03-components/BO-PDM-025_티켓상품-목록-테이블.md)) 페이지 최초 진입 시 자동 호출
- 검색 필터 변경 후 `검색` 버튼 클릭 또는 Enter 키 입력 시
- 페이지네이션 이동, 출력개수 변경, 정렬 변경 시

---

## 요청 (Request)

### Parameters (Query String)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_category | string | Y | 상품 카테고리 (`SHIPPING` \| `TICKET`) |
| approval_status | string | Y | 고정값 `APPROVED` (승인완료 상품만 조회) |
| product_type | string | N | 상품유형 필터 (예: `GENERAL_SHIPPING`, `INSTALLATION`, `TICKET_COUPON`, `TICKET_RESERVATION`) |
| sales_status | string | N | 판매상태 필터 (예: `SELLING`, `SALES_PAUSED`, `TEMP_OUT_OF_STOCK`, `OUT_OF_STOCK`, `SALES_ENDED`) |
| brand_id | number | N | 브랜드 ID (배송상품 목록 전용) |
| vendor_id | number | N | 판매사 ID (티켓상품 목록 전용) |
| product_name | string | N | 상품명 (부분 일치 검색) |
| created_at_from | date | N | 등록기간 시작일 (`YYYY-MM-DD`) |
| created_at_to | date | N | 등록기간 종료일 (`YYYY-MM-DD`) |
| sort_by | string | N | 정렬 컬럼 (기본: `created_at`) |
| sort_order | string | N | 정렬 방향 (`asc` \| `desc`, 기본: `desc`) |
| page | number | N | 페이지 번호 (기본: 1) |
| page_size | number | N | 페이지당 출력 건수 (기본: 10, 허용: 10/20/50/100) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| total_count | number | 전체 검색결과 건수 |
| page | number | 현재 페이지 번호 |
| page_size | number | 페이지당 출력 건수 |
| items | array | 상품 목록 배열 |
| items[].product_id | number | 상품 ID |
| items[].product_code | string | 상품코드 |
| items[].request_code | string | 요청코드 |
| items[].product_name | string | 상품명 |
| items[].product_type | string | 상품유형 |
| items[].product_category | string | 상품 카테고리 |
| items[].sales_status | string | 판매상태 |
| items[].brand_id | number | 브랜드 ID (배송형) |
| items[].brand_name | string | 브랜드명 (배송형) |
| items[].vendor_id | number | 판매사 ID (티켓형) |
| items[].vendor_name | string | 판매사명 (티켓형) |
| items[].sales_start_date | date | 판매시작일 |
| items[].sales_end_date | date | 판매종료일 |
| items[].created_at | datetime | 등록일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 필수 파라미터 누락 또는 잘못된 값 |
| 401 | UNAUTHORIZED | 인증 실패 |
| 403 | FORBIDDEN | 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 테이블에 `items` 데이터 렌더링
2. 상단 툴바에 `total_count` 표시 ("검색결과 N건")
3. 페이지네이션 UI 갱신 (`total_count`, `page`, `page_size` 기반)
4. 스크롤 위치를 테이블 상단으로 이동 (페이지 변경 시)

### 실패 시

1. 오류 토스트 메시지 표시
2. 이전 데이터 유지 (있는 경우)

### 데이터 없음 (total_count = 0)

1. 테이블 영역에 "검색 결과가 없습니다." 메시지 표시
2. 페이지네이션 숨김

---

## 유효성 검사

### 프론트엔드

- `product_category`는 컴포넌트 진입 시 자동 설정 (배송: `SHIPPING`, 티켓: `TICKET`)
- `approval_status`는 `APPROVED` 고정
- `page_size`는 허용값(10/20/50/100) 외 입력 불가
- `created_at_from` ≤ `created_at_to` 검증

### 백엔드

- `product_category` 필수 검증
- `approval_status = APPROVED` 고정 검증
- 날짜 형식 검증 (`YYYY-MM-DD`)
- `page_size` 허용값 검증
