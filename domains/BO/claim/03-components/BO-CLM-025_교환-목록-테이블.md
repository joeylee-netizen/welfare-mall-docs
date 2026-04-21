---
id: "BO-CLM-025"
title: "교환 목록 테이블"
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
  - "BO-CLM-004"
  - "BO-CLM-010"
tags: [클레임, 교환, 목록, 테이블, 승인, 거절, 수거요청, 최종승인, 검수거부, 일괄처리]
ui_type: table
design_tokens:
  header_bg: "#f7f8fa"
  row_hover: "#f0f4ff"
  row_selected: "#eff6ff"
  row_urgent_bg: "#fff7ed"
  badge_exchange_requested: "#3b82f6"
  badge_exchange_received: "#14b8a6"
  badge_exchange_pickup: "#8b5cf6"
  badge_exchange_picked: "#a855f7"
  badge_exchange_collected: "#f59e0b"
  badge_exchange_final: "#22c55e"
  badge_exchange_withdrawn: "#f97316"
  badge_liability_b: "#64748b"
  badge_liability_s: "#f43f5e"
  badge_inspection_passed: "#22c55e"
  badge_inspection_failed: "#ef4444"
  modal_overlay: "rgba(0, 0, 0, 0.5)"
---

# 교환 목록 테이블

## 개요

BO 관리자가 **배송형 교환 클레임**을 전담 조회·처리하는 목록 컴포넌트. [BO-CLM-001](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md) 메뉴 구조 중 **클레임 관리 > 교환 목록**에 해당. 반품 목록([BO-CLM-023](./BO-CLM-023_반품-목록-테이블.md))과 유사 구조지만 **교환 특화 컬럼**(교환 대상 옵션 / 교환주문번호 / 재배송 송장)과 **최종승인** 단계가 추가된다.

- 조회 대상: `claim_type = EXCHANGE` 고정
- 차별점(vs CLM-023): 교환 최종승인(회수완료→교환최종승인), 환불 프로세스 없음, 교환주문번호 발급 확인

---

## 검색 필터 구성 (14개)

| No | 필터명 | 데이터 필드 | 입력 유형 | 기본값 | 비고 |
|----|--------|------------|-----------|--------|------|
| 1 | 기간 | applied_at / status_changed_at / pickup_requested_at / exchange_final_approved_at | Select + DateRange + Radio | 클레임신청일, 최근 7일 | 기간유형: 신청일/상태변경일/수거요청일/최종승인일 |
| 2 | 하위 유형 | claim_sub_type | 체크박스 | 전체 | 전체/교환(사용자)/교환(관리자 대행) |
| 3 | FO 상태 | claim_status_fo | 체크박스 | 전체 | 전체/교환신청/교환접수/교환수거중/교환상품수거/교환회수완료/교환최종승인/교환철회 |
| 4 | BO 상태 | claim_status_bo | 체크박스 | 전체 | 전체/교환신청/교환접수/교환수거중/교환상품수거/교환회수완료/교환최종승인/교환철회 |
| 5 | 처리 채널 | process_channel | 체크박스 | 전체 | 전체/FO/BO_PROXY/SYSTEM |
| 6 | 접수 채널 | applied_channel | 체크박스 | 전체 | 전체/FO/BO_PROXY |
| 7 | 귀책 | liability_party | 체크박스 | 전체 | 전체/구매자/판매사 |
| 8 | 클레임 사유 | reason_code | Select | 전체 | 교환 허용 사유(단순변심/색상사이즈불만/상품불량/상품누락/다른상품) |
| 9 | 검수 결과 | inspection_result | 체크박스 | 전체 | 전체/PASSED/FAILED_*/미검수 |
| 10 | 판매사 | vendor_id | 조회 팝업 | - | 팝업 선택 |
| 11 | 고객사 | client_id | Select | 전체 | 멀티태넌트 필터 |
| 12 | 택배사 | pickup_carrier_code | Select | 전체 | 반송 택배사 |
| 13 | 교환주문번호 | exchange_order_id | Text | - | 완료건 역조회 |
| 14 | 검색어 | keyword | Select + Text | 사용안함 | 사용안함/클레임코드/주문번호/상품명/구매자명/반송장번호/교환주문번호 |

---

## 컬럼 구성 (28개)

