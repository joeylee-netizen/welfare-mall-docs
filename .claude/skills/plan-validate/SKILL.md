---
name: plan-validate
description: 기획 문서 유효성 검증 — frontmatter, ID 규칙, 참조 무결성 검사
---

# /plan-validate — 기획 문서 유효성 검증

## 설명
기획 문서의 frontmatter, ID 규칙, 참조 무결성을 검증한다.

## 사용법
```
/plan-validate                     # 전체 문서 검증
/plan-validate BO-PDM-001          # 특정 문서 검증
/plan-validate domains/BO/product  # 특정 도메인 검증
```

## 검증 항목

### 1. Frontmatter 검증
- [ ] 필수 필드 존재: id, title, type, domain, status, version, created, updated, author
- [ ] `id` 패턴: `{BO|FO}-{DOMAIN}-{SEQ}` 형식
- [ ] `type` 값이 폴더 위치와 일치 (policy → 01-policies/)
- [ ] `domain` 값이 상위 폴더명과 일치
- [ ] `status` 값이 유효 (draft/review/approved/deprecated)
- [ ] `updated` ≥ `created`
- [ ] 타입별 확장 필드 검증 (`_schema/` 참조)

### 2. 파일명 검증
- [ ] 파일명이 `{ID}_{title-slug}.md` 형식
- [ ] 파일명 접두사와 frontmatter `id` 일치

### 3. ID 검증
- [ ] 도메인 약어가 유효 (PDM, CLM, CMN)
- [ ] 동일 도메인 내 SEQ 중복 없음
- [ ] BO/FO 접두사가 폴더 위치와 일치

### 4. 참조 무결성
- [ ] `refs` 배열의 모든 ID에 대응하는 파일 존재
- [ ] 본문 `[ID](path)` 링크의 경로가 유효
- [ ] 순환 참조 없음

## 실행 절차

1. 대상 범위의 모든 `.md` 파일을 Glob으로 수집
2. 각 파일의 frontmatter 파싱
3. `_schema/` 스키마와 대조 검증
4. 참조 대상 파일 존재 여부 확인
5. 결과 보고서 출력

## 출력 형식

```
✅ PASS  BO-PDM-001 — 상품 등록 정책
✅ PASS  BO-PDM-010 — 상품 엔티티
❌ FAIL  BO-CLM-001 — refs에 존재하지 않는 ID: BO-CLM-010
⚠️ WARN  BO-PDM-020 — 선택 필드 ui_type 누락

---
결과: 전체 N건 | 통과 N | 실패 N | 경고 N
```
