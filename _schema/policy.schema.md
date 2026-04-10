# Policy 문서 스키마

> 정책(Policy) 문서의 Frontmatter 및 본문 구조 정의

## Frontmatter

### 필수 필드 (공통)

| 필드 | 타입 | 규칙 |
|------|------|------|
| `id` | string | 패턴: `{BO\|FO}-{DOMAIN}-{SEQ}` (예: `BO-PDM-001`). 파일명 접두사와 일치 |
| `title` | string | 비어 있지 않을 것 |
| `type` | enum | `policy` 고정 |
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

### 확장 필드 (Policy 전용)

| 필드 | 타입 | 설명 |
|------|------|------|
| `priority` | enum | `high` \| `medium` \| `low` — 정책 우선순위 |
| `effective_date` | date | 정책 시행일 (`YYYY-MM-DD`) |
| `conditions` | string[] | 정책 적용 조건 목록 |

## Frontmatter 예시

```yaml
---
id: "BO-PDM-001"
title: "상품 등록 정책"
type: policy
domain: product
status: draft
version: "1.0"
created: 2026-04-10
updated: 2026-04-10
author: "기획자"
refs: []
tags: [상품, 등록]
priority: high
effective_date: 2026-05-01
conditions:
  - "임직원 인증 완료"
  - "복지 포인트 잔액 확인"
---
```

## 권장 본문 구조

```markdown
# {title}

## 목적
정책의 목적 및 배경 설명.

## 정책 내용
### 규칙 1: {규칙명}
- 상세 내용

### 규칙 2: {규칙명}
- 상세 내용

## 적용 범위
대상 사용자, 시스템, 예외 사항.

## 관련 문서
- [BO-PDM-002](../02-data/BO-PDM-002_상품-엔티티.md) — 관련 데이터 정의
```