| No | 컬럼명 | 데이터 필드 | 타입 | 정렬 | 비고 |
|----|--------|------------|------|------|------|
| 1 | 체크박스 | - | checkbox | - | 전체/개별 선택. 일괄 처리 대상 |
| 2 | No | - | number | - | 행 번호 |
| 3 | 긴급 | - | icon | - | 관리자 액션 대기 건 ⚠️ |
| 4 | 클레임코드 | claim_code | link | - | 클릭 시 `BO-CLM-046` 상세 팝업 |
| 5 | 하위 유형 | claim_sub_type | text | - | 교환(사용자)/교환(대행) |
| 6 | FO 상태 | claim_status_fo | badge | - | |
| 7 | BO 상태 | claim_status_bo | badge | - | |
| 8 | 처리 채널 | process_channel | text | - | |
| 9 | 접수 채널 | applied_channel | text | - | |
| 10 | 주문번호 | order_id | link | - | 원주문 새 탭 |
| 11 | 고객사 | client_name | text | - | |
| 12 | 구매자 | buyer_id | link | - | |
| 13 | 판매사 | vendor_id | link | - | |
| 14 | 원상품 | representative_product_name | text | - | 원주문 상품명 |
| 15 | 원옵션 | original_option_name | text | - | 원주문 옵션 스냅샷 |
| 16 | 교환 대상 옵션 | exchange_target_option_name | text | - | 동일 상품 내 다른 옵션 |
| 17 | 수량 | total_item_quantity | number | - | |
| 18 | 귀책 | liability_party | badge | - | |
| 19 | 사유 | reason_code | text | - | |
| 20 | 반송장 | pickup_tracking_number | text | - | 택배사(송장). 미등록 시 "—" |
| 21 | 수거 요청일 | pickup_requested_at | date | - | |
| 22 | 회수 완료일 | pickup_returned_at | date | - | 판매사 회수 |
| 23 | 검수 결과 | inspection_result | badge | - | |
| 24 | 교환주문번호 | exchange_order_id | link | - | 최종승인 후 발급. 클릭 → 주문상세 |
| 25 | 재배송 송장 | exchange_redelivery_tracking | text | - | 주문 모듈에서 동기화 |
| 26 | 신청일시 | applied_at | date | desc | 기본 정렬 |
| 27 | 최종승인일 | exchange_final_approved_at | date | - | |
| 28 | 처리 | - | button | - | 상태별 개별 처리 버튼 |

> **긴급행 강조:** `EXCHANGE_REQUESTED` (승인 대기) 또는 `EXCHANGE_COLLECTED` (검수·최종승인 대기) 행은 `row_urgent_bg` 배경 + ⚠️ 아이콘.

---

## 상단 툴바 (일괄 처리)

| 버튼 | 활성 조건 | 동작 |
|------|-----------|------|
| **일괄 승인** | 선택 전원 `EXCHANGE_REQUESTED` | [BO-CLM-03A](../04-actions/BO-CLM-03A_반품교환-승인.md) 반복 호출 |
| **일괄 거절** | 선택 전원 `EXCHANGE_REQUESTED` | 반려사유 모달 → [BO-CLM-03B](../04-actions/BO-CLM-03B_반품교환-거절.md) 반복 호출 (재고 복원) |
| **일괄 수거요청** | 선택 전원 `EXCHANGE_RECEIVED` | 반송장 입력 모달 → [BO-CLM-03C](../04-actions/BO-CLM-03C_반품교환-수거요청.md) 반복 호출 |
| **일괄 최종승인** | 선택 전원 `EXCHANGE_COLLECTED` | 검수 통과 확인 모달 → [BO-CLM-03E](../04-actions/BO-CLM-03E_교환-최종승인.md) 반복 호출 (교환주문 발급) |
| **엑셀 다운로드** | 항상 활성 | xlsx (최대 10,000건) |

- 교환은 환불 프로세스가 없으므로 **환불재전송 일괄 버튼 없음**.

---

## 모달 구성

### (1) 승인 확인 모달

- **타이틀:** 교환 승인
- **본문:** "선택한 {N}건의 교환 신청을 승인합니다. 승인 시 '교환접수' 상태로 전환되며, 이후 반송장 등록이 필요합니다. 교환 대상 재고는 이미 선차감 상태입니다."
- **버튼:** [취소] [승인]

### (2) 거절 사유 입력 모달

