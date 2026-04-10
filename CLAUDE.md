# CLAUDE.md — 임직원 복지몰 기획 프로젝트

> 이 문서는 AI 에이전트가 프로젝트 작업 시 따라야 할 규칙을 정의한다.

---

## 프로젝트 개요

- **프로젝트명:** welfare-mall-planning (임직원 복지몰 기획)
- **목적:** 임직원 복지몰 서비스의 기획 문서 관리
- **현재 범위:** BO 관리자 시스템 (상품 관리 + 클레임 관리)
- **확장 예정:** BO 추가 도메인 → FO 추가

---

## 일반 규칙

1. **응답 언어:** 모든 텍스트는 한국어로 작성한다. 단, 코드/ID/기술 용어는 예외.
2. **구조 파일 우선 참조:** 작업 시작 전 `ARCHITECTURE.md`, `_schema/` 디렉토리를 먼저 확인한다.
3. **ID 형식:** `{BO|FO}-{DOMAIN}-{SEQ}` (예: `BO-PDM-001`).
4. **문서 수정 시:** 반드시 `updated` 날짜와 `version`을 갱신한다.

---

## 기획 문서 워크플로

### 기획 흐름 (6단계)

```
Policy → Data → Component → Action → Wireframe → Test Case
```

각 단계의 산출물은 이전 단계를 `refs`로 참조해야 한다.

### 문서 타입 (6가지)

| 타입 | 폴더 | 스키마 |
|------|------|--------|
| Policy (정책) | `01-policies/` | `_schema/policy.schema.md` |
| Data (데이터) | `02-data/` | `_schema/data.schema.md` |
| Component (컴포넌트) | `03-components/` | `_schema/component.schema.md` |
| Action (액션) | `04-actions/` | `_schema/action.schema.md` |
| Wireframe (와이어프레임) | `05-wireframes/` | `_schema/wireframe.schema.md` |
| Test Case (테스트케이스) | `06-testcases/` | `_schema/testcase.schema.md` |

---

## 문서 작성 규칙

### 파일명

```
{ID}_{title-slug}.md
```

- 예: `BO-PDM-001_상품-등록-정책.md`
- 와이어프레임 HTML: `BO-PDM-040_상품-목록-화면.html`
- 테스트케이스 Excel: `BO-PDM-050_상품-등록-TC.xlsx`

### Frontmatter

모든 기획 문서는 YAML Frontmatter 필수:

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
tags: []
# + 타입별 확장 필드 (스키마 참조)
---
```

**필수 필드:** id, title, type, domain, status, version, created, updated, author
**선택 필드:** refs, tags + 타입별 확장 필드

### 참조 방식

1. **구조적 참조** — Frontmatter `refs` 배열에 의존 문서 ID 나열
2. **맥락적 참조** — 본문에서 `[ID](상대경로)`로 Markdown 링크

```markdown
refs:
  - "BO-PDM-001"

본문에서: [BO-PDM-001](../01-policies/BO-PDM-001_상품-등록-정책.md)
```

---

## 도메인 약어

| 도메인 | 약어 | 폴더 |
|--------|------|------|
| 상품 관리 | `PDM` | `product` |
| 클레임 관리 | `CLM` | `claim` |
| 공통 | `CMN` | `common` |

---

## 와이어프레임 규칙

- `_wireframe/base.css`를 import하여 공통 스타일 사용
- `_wireframe/template.html`을 기반으로 BO 화면 작성
- HTML 속성으로 기획 문서 매핑:
  - `data-component="{ID}"` — 컴포넌트 문서 연결
  - `data-action="{ID}"` — 액션 문서 연결
  - `data-region` — 레이아웃 영역 표시
- 인라인 JS: 최대 50줄 (탭 전환, 모달 토글 등)
- 금지: 외부 라이브러리, fetch/API 호출
- 수정 이력: `v0.0.N` 형식, 날짜 `YYYYMMDD`, 최신이 위

---

## 테스트케이스 규칙

- MD 파일과 Excel 파일 병행 관리
- MD: 기획 문서 내 TC 개요 및 주요 케이스
- Excel: 상세 TC (실행 결과 기록 포함)
- `excel_path` 필드로 Excel 파일 참조

---

## Git 커밋 규칙

```
docs({scope}): {subject}
```

- `scope`: 도메인명 (product, claim) 또는 schema, wireframe
- `subject`: 변경 내용 요약 (한국어 가능)
- 예: `docs(product): add BO-PDM-001 상품 등록 정책`
- 예: `chore(schema): update testcase schema 필드 추가`
