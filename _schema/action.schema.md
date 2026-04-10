# Action 문서 스키마

> 액션(Action) 문서의 Frontmatter 및 본문 구조 정의

## Frontmatter

### 필수 필드 (공통)

| 필드 | 타입 | 규칙 |
|------|------|------|
| `id` | string | 패턴: `{BO\|FO}-{DOMAIN}-{SEQ}` (예: `BO-PDM-030`). 파일명 접두사와 일치 |
| `title` | string | 비어 있지 않을 것 |
| `type` | enum | `action` 고정 |
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

### 확장 필드 (Action 전용)

| 필드 | 타입 | 설명 |
|------|------|------|
| `trigger` | string | 액션 트리거 (예: `click`, `submit`, `change`, `load`) |
| `method` | enum | HTTP 메서드: `GET` \| `POST` \| `PUT` \| `PATCH` \| `DELETE` |
| `endpoint` | string | API 엔드포인트 경로 (예: `/api/v1/products`) |

## Frontmatter 예시

```yaml
---
id: "BO-PDM-030"
title: "상품 등록 액션"
type: action
domain: product
status: draft
version: "1.0"
created: 2026-04-10
updated: 2026-04-10
author: "기획자"
refs:
  - "BO-PDM-001"
  - "BO-PDM-020"
tags: [상품, 등록, API]
trigger: submit
method: POST
endpoint: "/api/v1/products"
---
```

## 권장 본문 구조

```markdown
# {title}

## 트리거 조건
액션이 실행되는 조건 (버튼 클릭, 폼 제출 등).

## 요청 (Request)

### Parameters / Body
| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| product_name | string | Y | 상품명 |

## 응답 (Response)

### 성공
| 필드명 | 타입 | 설명 |
|--------|------|------|
| product_id | number | 생성된 상품 ID |

### 실패
| 코드 | 메시지 | 설명 |
|------|--------|------|
| 400 | INVALID_PARAM | 필수 파라미터 누락 |

## 후속 처리
성공/실패 시 UI 동작, 알림, 리다이렉트 등.

## 유효성 검사
프론트엔드/백엔드 검증 규칙.
```
