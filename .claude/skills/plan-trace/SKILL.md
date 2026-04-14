---
name: plan-trace
description: 문서 ID 참조 추적 — 영향 범위 시각화 및 영향도 분석
---

# /plan-trace — ID 참조 추적

## 설명
특정 문서 ID를 참조하는 모든 문서를 추적하여 영향 범위를 시각화한다.
문서 수정/삭제 전 영향도 분석에 활용한다.

## 사용법
```
/plan-trace BO-PDM-001             # 이 ID를 참조하는 문서 추적
/plan-trace BO-PDM-001 --depth 2   # 2단계 깊이까지 추적
/plan-trace BO-PDM-001 --reverse   # 이 ID가 참조하는 문서 추적
```

## 추적 방식

### 순방향 추적 (기본)
"이 문서를 **누가** 참조하는가?"
- `refs` 배열에 해당 ID가 포함된 문서 검색
- 본문에서 `[{ID}]` 또는 `{ID}` 패턴이 있는 문서 검색

### 역방향 추적 (--reverse)
"이 문서가 **무엇을** 참조하는가?"
- 해당 문서의 `refs` 배열 나열
- 본문의 Markdown 링크에서 참조 ID 추출

## 실행 절차

1. 대상 ID의 파일 위치 확인
2. 전체 `.md` 파일에서 해당 ID를 Grep 검색
3. frontmatter `refs` + 본문 링크 구분
4. 깊이(depth)별 재귀 추적
5. 트리 형태로 출력

## 출력 형식

```
📄 BO-PDM-001 (상품 등록 정책)
├── 📄 BO-PDM-010 (상품 엔티티) — refs
│   ├── 📄 BO-PDM-020 (상품 목록 테이블) — refs
│   └── 📄 BO-PDM-030 (상품 등록 액션) — refs
├── 📄 BO-PDM-030 (상품 등록 액션) — refs
│   └── 📄 BO-PDM-040 (상품 목록 화면) — refs
└── 📄 BO-PDM-050 (상품 등록 TC) — body link

---
총 참조 문서: N건 (직접 N / 간접 N)
```
