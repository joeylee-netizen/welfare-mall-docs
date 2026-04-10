# Wireframe 문서 스키마

> 와이어프레임(Wireframe) 문서의 Frontmatter 및 본문 구조 정의

## Frontmatter

### 필수 필드 (공통)

| 필드 | 타입 | 규칙 |
|------|------|------|
| `id` | string | 패턴: `{BO\|FO}-{DOMAIN}-{SEQ}` (예: `BO-PDM-040`). 파일명 접두사와 일치 |
| `title` | string | 비어 있지 않을 것 |
| `type` | enum | `wireframe` 고정 |
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

### 확장 필드 (Wireframe 전용)

| 필드 | 타입 | 설명 |
|------|------|------|
| `route` | string | 화면 URL 경로 (예: `/bo/products`) |
| `layout` | enum | `bo-standard` \| `bo-full` \| `fo-app` \| `fo-web` |
| `mockup_type` | enum | `html` \| `image` \| `figma` |
| `mockup_path` | string | 목업 파일 상대 경로 |

## Frontmatter 예시

```yaml
---
id: "BO-PDM-040"
title: "상품 목록 화면"
type: wireframe
domain: product
status: draft
version: "1.0"
created: 2026-04-10
updated: 2026-04-10
author: "기획자"
refs:
  - "BO-PDM-020"
  - "BO-PDM-030"
tags: [상품, 목록, 화면]
route: "/bo/products"
layout: bo-standard
mockup_type: html
mockup_path: "./BO-PDM-040_상품-목록-화면.html"
---
```

## 권장 본문 구조

```markdown
# {title}

## 수정 이력

| 버전 | 날짜 | 작성자 | 변경 내용 |
|------|------|--------|----------|
| v0.0.1 | 20260410 | 기획자 | 초안 작성 |

## 화면 개요
화면의 목적 및 주요 기능 요약.

## 레이아웃
화면 영역 구분 및 배치 설명.

## 구성 요소
### 검색/필터 영역
- 컴포넌트 참조: [BO-PDM-020](../03-components/BO-PDM-020_상품-목록-테이블.md)

### 목록 영역
- 테이블 컬럼 및 동작 설명

### 버튼 영역
- 액션 참조: [BO-PDM-030](../04-actions/BO-PDM-030_상품-등록-액션.md)

## 기능 명세
상세 인터랙션, 조건부 표시, 상태 전이 등.
```

## HTML 와이어프레임 규칙

- `_wireframe/base.css`를 import하여 공통 스타일 사용
- `data-component="{ID}"` 속성으로 컴포넌트 매핑
- `data-action="{ID}"` 속성으로 액션 매핑
- `data-region` 속성으로 레이아웃 영역 표시
- 인라인 JS는 최대 50줄 이내 (탭 전환, 모달 토글 등)
- 외부 라이브러리, fetch 호출 금지
