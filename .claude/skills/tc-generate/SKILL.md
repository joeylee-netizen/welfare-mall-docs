# /tc-generate — 테스트케이스 자동 생성

## 설명
기획 문서(Policy, Action, Wireframe)를 분석하여 테스트케이스 MD 파일을 자동 생성한다.

## 사용법
```
/tc-generate BO-PDM-030            # 특정 Action 기반 TC 생성
/tc-generate domains/BO/product    # 도메인 전체 TC 생성
/tc-generate BO-CLM-001 --type api # 특정 타입 TC만 생성
```

## 생성 절차

### 1. 문서 수집
- 대상 도메인의 모든 기획 문서 읽기
- Policy → 비즈니스 규칙 추출
- Action → API 스펙, 유효성 검사 규칙 추출
- Wireframe → 화면 인터랙션 추출

### 2. TC 도출 규칙

#### Policy 기반
| 소스 | TC 도출 |
|------|---------|
| 각 규칙(rule) | 정상 1건 + 예외 1건 이상 |
| `conditions` 필드 | 조건 충족/미충족 각 1건 |
| 수량/금액 규칙 | 경계값 3건 (최소, 최대, 초과) |

#### Action 기반
| 소스 | TC 도출 |
|------|---------|
| endpoint + method | 성공 호출 1건 |
| 필수 파라미터 | 각 파라미터 누락 시 에러 1건 |
| 유효성 검사 | 규칙별 실패 1건 |
| 응답 코드 | 주요 에러 코드별 1건 |

#### Wireframe 기반
| 소스 | TC 도출 |
|------|---------|
| 검색/필터 | 검색 정상 + 결과 없음 각 1건 |
| 페이지네이션 | 첫/마지막 페이지 이동 1건 |
| 버튼 클릭 | 활성/비활성 상태 각 1건 |
| empty state | 데이터 0건 표시 1건 |

### 3. TC ID 자동 부여
- 도메인 내 기존 TC 문서의 최대 SEQ 확인
- 다음 번호부터 순차 부여
- ID 대역 가이드: 050~059 (ARCHITECTURE.md 참조)

### 4. 출력 파일

#### MD 파일 구조
```yaml
---
id: "BO-{DOMAIN}-{SEQ}"
title: "{대상} 테스트케이스"
type: testcase
domain: {domain}
status: draft
version: "1.0"
created: {today}
updated: {today}
author: "QA Engineer"
refs:
  - "{source_id_1}"
  - "{source_id_2}"
test_target: "{primary_source_id}"
test_type: {functional|ui|api}
excel_path: "./{ID}_{title-slug}.xlsx"
---

# {title}

## 테스트 개요
- 대상: [{source_id}](상대경로)
- 범위: {테스트 범위 설명}

## 테스트 케이스

### TC-001: {테스트명}

| 항목 | 내용 |
|------|------|
| 구분 | 정상 |
| 사전 조건 | {조건} |
| 테스트 절차 | 1. → 2. → 3. |
| 기대 결과 | {결과} |
| 우선순위 | 상 |

...
```

#### 저장 경로
```
domains/{BO|FO}/{domain}/06-testcases/{ID}_{title-slug}.md
```

### 5. 커버리지 리포트

생성 완료 후 커버리지 요약을 출력한다.

```
📋 TC 생성 완료: {domain}

생성 파일:
  📄 BO-PDM-050 — 상품 등록 TC (12건)
  📄 BO-PDM-051 — 상품 수정 TC (8건)

커버리지:
  Policy  ██████████░░ 80% (4/5 규칙)
  Action  ████████████ 100% (3/3 엔드포인트)
  화면    ████████░░░░ 67% (2/3 화면)

미커버:
  ⚠️ BO-PDM-001 규칙 3: "포인트 한도 초과 시 처리" — TC 없음
```
