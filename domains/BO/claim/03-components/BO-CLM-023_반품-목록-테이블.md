---
id: "BO-CLM-023"
title: "반품 목록 테이블"
type: component
domain: claim
status: draft
version: "1.0"
created: 2026-04-22
updated: 2026-04-22
author: "기획자"
refs:
  - "BO-CLM-001"
  - "BO-CLM-003"
  - "BO-CLM-010"
tags: [클레임, 반품, 목록, 테이블, 승인, 거절, 수거요청, 환불승인, 검수거부, 일괄처리]
ui_type: table
design_tokens:
  header_bg: "#f7f8fa"
  row_hover: "#f0f4ff"
  row_selected: "#eff6ff"
  row_urgent_bg: "#fff7ed"
  badge_return_requested: "#3b82f6"
  badge_return_received: "#14b8a6"
  badge_return_pickup: "#8b5cf6"
  badge_return_picked: "#a855f7"
  badge_return_collected: "#f59e0b"
  badge_refund_requested: "#6366f1"
  badge_refunded: "#22c55e"
  badge_refund_fail: "#ef4444"
  badge_return_withdrawn: "#f97316"
  badge_liability_b: "#64748b"
  badge_liability_s: "#f43f5e"
  badge_inspection_passed: "#22c55e"
  badge_inspection_failed: "#ef4444"
  modal_overlay: "rgba(0, 0, 0, 0.5)"
---

# 반품 목록 테이블

## 개요

BO 관리자가 **배송형 반품 클레임**을 전담 조회·처리하는 목록 컴포넌트이다. [BO-CLM-001](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md) 메뉴 구조 중 **클레임 관리 > 반품 목록**에 해당한다. BO/PO 작업 흐름에 특화되어 있다: (a) 반품신청 건 수기 승인·거절, (b) 반품접수 건 수거요청(반송장 등록), (c) 반품회수완료 건 환불승인·검수거부, (d) 환불실패 건 재전송. 일괄 처리 지원.

- 조회 대상: `claim_type = RETURN` 고정
- 차별점(vs CLM-020): BO 작업 단계별 긴급 행 강조, 각 전이 단계에 맞는 **개별 처리 버튼** 제공, 일괄 처리 버튼 다수

---

## 검색 필터 구성 (14개)

| No | 필터명 | 데이터 필드 | 입력 유형 | 기본값 | 비고 |
|----|--------|------------|-----------|--------|------|
| 1 | 기간 | applied_at / status_changed_at / pickup_requested_at / completed_at | Select + DateRange + Radio | 클레임신청일, 최근 7일 | 기간유형: 신청일/상태변경일/수거요청일/완료일 |
| 2 | 하위 유형 | claim_sub_type | 체크박스 | 전체 | 전체/반품(사용자)/반품(관리자 대행) |
| 3 | FO 상태 | claim_status_fo | 체크박스 | 전체 | 전체/반품신청/반품접수/반품수거중/반품상품수거/반품회수완료/반품완료/반품철회 |
| 4 | BO 상태 | claim_status_bo | 체크박스 | 전체 | 전체/반품신청/반품접수/반품수거중/반품상품수거/반품회수완료/환불요청완료/환불완료/환불실패/반품철회 |
| 5 | 처리 채널 | process_channel | 체크박스 | 전체 | 전체/FO/BO_PROXY/SYSTEM |
| 6 | 접수 채널 | applied_channel | 체크박스 | 전체 | 전체/FO/BO_PROXY |
| 7 | 귀책 | liability_party | 체크박스 | 전체 | 전체/구매자/판매사 |
| 8 | 클레임 사유 | reason_code | Select | 전체 | 반품 허용 사유(단순변심/색상사이즈불만/상품불량/상품누락/다른상품/기타) |
| 9 | 검수 결과 | inspection_result | 체크박스 | 전체 | 전체/PASSED/FAILED_DEFECT/FAILED_MISSING/FAILED_OTHER/미검수 |
| 10 | 판매사 | vendor_id | 조회 팝업 | - | 팝업 선택 |
| 11 | 고객사 | client_id | Select | 전체 | 멀티태넌트 필터 |
| 12 | 환불 수단 | refund_method | 체크박스 | 전체 | 전체/카드/계좌이체/복지포인트 |
| 13 | 택배사 | pickup_carrier_code | Select | 전체 | 반송 택배사 |
| 14 | 검색어 | keyword | Select + Text | 사용안함 | 사용안함/클레임코드/주문번호/상품명/구매자명/반송장번호 |

---

## 컬럼 구성 (27개)

