---
id: "BO-PDM-03D"
title: "할인프로모션 수정"
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
tags: [할인프로모션, 수정, API]
trigger: submit
method: PUT
endpoint: "/api/v1/promotions/{promotion_code}"
---

# 할인프로모션 수정

## 트리거 조건

- 할인프로모션 등록 폼([BO-PDM-027](../03-components/BO-PDM-027_할인프로모션-등록-폼.md))에서 **수정 모드** 상태
- 수정 필드 변경 후 `저장` 버튼 클릭
- 확인 모달에서 `확인` 클릭 시 API 호출 실행

---

## 사전 호출

| API | 용도 |
|-----|------|
| `GET /api/v1/promotions/{promotion_code}` | 기존 프로모션 상세 데이터 로드 |

---

## 수정 범위 (상태별)

| 상태 | 수정 가능 필드 | 비고 |
|------|---------------|------|
| DRAFT | 프로모션명, 프로모션구분, 할인금액, 시작일시, 종료일시, 쿠폰적용여부, 대상상품, 비고 | 전체 수정 가능 |
| ACTIVE | 종료일시, 쿠폰적용여부 | 제한 수정 |

---

## 요청 (Request)

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| promotion_name | string | C | 프로모션명 (DRAFT 시 수정 가능) |
| promotion_type | string | C | 프로모션구분 (DRAFT 시 수정 가능) |
| discount_amount | number | C | 할인금액 (DRAFT 시 수정 가능) |
| start_date | datetime | C | 시작일시 (DRAFT 시 수정 가능) |
| end_date | datetime | Y | 종료일시 |
| coupon_applicable | boolean | Y | 쿠폰적용여부 |
| product_codes | string[] | C | 대상 상품코드 배열 (DRAFT 시 수정 가능) |
| memo | string | N | 비고 |

> **C** = 조건부 필수. 상태에 따라 필수 여부가 결정된다.

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| promotion_code | string | 프로모션코드 |
| status | string | 현재 상태 |
| updated_at | datetime | 수정일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 필수 파라미터 누락 또는 유효성 검증 실패 |
| 400 | INVALID_DATE_RANGE | 종료일시가 시작일시 이전 |
| 400 | FIELD_NOT_EDITABLE | 현재 상태에서 수정 불가한 필드 변경 시도 |
| 404 | NOT_FOUND | 존재하지 않는 프로모션코드 |
| 409 | PRODUCT_NOT_APPROVED | 대상 상품 중 승인완료 상태가 아닌 건 존재 |
| 409 | DATE_OVERLAP | 동일 상품에 기간이 중복되는 프로모션 존재 |
| 401 | UNAUTHORIZED | 인증 실패 |
| 403 | FORBIDDEN | 권한 없음 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트 표시: "할인프로모션이 수정되었습니다."
2. 데이터 갱신 (수정된 정보 반영)

### 실패 시

1. 오류 토스트 메시지 표시
2. 폼 데이터 유지 (입력값 손실 방지)
3. 모달 닫힘

---

## 유효성 검사

### 프론트엔드 (모달 표시 전)

**DRAFT 상태:**
- `promotion_name`: 빈 값 불가, 100자 이내
- `promotion_type`: 미선택 불가
- `discount_amount`: 0 초과 정수
- `start_date`, `end_date`: 종료일시 > 시작일시
- `product_codes`: 1건 이상 필수

**ACTIVE 상태:**
- `end_date`: 현재일시 이후
- `coupon_applicable`: 값 필수

### 백엔드

- 프론트엔드 검증 항목 동일 적용
- 상태별 수정 가능 필드 범위 검증
- 기간 변경 시 동일 상품에 대한 기간 중복 재검증
- 대상 상품 승인완료(`APPROVED`) 여부 검증
