---
id: "BO-CLM-020"
title: "배송상품 클레임 목록 테이블"
type: component
domain: claim
status: draft
version: "2.0"
created: 2026-04-21
updated: 2026-04-22
author: "기획자"
refs:
  - "BO-CLM-001"
  - "BO-CLM-002"
  - "BO-CLM-003"
  - "BO-CLM-004"
  - "BO-CLM-010"
tags: [클레임, 통합, 배송, 목록, 테이블, 주문취소, 반품, 교환]
ui_type: table
design_tokens:
  header_bg: "#f7f8fa"
  row_hover: "#f0f4ff"
  row_selected: "#eff6ff"
  badge_cancel_requested: "#3b82f6"
  badge_sale_cancel_requested: "#6366f1"
  badge_refund_requested: "#8b5cf6"
  badge_refunded: "#22c55e"
  badge_refund_failed: "#ef4444"
  badge_cancel_withdrawn: "#f97316"
  badge_return_requested: "#3b82f6"
  badge_return_received: "#14b8a6"
  badge_return_pickup: "#8b5cf6"
  badge_return_collected: "#f59e0b"
  badge_exchange_requested: "#3b82f6"
  badge_exchange_received: "#14b8a6"
  badge_exchange_final: "#22c55e"
  badge_withdrawn: "#f97316"
  badge_liability_buyer: "#64748b"
  badge_liability_seller: "#f43f5e"
  badge_inspection_passed: "#22c55e"
  badge_inspection_failed: "#ef4444"
  modal_overlay: "rgba(0, 0, 0, 0.5)"
---

# 배송상품 클레임 목록 테이블

## 개요

BO 관리자가 **배송형 상품의 전체 클레임**(주문취소·반품·교환)을 **통합 조회**하는 컴포넌트이다. [BO-CLM-001](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md) 메뉴 구조 중 **클레임 관리 > 배송상품 클레임 목록**에 해당한다. 클레임 유형은 **필터(체크박스)** 로 선택하며, 유형별 전용 화면(주문취소/반품/교환)은 LNB에서 별도 진입한다.

- 조회 대상: `claim_type ∈ {CANCEL, RETURN, EXCHANGE}` — **클레임 유형 필터**로 제한
- 기본 필터: `전체` (모든 유형)
- 목록 행 클릭 시: 행의 `claim_type`에 따라 상세 팝업 분기
  - `CANCEL` → [BO-CLM-042](../05-wireframes/BO-CLM-042_주문취소-상세-화면.md)
  - `RETURN` → [BO-CLM-044](../05-wireframes/BO-CLM-044_반품-상세-화면.md)
  - `EXCHANGE` → [BO-CLM-046](../05-wireframes/BO-CLM-046_교환-상세-화면.md)

> **v2.0 변경:** 기존 탭 UI(주문취소/반품/교환) 삭제. 클레임 유형 전환은 **LNB 별도 메뉴**로 이동하고 본 화면은 **통합 필터 기반 단일 목록**으로 단순화.

---

## 검색 필터 구성 (15개)

