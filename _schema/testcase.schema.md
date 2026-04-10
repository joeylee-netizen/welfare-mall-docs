# Test Case 문서 스키마

> 테스트케이스(TC) 문서의 Frontmatter 및 본문 구조 정의
> MD 파일과 Excel 파일을 병행 관리한다.

## Frontmatter

### 필수 필드 (공통)

| 필드 | 타입 | 규칙 |
|------|------|------|
| `id` | string | 패턴: `{BO\|FO}-{DOMAIN}-{SEQ}` (예: `BO-PDM-050`). 파일명 접두사와 일치 |
| `title` | string | 비어 있지 않을 것 |
| `type` | enum | `testcase` 고정 |
| `domain` | string | 상위 폴더명과 일치 (product, claim, common) |
| `status` | enum | `draft` \| `review` \| `approved` \| `deprecated` |
| `version` | string | 시맨틱 버전 (예: `1.0`) |
| `created` | date | `YYYY-MM-DD` |
| `updated` | date | `YYYY-MM-DD`, `created` 이상 |
| `author` | string | 비어 있지 않을 것 |

### 선택 필드 (공통)

| 필드 | 타입 | 규칙 |
|------|------|------|
| `refs` | string[] | 유효한 ID 형식; 대상 파일 존재 |
| `tags` | string[] | 자유 형식 |

### 확장 필드 (Test Case 전용)

| 필드 | 타입 | 설명 |
|------|------|------|
| `test_target` | string | 테스트 대상 문서 ID (예: `BO-PDM-030`) |
| `test_type` | enum | `functional` \| `ui` \| `api` \| `e2e` \| `regression` |
| `excel_path` | string | Excel TC 파일 상대 경로 |

## Frontmatter 예시

```yaml
---
id: "BO-PDM-050"
title: "상품 등록 테스트케이스"
type: testcase
domain: product
status: draft
version: "1.0"
created: 2026-04-10
updated: 2026-04-10
author: "기획자"
refs:
  - "BO-PDM-030"
  - "BO-PDM-040"
tags: [상품, 등록, TC]
test_target: "BO-PDM-030"
test_type: functional
excel_path: "./BO-PDM-050_상품-등록-TC.xlsx"
---
```

## 권장 본문 구조 (MD)

```markdown
# {title}

## 테스트 개요
테스트 목적, 범위, 전제 조건.

## 테스트 케이스

### TC-001: {테스트명}

| 항목 | 내용 |
|------|------|
| 구분 | 정상 / 예외 |
| 사전 조건 | 로그인 상태, 권한 보유 |
| 테스트 절차 | 1. 상품 등록 버튼 클릭 → 2. 필수 항목 입력 → 3. 저장 |
| 기대 결과 | 상품이 목록에 추가됨 |
| 우선순위 | 상 / 중 / 하 |

### TC-002: {테스트명}
...

## Excel 파일 안내
상세 TC는 Excel 파일 참조: `{excel_path}`
```

## Excel 형식 규칙

| 컬럼 | 설명 |
|------|------|
| TC ID | TC-001 형식 |
| 구분 | 정상 / 예외 / 경계값 |
| 테스트명 | 테스트 제목 |
| 사전 조건 | 테스트 전 필요 상태 |
| 테스트 절차 | 단계별 상세 절차 |
| 기대 결과 | 예상 결과 |
| 실제 결과 | QA 실행 후 기록 |
| 결과 | Pass / Fail / Block |
| 우선순위 | 상 / 중 / 하 |
| 비고 | 추가 메모 |
