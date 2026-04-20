---
id: "BO-PDM-03F"
title: "할인프로모션 목록 조회"
type: action
domain: product
status: draft
version: "2.0"
created: 2026-04-20
updated: 2026-04-21
author: "기획자"
refs:
  - "BO-PDM-002"
  - "BO-PDM-026"
tags: [할인프로모션, 목록, 조회, 검색, API]
trigger: load
method: GET
endpoint: "/api/v1/promotions"
---

# 할인프로모션 목록 조회

## 트리거 조건

- 할인프로모션 목록([BO-PDM-026](../03-components/BO-PDM-026_할인프로모션-목록-테이블.md)) 페이지 최초 진입 시 자동 호출
- 검색 필터 변경 후 `검색` 버튼 클릭 또는 Enter 키 입력 시
- 페이지네이션 이동, 출력개수 변경, 정렬 변경 시

---

## 요청 (Request)

### Parameters (Query String)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| promotion_type | string | N | 프로모션구분 필터 (`MZ_VENDOR` \| `MZ_PLATFORM` \| `VENDOR`), 복수 가능 |
| vendor_id | number | N | 판매사 ID (팝업 선택) |
| status | string | N | 진행상태 필터 (`DRAFT` \| `ACTIVE` \| `PAUSED` \| `ENDED`), 복수 가능 |
| coupon_applicable | boolean | N | 쿠폰적용여부 필터 (`true` \| `false`) |
| date_type | string | N | 기간유형 (`created_at` \| `start_date` \| `end_date`, 기본: `created_at`) |
| date_from | date | N | 기간 시작일 (`YYYY-MM-DD`) |
| date_to | date | N | 기간 종료일 (`YYYY-MM-DD`) |
| keyword_type | string | N | 검색 유형 (`promotion_name` \| `promotion_code` \| `product_code`) |
| keyword | string | N | 검색어 |
| sort_by | string | N | 정렬 컬럼 (기본: `created_at`) |
| sort_order | string | N | 정렬 방향 (`asc` \| `desc`, 기본: `desc`) |
| page | number | N | 페이지 번호 (기본: 1) |
| page_size | number | N | 페이지당 출력 건수 (기본: 20, 허용: 20/50/100) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| total_count | number | 전체 검색결과 건수 |
| page | number | 현재 페이지 번호 |
| page_size | number | 페이지당 출력 건수 |
| items | array | 프로모션 목록 배열 |
| items[].promotion_id | number | 프로모션 ID |
| items[].promotion_code | string | 프로모션코드 |
| items[].promotion_name | string | 프로모션명 |
| items[].promotion_type | string | 프로모션구분 (`MZ_VENDOR` \| `MZ_PLATFORM` \| `VENDOR`) |
| items[].vendor_code | string | 판매사코드 |
| items[].vendor_name | string | 판매사명 |
| items[].coupon_applicable | boolean | 쿠폰적용여부 |
| items[].start_date | datetime | 프로모션 시작일시 |
| items[].end_date | datetime | 프로모션 종료일시 |
| items[].status | string | 진행상태 (`DRAFT` \| `ACTIVE` \| `PAUSED` \| `ENDED`) |
| items[].created_by | string | 등록자 |
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

- `page_size`는 허용값(20/50/100) 외 입력 불가
- `date_from` <= `date_to` 검증
- `keyword_type` 선택 시 `keyword` 필수

### 백엔드

- 날짜 형식 검증 (`YYYY-MM-DD`)
- `page_size` 허용값 검증
- `promotion_type`, `status` 유효 Enum 값 검증
- DELETED 상태 프로모션은 조회 결과에서 제외