| No | 필터명 | 데이터 필드 | 입력 유형 | 기본값 | 비고 |
|----|--------|------------|-----------|--------|------|
| 1 | 기간 | applied_at / status_changed_at / completed_at | Select + DateRange + Radio | 클레임신청일, 최근 7일 | 기간유형: 클레임신청일/상태변경일/완료일. 퀵선택: 오늘/7일/1개월/3개월/1년 |
| 2 | **클레임 유형** | claim_type | 체크박스 | 전체 | **전체/주문취소/반품/교환** (v2.0 탭→필터 전환) |
| 3 | 클레임 하위 유형 | claim_sub_type | 체크박스 | 전체 | 전체/주문취소(사용자)/판매취소(관리자)/반품(사용자)/반품(대행)/교환(사용자)/교환(대행) |
| 4 | FO 클레임 상태 | claim_status_fo | 체크박스 | 전체 | 전 유형 FO 상태값 통합 |
| 5 | BO 클레임 상태 | claim_status_bo | 체크박스 | 전체 | 전 유형 BO 상태값 통합 |
| 6 | 처리 채널 | process_channel | 체크박스 | 전체 | 전체/FO/BO_PROXY/SYSTEM |
| 7 | 접수 채널 | applied_channel | 체크박스 | 전체 | 전체/FO/BO_PROXY/ADMIN |
| 8 | 귀책 구분 | liability_party | 체크박스 | 전체 | 전체/구매자/판매사 |
| 9 | 클레임 사유 | reason_code | Select | 전체 | ClaimReasonCode 전체 값 |
| 10 | 판매사 | vendor_id | 조회 팝업 | - | 팝업 선택 → 판매사명 출력 |
| 11 | 고객사 | client_id | Select | 전체 | 멀티태넌트 필터(관리자 권한 범위 내) |
| 12 | 구매자 | buyer_id | 조회 팝업 | - | 팝업 선택 → 구매자ID(이름) 출력 |
| 13 | 환불 수단 | refund_method | 체크박스 | 전체 | 전체/카드/계좌이체/복지포인트 (주문취소·반품만 적용) |
| 14 | 승인 필요 여부 | approval_required | 라디오 | 전체 | 전체/Yes/No — 수기 승인 대기 건 필터 |
| 15 | 검색어 | keyword | Select + Text | 사용안함 | 사용안함/클레임코드/주문번호/상품명(콤마 복수)/구매자명/반송장번호/교환주문번호. 사용안함 외 선택 시 1~14번 비활성화 |

- `검색` 버튼 클릭 또는 Enter 키로 조회를 실행한다.
- `초기화` 버튼 클릭 시 모든 필터를 기본값으로 되돌린다.

---

## 컬럼 구성 (29개)

| No | 컬럼명 | 데이터 필드 | 타입 | 정렬 | 비고 |
|----|--------|------------|------|------|------|
| 1 | No | - | number | - | 행 번호(페이지 기준 자동 채번) |
| 2 | 클레임코드 | claim_code | link | - | 클릭 시 상세 팝업(주문취소=`BO-CLM-042`) |
| 3 | 클레임 유형 | claim_type | text | - | 주문취소/반품/교환 |
| 4 | 하위 유형 | claim_sub_type | text | - | 주문취소(사용자)/판매취소(관리자) 등 |
| 5 | FO 상태 | claim_status_fo | badge | - | FO 노출 상태 |
| 6 | BO 상태 | claim_status_bo | badge | - | BO 세부 상태 |
| 7 | 처리 채널 | process_channel | text | - | FO/BO_PROXY/SYSTEM |
| 8 | 접수 채널 | applied_channel | text | - | FO/BO_PROXY/ADMIN |
| 9 | 주문번호 | order_id | link | - | 클릭 시 주문상세 새 탭(주문관리 모듈) |
| 10 | 출고지 | shipment_name | text | - | 출고지명 |
| 11 | 고객사 | client_name | text | - | 멀티태넌트 |
| 12 | 구매자 | buyer_id | link | - | 구매자ID(이름). 클릭 시 회원 상세 팝업 |
| 13 | 판매사 | vendor_id | link | - | 판매사코드(판매사명). 클릭 시 판매사 정보 팝업 |
| 14 | 대표 상품명 | representative_product_name | text | - | 클레임 아이템 중 첫 번째 상품명(+N 표시) |
| 15 | 대상 수량 | total_item_quantity | number | - | ClaimItem 합계 |
| 16 | 귀책 | liability_party | badge | - | 구매자(gray)/판매사(red) |
| 17 | 사유 | reason_code | text | - | ClaimReasonCode 라벨 |
| 18 | 환불 총액 | refund_total_amount | number | - | 금액(원) |
| 19 | PG 환불액 | refund_pg_amount | number | - | 금액(원) |
| 20 | 포인트 복원 | refund_point_amount | number | - | 금액(원) |
| 21 | 클레임배송비 | shipping_fee_charged | number | - | 금액(원) |
| 22 | **수거 상태** | (계산) | badge | - | 반품/교환 전용. 반송장 등록/수령/회수 단계 표시. 주문취소는 "—" |
| 23 | **반송장** | pickup_tracking_number | text | - | 택배사(송장번호). 반품/교환만 |
| 24 | **검수 결과** | inspection_result | badge | - | PASSED/FAILED_*/미검수/해당없음(주문취소) |
| 25 | **교환주문번호** | exchange_order_id | link | - | 교환 최종승인 시 표시. 클릭 → 주문상세 |
| 26 | 신청일시 | applied_at | date | desc | YYYY.MM.DD HH:MM:SS, 기본 정렬 |
| 27 | 상태변경일 | status_changed_at | date | - | 최근 상태 전이 시각 |
| 28 | 완료일 | completed_at | date | - | 환불완료·교환최종승인·철회 시점. 미완료 시 "—" |
| 29 | 승인 필요 | approval_required | badge | - | Yes/No |

