---
id: "BO-PDM-039"
title: "상품 승인목록 조회"
type: action
domain: product
status: draft
version: "1.0"
created: 2026-04-13
updated: 2026-04-13
author: "기획자"
refs:
  - "BO-PDM-001"
  - "BO-PDM-022"
  - "BO-PDM-023"
tags: [상품, 승인, 목록, 조회, 검색, API]
trigger: load
method: GET
endpoint: "/api/v1/products/approval"
---

# 상품 승인목록 조회

## 트리거 조건

- 배송상품 승인 목록([BO-PDM-022](../03-components/BO-PDM-022_배송상품-승인-목록-테이블.md)) 또는 티켓상품 승인 목록([BO-PDM-023](../03-components/BO-PDM-023_티켓상품-승인-목록-테이블.md)) 페이지 최초 진입 시 자동 호출
- 검색 필터 변경 후 `검색` 버튼 클릭 또는 Enter 키 입력 시
- 페이지네이션 이동 시
- 승인/반려 처리 완료 후 목록 재조회 시

---

## 요청 (Request)

### Parameters (Query String)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_category | string | Y | 상품 카테고리 (`SHIPPING` \| `TICKET`) |
| approval_status | string | N | 승인상태 필터 (기본: `PENDING`). `REGISTERING`, `PENDING`, `WITHDRAWN`, `APPROVED`, `REJECTED` |
| product_type | string | N | 상품유형 필터 |
| product_name | string | N | 상품명 (부분 일치 검색) |
| created_at_from | date | N | 요청기간 시작일 (`YYYY-MM-DD`) |
| created_at_to | date | N | 요청기간 종료일 (`YYYY-MM-DD`) |
| sort_by | string | N | 정렬 컬럼 (기본: `created_at`) |
| sort_order | string | N | 정렬 방향 (`asc` \| `desc`, 기본: `desc`) |
| page | number | N | 페이지 번호 (기본: 1) |
| page_size | number | N | 페이지당 출력 건수 (기본: 10) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| total_count | number | 전체 검색결과 건수 |
| page | number | 현재 페이지 번호 |
| page_size | number | 페이지당 출력 건수 |
| items | array | 승인 목록 배열 |
| items[].product_id | number | 상품 ID |
| items[].request_code | string | 요청코드 |
| items[].product_name | string | 상품명 |
| items[].product_type | string | 상품유형 |
| items[].product_category | string | 상품 카테고리 |
| items[].approval_status | string | 승인상태 |
| items[].brand_id | number | 브랜드 ID (배송형) |
| items[].brand_name | string | 브랜드명 (배송형) |
| items[].vendor_id | number | 판매사 ID (티켓형) |
| items[].vendor_name | string | 판매사명 (티켓형) |
| items[].created_at | datetime | 승인요청일시 |
| items[].changed_by | string | 요청자 |

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
3. 페이지네이션 UI 갱신
4. 체크박스 선택 상태 초기화
5. `PENDING` 상태 행에만 체크박스 활성화

### 실패 시

1. 오류 토스트 메시지 표시
2. 이전 데이터 유지 (있는 경우)

### 데이터 없음 (total_count = 0)

1. 배송: "승인 대기 중인 배송상품이 없습니다." 메시지 표시
2. 티켓: "승인 대기 중인 티켓상품이 없습니다." 메시지 표시
3. 페이지네이션 숨김

---

## 유효성 검사

### 프론트엔드

- `product_category`는 컴포넌트 진입 시 자동 설정 (배송: `SHIPPING`, 티켓: `TICKET`)
- `approval_status` 기본값 `PENDING` 자동 설정
- `created_at_from` ≤ `created_at_to` 검증

### 백엔드

- `product_category` 필수 검증
- `approval_status` Enum 값 검증
- 날짜 형식 검증 (`YYYY-MM-DD`)