| No | 컬럼명 | 데이터 필드 | 타입 | 정렬 | 비고 |
|----|--------|------------|------|------|------|
| 1 | 체크박스 | - | checkbox | - | 전체/개별 선택. 일괄 처리 대상 |
| 2 | No | - | number | - | 행 번호 |
| 3 | 긴급 | - | icon | - | 관리자 액션 대기 건 ⚠️ 표시 + 행 배경 강조 |
| 4 | 클레임코드 | claim_code | link | - | 클릭 시 `BO-CLM-044` 상세 팝업 |
| 5 | 하위 유형 | claim_sub_type | text | - | 반품(사용자)/반품(대행) |
| 6 | FO 상태 | claim_status_fo | badge | - | |
| 7 | BO 상태 | claim_status_bo | badge | - | |
| 8 | 처리 채널 | process_channel | text | - | |
| 9 | 접수 채널 | applied_channel | text | - | |
| 10 | 주문번호 | order_id | link | - | 클릭 시 주문상세 새 탭 |
| 11 | 고객사 | client_name | text | - | |
| 12 | 구매자 | buyer_id | link | - | 구매자ID(이름) |
| 13 | 판매사 | vendor_id | link | - | 판매사코드(판매사명) |
| 14 | 대표 상품명 | representative_product_name | text | - | "상품명 외 N건" |
| 15 | 대상 수량 | total_item_quantity | number | - | |
| 16 | 귀책 | liability_party | badge | - | |
| 17 | 사유 | reason_code | text | - | |
| 18 | 반송장 | pickup_tracking_number | text | - | 택배사(송장번호). 미등록 시 "—" |
| 19 | 수거 요청일 | pickup_requested_at | date | - | |
| 20 | 수거일 | pickup_picked_up_at | date | - | 택배사 수령 |
| 21 | 회수 완료일 | pickup_returned_at | date | - | 판매사 회수 |
| 22 | 검수 결과 | inspection_result | badge | - | 통과(green)/거부(red)/미검수(gray) |
| 23 | 환불 총액 | refund_total_amount | number | - | |
| 24 | 신청일시 | applied_at | date | desc | 기본 정렬 |
| 25 | 상태변경일 | status_changed_at | date | - | |
| 26 | 완료일 | completed_at | date | - | |
| 27 | 처리 | - | button | - | 상태별 개별 처리 버튼(승인/거절/수거요청/환불승인/검수거부/수기완료/재전송) |

> **긴급행 강조:** `RETURN_REQUESTED` (승인 대기) 또는 `RETURN_COLLECTED` (검수 대기) 또는 `REFUND_FAILED` (재전송 대기) 행은 `row_urgent_bg`(`#fff7ed`) 배경색 + ⚠️ 아이콘.

---

## 상단 툴바 (일괄 처리)

| 버튼 | 활성 조건 | 동작 |
|------|-----------|------|
| **일괄 승인** | 선택 행 전체가 `RETURN_REQUESTED` | 확인 모달 → `BO-CLM-03A` 반복 호출 |
| **일괄 거절** | 선택 행 전체가 `RETURN_REQUESTED` | 반려사유 입력 모달 → `BO-CLM-03B` 반복 호출 |
| **일괄 수거요청** | 선택 행 전체가 `RETURN_RECEIVED` | 반송장 입력 모달(행별 송장번호 입력 테이블) → `BO-CLM-03C` 반복 호출 |
| **일괄 환불승인** | 선택 행 전체가 `RETURN_COLLECTED` | 검수 결과 `PASSED` 확인 모달 → `BO-CLM-03D` 반복 호출 |
| **일괄 환불재전송** | 선택 행 전체가 `REFUND_FAILED` | 확인 모달 → `BO-CLM-038` 반복 호출 |
| **엑셀 다운로드** | 항상 활성 | 현재 조회 조건 기준 xlsx (최대 10,000건) |

- 일괄 처리 응답은 건별 결과 배열로 반환, 성공/실패 건수를 토스트로 요약 표시.

---

## 모달 구성

### (1) 승인 확인 모달 (일괄/단건 승인)

- **타이틀:** 반품 승인
- **본문:** "선택한 {N}건의 반품 신청을 승인합니다. 승인 시 '반품접수' 상태로 전환되며, 이후 반송장 등록이 필요합니다."
- **버튼:** [취소] [승인]

### (2) 거절 사유 입력 모달 (일괄/단건 거절)