---

## 상태 badge 정의

### BO 클레임 상태 (`claim_status_bo`)

#### 주문취소 계열

| 값 | 라벨 | 색상 |
|----|------|------|
| CANCEL_REQUESTED | 주문취소 신청 | blue |
| SALE_CANCEL_REQUESTED | 판매취소 신청 | indigo |
| REFUND_REQUESTED | 환불요청 완료 | purple |
| REFUNDED | 환불완료 | green |
| REFUND_FAILED | 환불실패 | red |
| CANCEL_WITHDRAWN | 주문취소 철회 | orange |

#### 반품 계열

| 값 | 라벨 | 색상 |
|----|------|------|
| RETURN_REQUESTED | 반품신청 | blue |
| RETURN_RECEIVED | 반품접수 | teal |
| RETURN_PICKUP_IN_PROGRESS | 반품수거중 | violet |
| RETURN_PICKED_UP | 반품상품수거 | purple |
| RETURN_COLLECTED | 반품회수완료 | amber |
| RETURN_WITHDRAWN | 반품철회 | orange |

#### 교환 계열

| 값 | 라벨 | 색상 |
|----|------|------|
| EXCHANGE_REQUESTED | 교환신청 | blue |
| EXCHANGE_RECEIVED | 교환접수 | teal |
| EXCHANGE_PICKUP_IN_PROGRESS | 교환수거중 | violet |
| EXCHANGE_PICKED_UP | 교환상품수거 | purple |
| EXCHANGE_COLLECTED | 교환회수완료 | amber |
| EXCHANGE_FINAL_APPROVED | 교환최종승인 | green |
| EXCHANGE_WITHDRAWN | 교환철회 | orange |

### 검수 결과 (`inspection_result`)

| 값 | 라벨 | 색상 |
|----|------|------|
| PASSED | 통과 | green |
| FAILED_DEFECT | 훼손 | red |
| FAILED_MISSING | 누락 | red |
| FAILED_OTHER | 기타 거부 | red |
| (NULL) | 미검수 | gray |

### 귀책 (`liability_party`)

| 값 | 라벨 | 색상 |
|----|------|------|
| BUYER | 구매자 | gray |
| SELLER | 판매사 | red |

---

## 레이아웃

```
┌────────────────────────────────────────────────────────┐
│ 페이지 타이틀: 배송상품 클레임 목록                     │
├────────────────────────────────────────────────────────┤
│ 검색 필터 영역 (테이블형 2열, 15개 항목)                │
│ 기간 | 클레임유형(NEW) | 하위유형 | FO상태 | BO상태       │
│ 처리채널 | 접수채널 | 귀책 | 사유 | 판매사               │
│ 고객사 | 구매자 | 환불수단 | 승인필요 | 검색어            │
│                            [초기화] [검색]               │
├────────────────────────────────────────────────────────┤
│ 툴바: 조회결과 N건 | [엑셀 다운로드]                    │
├────────────────────────────────────────────────────────┤
│ 클레임 목록 테이블 (29컬럼, 가로 스크롤)                 │
│ 기본 정렬: 신청일시 DESC / 페이지당 20·50·100건          │
├────────────────────────────────────────────────────────┤
│ 페이지네이션                                             │
└────────────────────────────────────────────────────────┘
```

