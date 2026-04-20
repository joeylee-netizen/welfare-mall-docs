---
id: "BO-PDM-03C"
title: "할인프로모션 등록"
type: action
domain: product
status: draft
version: "1.0"
created: 2026-04-20
updated: 2026-04-20
author: "기획자"
refs:
  - "BO-PDM-002"
  - "BO-PDM-011"
  - "BO-PDM-027"
tags: [할인프로모션, 등록, 저장, API]
trigger: submit
method: POST
endpoint: "/api/v1/promotions"
---

# 할인프로모션 등록

## 트리거 조건

- 할인프로모션 등록 폼([BO-PDM-027](../03-components/BO-PDM-027_할인프로모션-등록-폼.md))에서 **등록 모드** 상태
- 모든 필수 필드 입력 후 `저장` 버튼 클릭
- 확인 모달에서 `확인` 클릭 시 API 호출 실행

---

## 사전 호출

없음 (등록 모드는 빈 폼으로 진입)

---

## 요청 (Request)

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| promotion_name | string | Y | 프로모션명 (최대 100자) |
| promotion_type | string | Y | 프로모션구분 (`MZ_VENDOR` \| `MZ_PLATFORM` \| `VENDOR`) |
| discount_amount | number | Y | 할인금액 (원 단위, 0 초과) |
| start_date | datetime | Y | 시작일시 (`YYYY-MM-DD HH:mm`) |
| end_date | datetime | Y | 종료일시 (`YYYY-MM-DD HH:mm`) |
| coupon_applicable | boolean | Y | 쿠폰적용여부 |
| product_codes | string[] | Y | 대상 상품코드 배열 (1건 이상) |
| memo | string | N | 비고 |

---

## 응답 (Response)

### 성공 (201 Created)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| promotion_id | number | 생성된 프로모션 ID |
| promotion_code | string | 자동 발급된 프로모션코드 |
| status | string | `ACTIVE` |
| created_at | datetime | 등록일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 필수 파라미터 누락 또는 유효성 검증 실패 |
| 400 | PROMOTION_NAME_TOO_LONG | 프로모션명 100자 초과 |
| 400 | INVALID_DISCOUNT_AMOUNT | 할인금액 0 이하 |
| 400 | INVALID_DATE_RANGE | 종료일시가 시작일시 이전 |
| 400 | NO_TARGET_PRODUCTS | 대상 상품 0건 |
| 409 | PRODUCT_NOT_APPROVED | 대상 상품 중 승인완료 상태가 아닌 건 존재 |
| 409 | DATE_OVERLAP | 동일 상품에 기간이 중복되는 프로모션 존재 |
| 401 | UNAUTHORIZED | 인증 실패 |
| 403 | FORBIDDEN | 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트 표시: "할인프로모션이 등록되었습니다."
2. 할인프로모션 목록 화면([BO-PDM-048](../05-wireframes/BO-PDM-048_할인프로모션-목록-화면.md))으로 이동

### 실패 시

1. 오류 토스트 메시지 표시
2. 폼 데이터 유지 (입력값 손실 방지)
3. 모달 닫힘

---

## 유효성 검사

### 프론트엔드 (모달 표시 전)

- `promotion_name`: 빈 값 불가, 100자 이내
- `promotion_type`: 미선택 불가
- `discount_amount`: 0 초과 정수
- `start_date`, `end_date`: 모두 필수, 종료일시 > 시작일시
- `product_codes`: 1건 이상 필수

### 백엔드

- 프론트엔드 검증 항목 동일 적용
- 대상 상품 승인완료(`APPROVED`) 여부 검증
- 동일 상품에 대한 기간 중복 검증
- `promotion_type` 유효값 검증
