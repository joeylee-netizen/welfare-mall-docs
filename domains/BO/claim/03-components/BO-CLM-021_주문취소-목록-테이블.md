---
id: "BO-CLM-021"
title: "주문취소 목록 테이블"
type: component
domain: claim
status: draft
version: "1.0"
created: 2026-04-21
updated: 2026-04-21
author: "기획자"
refs:
  - "BO-CLM-001"
  - "BO-CLM-002"
  - "BO-CLM-010"
  - "BO-CLM-020"
tags: [클레임, 주문취소, 목록, 테이블, 승인, 반려, 일괄처리, 환불재전송]
ui_type: table
design_tokens:
  header_bg: "#f7f8fa"
  row_hover: "#f0f4ff"
  row_selected: "#eff6ff"
  row_urgent_bg: "#fff7ed"
  badge_cancel_requested: "#3b82f6"
  badge_sale_cancel_requested: "#6366f1"
  badge_refund_requested: "#8b5cf6"
  badge_refunded: "#22c55e"
  badge_refund_failed: "#ef4444"
  badge_cancel_withdrawn: "#f97316"
  modal_overlay: "rgba(0, 0, 0, 0.5)"
---

# 주문취소 목록 테이블

## 개요

BO 관리자가 **배송형 주문취소 클레임**을 전담 조회·처리하는 목록 컴포넌트이다. [BO-CLM-001](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md) 메뉴 구조 중 **클레임 관리 > 주문취소 목록**에 해당한다. [BO-CLM-020](./BO-CLM-020_배송상품-클레임-목록-테이블.md)(통합 목록)과 별개 화면으로 운영되며, **BO/PO 작업 흐름**(상품준비중 건 수기 승인, 환불실패 재전송 대상 표시 및 일괄 처리)에 특화되어 있다.

- 조회 대상: `claim_type = CANCEL` 고정 (주문취소/판매취소 모두 포함)
- 차별점(vs CLM-020): (a) 승인 필요 건 상단 강조, (b) 일괄 승인·반려·환불재전송 버튼, (c) 행 내 **개별 처리 버튼** 제공

---

## 검색 필터 구성 (13개)

| No | 필터명 | 데이터 필드 | 입력 유형 | 기본값 | 비고 |
|----|--------|------------|-----------|--------|------|
| 1 | 기간 | applied_at / status_changed_at / completed_at | Select + DateRange + Radio | 클레임신청일, 최근 7일 | 기간유형: 신청일/상태변경일/환불완료일. 퀵: 오늘/7일/1개월/3개월/1년 |
| 2 | 하위 유형 | claim_sub_type | 체크박스 | 전체 | 전체/주문취소(사용자)/판매취소(관리자) |
| 3 | FO 상태 | claim_status_fo | 체크박스 | 전체 | 전체/주문취소/주문취소 철회 |
| 4 | BO 상태 | claim_status_bo | 체크박스 | 전체 | 전체/주문취소 신청/판매취소 신청/환불요청 완료/환불완료/환불실패/주문취소 철회 |
| 5 | 처리 채널 | process_channel | 체크박스 | 전체 | 전체/FO/BO_PROXY/SYSTEM |
| 6 | 접수 채널 | applied_channel | 체크박스 | 전체 | 전체/FO/BO_PROXY/ADMIN |
| 7 | 승인 필요 여부 | approval_required | 라디오 | 전체 | 전체/Yes(대기)/No |
| 8 | 귀책 구분 | liability_party | 체크박스 | 전체 | 전체/구매자/판매사 |
| 9 | 클레임 사유 | reason_code | Select | 전체 | ClaimReasonCode 전체 |
| 10 | 판매사 | vendor_id | 조회 팝업 | - | 팝업 선택 |
| 11 | 고객사 | client_id | Select | 전체 | 멀티태넌트 필터 |
| 12 | 환불 수단 | refund_method | 체크박스 | 전체 | 전체/카드/계좌이체/복지포인트 |
| 13 | 검색어 | keyword | Select + Text | 사용안함 | 사용안함/클레임코드/주문번호/상품명(콤마 복수)/구매자명 |