> **v2.0 삭제:** 기존 "[주문취소] [반품(준비중)] [교환(준비중)]" 탭 UI 전면 제거. 유형 필터링은 검색 필터 #2 `클레임 유형` 체크박스로 수행.

---

## 인터랙션

### (1) 검색

- 검색어 select에서 `사용안함` 이외 선택 시 1~14 필터 비활성(배경 회색)한다.
- 검색어에 콤마(`,`)로 복수 값을 입력하면 OR 조건으로 검색한다(상품명·클레임코드·주문번호 한정).
- 검색 실행 시 [BO-CLM-030 클레임 목록 조회](../04-actions/BO-CLM-030_클레임-목록-조회.md) 액션 호출(`claim_type` 필터값 전송).

### (2) 행 선택/클릭 — 유형별 상세 분기

행의 `claim_type`에 따라 상세 팝업이 분기된다.

| claim_type | 상세 팝업 |
|------------|-----------|
| `CANCEL` | [BO-CLM-042 주문취소 상세 화면](../05-wireframes/BO-CLM-042_주문취소-상세-화면.md) |
| `RETURN` | [BO-CLM-044 반품 상세 화면](../05-wireframes/BO-CLM-044_반품-상세-화면.md) |
| `EXCHANGE` | [BO-CLM-046 교환 상세 화면](../05-wireframes/BO-CLM-046_교환-상세-화면.md) |

- **클레임코드 셀 클릭:** `claim_type`별 분기 팝업 오픈.
- **주문번호 셀 클릭:** 주문관리 모듈 주문상세 새 탭.
- **교환주문번호 셀 클릭:** 교환주문 상세 새 탭(EXCHANGE_FINAL_APPROVED 이후).
- **판매사·구매자 셀 클릭:** 각각 정보 팝업.

### (3) 엑셀 다운로드

- 현재 필터 조건 기준 조회 결과를 엑셀(xlsx)로 다운로드. 최대 10,000건(초과 시 토스트).
- 컬럼 구성은 화면 테이블과 동일하되 badge는 텍스트로 변환.

### (4) 페이지네이션

- 페이지당 건수: 20(기본) / 50 / 100.
- 페이지 이동 시 URL 쿼리(`?page=2`)에 반영.

---

## 상태 처리

| 상태 | 표시 |
|------|------|
| 로딩 | 테이블 영역 상단에 진행 바, 기존 데이터 dimmed |
| 데이터 없음 | "조회 결과가 없습니다. 필터를 조정해 주세요." 중앙 표시 |
| 필터 충돌 | 검색어 select = `사용안함` 해제 경고 토스트(에러) |
| 조회 실패 | "목록을 불러오는 데 실패했습니다. 잠시 후 다시 시도해 주세요." + 재시도 버튼 |
| 권한 부족(403) | 상단 알림 배너: "클레임 관리 권한이 없습니다." |

---

## 관련 문서

- [BO-CLM-001](../01-policies/BO-CLM-001_클레임-관리-공통-정책.md) — 공통 정책
- [BO-CLM-002](../01-policies/BO-CLM-002_주문취소-정책.md) — 주문취소 정책
- [BO-CLM-003](../01-policies/BO-CLM-003_반품-정책.md) — 반품 정책
- [BO-CLM-004](../01-policies/BO-CLM-004_교환-정책.md) — 교환 정책
- [BO-CLM-010](../02-data/BO-CLM-010_클레임-엔티티.md) — 클레임 엔티티
- [BO-CLM-030](../04-actions/BO-CLM-030_클레임-목록-조회.md) — 클레임 목록 조회 액션
- [BO-CLM-040](../05-wireframes/BO-CLM-040_배송상품-클레임-목록-화면.md) — 배송상품 클레임 목록 화면(통합)
- 유형별 전용 화면: [BO-CLM-041](../05-wireframes/BO-CLM-041_주문취소-목록-화면.md) / [BO-CLM-043](../05-wireframes/BO-CLM-043_반품-목록-화면.md) / [BO-CLM-045](../05-wireframes/BO-CLM-045_교환-목록-화면.md)
