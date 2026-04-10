# Data 문서 스키마

> 데이터(Data) 문서의 Frontmatter 및 본문 구조 정의

## Frontmatter

### 필수 필드 (공통)

| 필드 | 타입 | 규칙 |
|------|------|------|
| `id` | string | 패턴: `{BO\|FO}-{DOMAIN}-{SEQ}` (예: `BO-PDM-010`). 파일명 접두사와 일치 |
| `title` | string | 비어 있지 않을 것 |
| `type` | enum | `data` 고정 |
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

### 확장 필드 (Data 전용)

| 필드 | 타입 | 설명 |
|------|------|------|
| `entity_name` | string | 영문 엔티티명 (PascalCase, 예: `Product`) |
| `source_system` | string | 데이터 출처 시스템명 |

## Frontmatter 예시

```yaml
---
id: "BO-PDM-010"
title: "상품 엔티티"
type: data
domain: product
status: draft
version: "1.0"
created: 2026-04-10
updated: 2026-04-10
author: "기획자"
refs:
  - "BO-PDM-001"
tags: [상품, 엔티티]
entity_name: "Product"
source_system: "복지몰 DB"
---
```

## 권장 본문 구조

```markdown
# {title}

## 필드 정의

| 필드명 | 타입 | 필수 | 설명 | 비고 |
|--------|------|------|------|------|
| product_id | BIGINT | Y | 상품 고유 ID | PK, Auto Increment |
| product_name | VARCHAR(200) | Y | 상품명 | |

## 관계
엔티티 간 관계 정의 (1:N, N:M 등). 관련 엔티티는 `[ID](path)`로 참조.

## 인덱스 (선택)
주요 조회 패턴에 대한 인덱스 설계.

## 제약 조건 (선택)
유니크 제약, 체크 제약 등.
```
