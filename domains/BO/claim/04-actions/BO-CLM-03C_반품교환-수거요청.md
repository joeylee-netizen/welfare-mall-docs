---
id: "BO-CLM-03C"
title: "반품/교환 수거요청"
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
tags: [클레임, 반품, 교환, 수거, 반송장, 택배, API]
trigger: submit
method: PATCH
endpoint: "/api/v1/claims/{claim_id}/pickup/request"
---

# 반품/교환 수거요청

## 트리거 조건

- 반품/교환 목록의 개별/일괄 `수거요청` 버튼 → 반송장 등록 모달 → `수거요청`
- 상세 모달의 `수거요청` 버튼 → 반송장 등록 모달
- 대상 상태: `RETURN_RECEIVED` 또는 `EXCHANGE_RECEIVED`

반품·교환 공용. 반송장 등록 즉시 택배사 수거 API 연동.

---

## 요청 (Request)

### Path Parameters

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| claim_id | number | Y | 클레임 ID |

### Body (JSON)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| pickup_carrier_code | string | Y | 반송 택배사 코드 (CJ/로젠/한진/우체국/기타) |
| pickup_tracking_number | string | Y | 송장번호(택배사별 형식 검증) |
| pickup_requested_at | date | Y | 수거 요청일 (기본값: 오늘) |
| memo | string | N | 수거 메모(0~500자) |

---

## 응답 (Response)

### 성공 (200 OK)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| claim_id | number | 클레임 ID |
| claim_type | string | `RETURN` \| `EXCHANGE` |
| claim_status_bo | string | `RETURN_PICKUP_IN_PROGRESS` 또는 `EXCHANGE_PICKUP_IN_PROGRESS` |
| claim_status_fo | string | 동일 |
| pickup_carrier_name | string | 택배사명 스냅샷 |
| pickup_tracking_number | string | 저장된 송장번호 |
| pickup_requested_at | datetime | 수거 요청 일시 |

### 실패

| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_CARRIER | 미지원 택배사 코드 |
| 400 | INVALID_TRACKING_FORMAT | 송장번호 형식 오류(택배사별 정규식) |
| 400 | PICKUP_DATE_INVALID | 수거 요청일이 오늘 이전 또는 30일 이후 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | CLAIM_NOT_FOUND | 존재하지 않는 클레임 |
| 409 | INVALID_STATUS_TRANSITION | `RETURN_RECEIVED` / `EXCHANGE_RECEIVED` 외 상태 |
| 409 | TRACKING_NUMBER_DUPLICATE | 동일 송장번호가 이미 다른 클레임에 등록됨 |
| 500 | CARRIER_API_ERROR | 택배사 API 호출 실패 |
| 500 | INTERNAL_ERROR | 서버 오류 |

---

## 후속 처리

### 성공 시

1. 성공 토스트: "수거 요청이 등록되었습니다. ({택배사명} {송장번호})"
2. 해당 행의 상태를 `*_PICKUP_IN_PROGRESS`로 갱신, 반송장/수거 요청일 표시
3. 상세 재조회 → 섹션 4(수거·검수 정보)에 반송장 정보 반영
4. **택배사 연동(비동기):** 수거 예약 API 호출. 성공 시 ClaimHistory에 수거 예약 이벤트 기록
5. ClaimHistory에 `(RECEIVED → PICKUP_IN_PROGRESS)` + 처리자 기록
6. FO 구매자에게 수거 예정 알림톡 발송(택배사/송장번호 포함)

### 실패 시

1. 오류 토스트
2. 400 입력 오류 시 모달 유지, 재입력 가능
3. 500 CARRIER_API_ERROR: 상태는 전환 성공으로 기록되나 택배사 연동 실패 플래그 + CS 재시도 큐 등록

---

## 유효성 검사

### 프론트엔드

- 버튼 활성 조건: `claim_status_bo = RETURN_RECEIVED` 또는 `EXCHANGE_RECEIVED`
- 택배사 Select 필수
- 송장번호 공백·특수문자 제외 확인
- 수거 요청일: 오늘 ~ +30일 범위

### 백엔드

- 상태 검증
- 택배사 코드 화이트리스트 검증
- 송장번호 택배사별 정규식 검증
- 송장번호 UNIQUE 검증(진행 중 클레임 간)
- 택배사 API 호출(비동기 큐잉, 실패 시 재시도 3회)
- ClaimHistory 자동 기록