- **타이틀:** 반품 거절
- **본문 필드:**
  - 반려사유 (textarea, 10~200자 필수)
  - FO 안내 문구 템플릿: 반품 불가 사유 5가지 라디오 ([BO-CLM-003 §1-(2)](../01-policies/BO-CLM-003_반품-정책.md#1-반품-개요-및-조건))
- **버튼:** [취소] [거절 확정]

### (3) 수거요청 모달 (일괄/단건)

- **타이틀:** 반품 수거요청 (반송장 등록)
- **본문 필드:**
  - 택배사 선택 (Select: CJ/로젠/한진/우체국/기타)
  - 송장번호 입력 (일괄의 경우 행별 테이블 입력)
  - 수거 요청일 (기본값: 오늘)
- **버튼:** [취소] [수거요청]

### (4) 환불승인 모달 (일괄/단건)

- **타이틀:** 반품 검수 통과·환불승인
- **본문 필드:**
  - 검수 결과: `PASSED` 고정 표시
  - 검수 메모 (textarea, 0~1000자 선택)
- **본문:** "선택한 {N}건의 검수를 통과 처리하고 환불을 진행합니다."
- **버튼:** [취소] [환불승인]

### (5) 검수거부 모달 (개별)

- **타이틀:** 반품 검수거부
- **본문 필드:**
  - 거부 사유 (Radio: `FAILED_DEFECT` / `FAILED_MISSING` / `FAILED_OTHER`)
  - 검수 메모 (textarea, `FAILED_OTHER` 선택 시 필수 10~1000자)
- **본문:** "검수거부 시 반품이 철회되며 원상품이 구매자에게 재배송됩니다."
- **버튼:** [취소] [검수거부 확정]

### (6) 환불 재전송 확인 모달

- **타이틀:** 환불 재전송
- **본문:** "선택한 {N}건의 환불 요청을 재전송합니다. 당일 수동 재시도는 1회로 제한됩니다."
- **버튼:** [취소] [재전송]

---

## 인터랙션

### (1) 검색

- 검색어 select에서 `사용안함` 외 선택 시 1~13번 필터 비활성.
- 실행 시 [BO-CLM-030 클레임 목록 조회](../04-actions/BO-CLM-030_클레임-목록-조회.md) 호출(claim_type=RETURN 고정).

### (2) 행 클릭

- 클레임코드 셀 클릭 → [BO-CLM-044 반품 상세 화면](../05-wireframes/BO-CLM-044_반품-상세-화면.md) 모달 오픈.
- 상세 팝업 처리 후 목록 자동 새로고침.

### (3) 개별 행 버튼

- `RETURN_REQUESTED`: 승인/거절 버튼 표시
- `RETURN_RECEIVED`: 수거요청 버튼 표시
- `RETURN_COLLECTED`: 환불승인/검수거부 버튼 표시
- `REFUND_FAILED`: 수기완료/재전송 버튼 표시
- 기타 상태: `-` (버튼 없음, 조회만)

### (4) 체크박스·일괄 처리

- 헤더 체크박스: 현재 페이지 전체 선택.
- 선택 행의 `claim_status_bo`가 동일할 때에만 해당 일괄 버튼 활성(조건 조합).

---

## 상태 처리

| 상태 | 표시 |
|------|------|
| 로딩 | 상단 진행 바, 기존 목록 dimmed |
| 데이터 없음 | "조회 결과가 없습니다." 중앙 |
| 일괄 처리 진행 | 모달 내 진행률(건/총건수) |
| 부분 실패 | 토스트: "N건 성공, M건 실패" |
| 상태 충돌(409) | "대상 건의 상태가 변경되었습니다. 새로고침합니다." + 자동 새로고침 |
| 반송장 중복 입력(409) | "이미 반송장이 등록된 건이 있습니다." |
| 일일 재시도 초과 | 재전송 버튼 비활성 + 툴팁 |
| 조회 실패 | 에러 토스트 + 재시도 버튼 |

---

## 관련 문서

- [BO-CLM-003](../01-policies/BO-CLM-003_반품-정책.md) — 반품 정책
- [BO-CLM-010](../02-data/BO-CLM-010_클레임-엔티티.md) — 클레임 엔티티(반품 상태값·검수 결과)
- [BO-CLM-020](./BO-CLM-020_배송상품-클레임-목록-테이블.md) — 통합 클레임 목록
- [BO-CLM-024](./BO-CLM-024_반품-상세-폼.md) — 반품 상세 폼(모달)
- [BO-CLM-030](../04-actions/BO-CLM-030_클레임-목록-조회.md) — 목록 조회
- [BO-CLM-03A](../04-actions/BO-CLM-03A_반품교환-승인.md) — 반품/교환 승인
- [BO-CLM-03B](../04-actions/BO-CLM-03B_반품교환-거절.md) — 반품/교환 거절
- [BO-CLM-03C](../04-actions/BO-CLM-03C_반품교환-수거요청.md) — 수거요청(반송장 등록)
- [BO-CLM-03D](../04-actions/BO-CLM-03D_반품-환불승인.md) — 반품 환불승인
- [BO-CLM-03F](../04-actions/BO-CLM-03F_반품교환-검수거부.md) — 검수거부
- [BO-CLM-037](../04-actions/BO-CLM-037_환불-수기-완료.md) — 환불 수기 완료(재사용)
- [BO-CLM-038](../04-actions/BO-CLM-038_환불-재전송.md) — 환불 재전송(재사용)
- [BO-CLM-043](../05-wireframes/BO-CLM-043_반품-목록-화면.md) — 반품 목록 화면