---

## 컬럼 구성 (26개)

| No | 컬럼명 | 데이터 필드 | 타입 | 정렬 | 비고 |
|----|--------|------------|------|------|------|
| 1 | 체크박스 | - | checkbox | - | 전체 선택 / 개별 선택. 일괄 처리 대상 지정 |
| 2 | No | - | number | - | 행 번호 |
| 3 | 긴급 | approval_required | icon | - | `Yes`일 때 주황 배경 + 경광등 아이콘 |
| 4 | 클레임코드 | claim_code | link | - | 클릭 시 `BO-CLM-042` 상세 팝업 |
| 5 | 하위 유형 | claim_sub_type | text | - | 주문취소(사용자)/판매취소(관리자) |
| 6 | FO 상태 | claim_status_fo | badge | - | |
| 7 | BO 상태 | claim_status_bo | badge | - | |
| 8 | 처리 채널 | process_channel | text | - | |
| 9 | 접수 채널 | applied_channel | text | - | |
| 10 | 주문번호 | order_id | link | - | 클릭 시 주문상세 새 탭 |
| 11 | 출고지 | shipment_name | text | - | |
| 12 | 고객사 | client_name | text | - | |
| 13 | 구매자 | buyer_id | link | - | 구매자ID(이름) |
| 14 | 판매사 | vendor_id | link | - | 판매사코드(판매사명) |
| 15 | 대표 상품명 | representative_product_name | text | - | `상품명 외 N건` 형식 |
| 16 | 대상 수량 | total_item_quantity | number | - | ClaimItem 합계 |
| 17 | 귀책 | liability_party | badge | - | |
| 18 | 사유 | reason_code | text | - | |
| 19 | 환불 총액 | refund_total_amount | number | - | |
| 20 | PG 환불액 | refund_pg_amount | number | - | |
| 21 | 포인트 복원 | refund_point_amount | number | - | |
| 22 | 신청일시 | applied_at | date | desc | 기본 정렬 |
| 23 | 상태변경일 | status_changed_at | date | - | |
| 24 | 환불완료일 | completed_at | date | - | |
| 25 | 승인/반려 | - | button | - | 개별 처리 버튼. 활성: `CANCEL_REQUESTED` + `approval_required = Yes` |
| 26 | 환불재전송 | - | button | - | 개별 재전송 버튼. 활성: `REFUND_FAILED`만 |

> **긴급행 강조:** `approval_required = Yes` AND `claim_status_bo = CANCEL_REQUESTED`인 행은 `row_urgent_bg`(`#fff7ed`) 배경색을 적용하여 시각적으로 강조.

---

## 상단 툴바 (일괄 처리)

| 버튼 | 활성 조건 | 동작 |
|------|-----------|------|
| **일괄 승인** | 선택 행 전체가 `CANCEL_REQUESTED` + `approval_required = Yes` | 확인 모달 → `BO-CLM-035` 반복 호출 |
| **일괄 반려** | 선택 행 전체가 `CANCEL_REQUESTED` + `approval_required = Yes` | 반려사유 입력 모달(공통 적용) → `BO-CLM-036` 반복 호출 |
| **일괄 환불재전송** | 선택 행 전체가 `REFUND_FAILED` | 확인 모달 → `BO-CLM-038` 반복 호출 |
| **엑셀 다운로드** | 항상 활성 | 현재 조회 조건 기준 xlsx 다운로드(최대 10,000건) |

- 일괄 처리 시 서버 응답은 **건별 결과 배열**로 반환되며, 화면은 성공/실패 건수를 토스트로 요약 표시한다.

---

## 모달 구성

### (1) 승인 확인 모달 (일괄 승인)

- **타이틀:** 주문취소 일괄 승인
- **본문:** "선택한 {N}건의 주문취소를 승인하시겠습니까? 승인 시 즉시 환불요청이 진행됩니다."
- **버튼:** [취소] [승인]
- **검증:** 선택 행 중 조건 불일치 건이 있으면 모달 진입 전 에러 토스트.

