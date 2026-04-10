# Component 문서 스키마

> 컴포넌트(Component) 문서의 Frontmatter 및 본문 구조 정의

## Frontmatter

### 필수 필드 (공통)

| 필드 | 타입 | 규칙 |
|------|------|------|
| `id` | string | 패턴: `{BO\|FO}-{DOMAIN}-{SEQ}` (예: `BO-PDM-020`). 파일명 접두사와 일치 |
| `title` | string | 비어 있지 않을 것 |
| `type` | enum | `component` 고정 |
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

### 확장 필드 (Component 전용)

| 필드 | 타입 | 설명 |
|------|------|------|
| `ui_type` | enum | `table` \| `form` \| `modal` \| `card` \| `filter` \| `list` \| `detail` \| `dashboard` |
| `design_tokens` | object | 컴포넌트에 적용되는 디자인 토큰 (색상, 간격 등) |

## Frontmatter 예시

```yaml
---
id: "BO-PDM-020"
title: "상품 목록 테이블"
type: component
domain: product
status: draft
version: "1.0"
created: 2026-04-10
updated: 2026-04-10
author: "기획자"
refs:
  - "BO-PDM-010"
tags: [상품, 목록, 테이블]
ui_type: table
design_tokens:
  header_bg: "#f7f8fa"
  row_hover: "#f0f4ff"
---
```

## 권장 본문 구조

```markdown
# {title}

## 화면 구성
컴포넌트의 레이아웃 및 구성 요소 설명.

## 컬럼 정의 (테이블인 경우)

| No | 컬럼명 | 데이터 필드 | 타입 | 정렬 | 비고 |
|----|--------|------------|------|------|------|
| 1 | 상품코드 | product_code | text | - | 링크 → 상세 |

## 상태 정의
상태별 표시 방식 (badge, 색상 등).

## 인터랙션
사용자 조작에 대한 동작 정의.
```