- **타이틀:** 교환 거절
- **본문 필드:**
  - 반려사유 (10~200자 필수)
  - FO 안내 문구 템플릿: 교환 불가 사유 라디오
- **안내:** "거절 시 선차감된 교환 재고가 자동 복원됩니다."
- **버튼:** [취소] [거절 확정]

### (3) 수거요청 모달

- **타이틀:** 교환 수거요청 (반송장 등록)
- **본문 필드:** 택배사 / 송장번호 / 수거 요청일 (반품과 동일 구조)
- **버튼:** [취소] [수거요청]

### (4) 최종승인 모달

- **타이틀:** 교환 최종승인
- **본문 필드:**
  - 검수 결과: `PASSED` 고정
  - 검수 메모 (0~1000자 선택)
- **본문:** "선택한 {N}건의 검수를 통과 처리하고 교환을 최종 승인합니다. 최종 승인 시 교환주문번호가 자동 발급되며 교환 상품이 재배송됩니다."
- **버튼:** [취소] [최종승인]

### (5) 검수거부 모달 (개별)

- **타이틀:** 교환 검수거부
- **본문 필드:** 거부 사유(Enum) / 검수 메모
- **본문:** "검수거부 시 교환이 철회되며, 선차감된 교환 재고가 복원되고 원상품이 구매자에게 재배송됩니다."
- **버튼:** [취소] [검수거부 확정]

---

## 인터랙션

### (1) 검색

- 검색어 select에서 `사용안함` 외 선택 시 1~13번 필터 비활성.
- 실행 시 [BO-CLM-030](../04-actions/BO-CLM-030_클레임-목록-조회.md) 호출(claim_type=EXCHANGE 고정).

### (2) 행 클릭

- 클레임코드 셀 클릭 → [BO-CLM-046 교환 상세 화면](../05-wireframes/BO-CLM-046_교환-상세-화면.md) 모달 오픈.
- 교환주문번호 셀 클릭(최종승인 후) → 주문상세 새 탭.

### (3) 개별 행 버튼

- `EXCHANGE_REQUESTED`: 승인/거절
- `EXCHANGE_RECEIVED`: 수거요청
- `EXCHANGE_COLLECTED`: 최종승인/검수거부
- `EXCHANGE_FINAL_APPROVED` / `EXCHANGE_WITHDRAWN`: "—" (종결)

---

## 상태 처리

| 상태 | 표시 |
|------|------|
| 로딩 | 상단 진행 바, 기존 목록 dimmed |
| 데이터 없음 | "조회 결과가 없습니다." 중앙 |
| 일괄 처리 진행 | 모달 내 진행률 |
| 부분 실패 | 토스트 요약 |
| 409 상태 충돌 | 자동 새로고침 |
| **재고 복원 실패** | 토스트: "재고 복원 실패. 관리자에게 문의하세요." + 이력 남김 |
| **교환주문 발급 실패** | 토스트: "교환주문 발급에 실패했습니다. 재시도 가능." + 버튼 활성 유지 |
| 조회 실패 | 에러 토스트 + 재시도 |

---

## 관련 문서

- [BO-CLM-004](../01-policies/BO-CLM-004_교환-정책.md) — 교환 정책
- [BO-CLM-003](../01-policies/BO-CLM-003_반품-정책.md) — 반품 조건(교환 조건 공유)
- [BO-CLM-010](../02-data/BO-CLM-010_클레임-엔티티.md) — 클레임 엔티티(교환 필드)
- [BO-CLM-020](./BO-CLM-020_배송상품-클레임-목록-테이블.md) — 통합 클레임 목록
- [BO-CLM-026](./BO-CLM-026_교환-상세-폼.md) — 교환 상세 폼(모달)
- [BO-CLM-030](../04-actions/BO-CLM-030_클레임-목록-조회.md)
- [BO-CLM-03A](../04-actions/BO-CLM-03A_반품교환-승인.md) / [BO-CLM-03B](../04-actions/BO-CLM-03B_반품교환-거절.md) / [BO-CLM-03C](../04-actions/BO-CLM-03C_반품교환-수거요청.md) / [BO-CLM-03E](../04-actions/BO-CLM-03E_교환-최종승인.md) / [BO-CLM-03F](../04-actions/BO-CLM-03F_반품교환-검수거부.md)
- [BO-CLM-045](../05-wireframes/BO-CLM-045_교환-목록-화면.md) — 교환 목록 화면