### (2) 반려 사유 입력 모달 (개별·일괄 반려)

- **타이틀:** 주문취소 반려
- **본문 필드:**
  - 반려사유(Textarea, 10~200자 필수)
  - FO 고객 안내 문구 템플릿(선택): 출고 준비 완료 / 배송 시작 임박 / 기타 → 선택 시 본문 하단 자동 추가
- **버튼:** [취소] [반려 확정]
- **검증:** 반려사유 길이 검사. `반려 확정` 클릭 시 [BO-CLM-036](../04-actions/BO-CLM-036_주문취소-반려.md) 호출.

### (3) 환불 재전송 확인 모달 (개별·일괄)

- **타이틀:** 환불 재전송
- **본문:** "선택한 {N}건의 환불 요청을 재전송합니다. 당일 수동 재시도는 1회로 제한됩니다."
- **버튼:** [취소] [재전송]
- **검증:** 선택 행 전원이 `REFUND_FAILED`인지 확인.

---

## 인터랙션

### (1) 검색

- 검색어 select에서 `사용안함` 이외 선택 시 1~12 필터 비활성화.
- 실행 시 [BO-CLM-031 주문취소 목록 조회](../04-actions/BO-CLM-031_주문취소-목록-조회.md) 호출.

### (2) 행 클릭

- 클레임코드 셀 클릭 → [BO-CLM-042 주문취소 상세 화면](../05-wireframes/BO-CLM-042_주문취소-상세-화면.md)을 모달로 오픈.
- 상세 팝업 닫힘 후 목록을 자동 새로고침(상태 변경 반영).

### (3) 개별 행 버튼

- `승인/반려` 버튼: 조건 충족 시 개별 처리 모달 열기.
- `환불재전송` 버튼: `REFUND_FAILED` 상태만 활성.

### (4) 체크박스

- 헤더 체크박스: 현재 페이지 전체 선택.
- 선택된 행이 1개 이상일 때 툴바의 일괄 처리 버튼이 활성화된다(조건 조합에 따라 개별 버튼만 활성).

---

## 상태 처리

| 상태 | 표시 |
|------|------|
| 로딩 | 상단 진행 바, 기존 목록 dimmed |
| 데이터 없음 | "조회 결과가 없습니다." 중앙 |
| 일괄 처리 진행 중 | 모달 내 진행률(건/총건수), 취소 버튼 비활성 |
| 일괄 처리 부분 실패 | 토스트: "N건 성공, M건 실패. 실패 건은 목록에서 확인해 주세요." |
| 조회 실패 | 에러 토스트 + 재시도 버튼 |
| 권한 부족(403) | 상단 알림: "클레임 관리 권한이 없습니다." |
| 409 상태 충돌 | "대상 건의 상태가 변경되어 처리할 수 없습니다. 목록을 새로고침합니다." + 자동 새로고침 |

---

## 관련 문서

- [BO-CLM-002](../01-policies/BO-CLM-002_주문취소-정책.md) — 주문취소 정책
- [BO-CLM-010](../02-data/BO-CLM-010_클레임-엔티티.md) — 클레임 엔티티
- [BO-CLM-020](./BO-CLM-020_배송상품-클레임-목록-테이블.md) — 통합 클레임 목록
- [BO-CLM-022](./BO-CLM-022_주문취소-상세-폼.md) — 주문취소 상세 폼
- [BO-CLM-031](../04-actions/BO-CLM-031_주문취소-목록-조회.md) — 주문취소 목록 조회
- [BO-CLM-035](../04-actions/BO-CLM-035_주문취소-승인.md) — 주문취소 승인
- [BO-CLM-036](../04-actions/BO-CLM-036_주문취소-반려.md) — 주문취소 반려
- [BO-CLM-038](../04-actions/BO-CLM-038_환불-재전송.md) — 환불 재전송
- [BO-CLM-041](../05-wireframes/BO-CLM-041_주문취소-목록-화면.md) — 주문취소 목록 화면
