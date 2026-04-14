---
name: publish-audit
description: 발행 와이어프레임 검증 — HTML이 기획 문서와 일치하는지 사후 검증
---

# /publish-audit — 발행 와이어프레임 검증

## 설명
발행된 HTML 와이어프레임이 기획 문서(Wireframe MD, Component, Action)와 일치하는지 검증한다.
publisher 에이전트가 생성한 결과물의 품질을 사후 검증하는 역할.

## 사용법
```
/publish-audit BO-PDM-040                         # 특정 화면 검증
/publish-audit domains/BO/product/05-wireframes/  # 도메인 전체 검증
```

## 검증 항목

### 1. 기획 문서 ↔ HTML 일치성

#### meta 태그 검증
- [ ] `wfr-id`가 Wireframe MD의 `id`와 일치
- [ ] `wfr-title`이 Wireframe MD의 `title`과 일치
- [ ] `wfr-domain`이 Wireframe MD의 `domain`과 일치
- [ ] `wfr-route`가 Wireframe MD의 `route`와 일치
- [ ] `wfr-version`이 Wireframe MD의 `version`과 일치

#### 컴포넌트 매핑 검증
- [ ] Wireframe MD `refs`의 모든 Component ID가 HTML에 `data-component`로 존재
- [ ] `data-component` 값이 실제 Component 문서의 `id`와 일치
- [ ] `data-title` 값이 Component 문서의 `title`과 일치

#### 액션 매핑 검증
- [ ] Wireframe MD `refs`의 모든 Action ID가 HTML에 `data-action`으로 존재
- [ ] `data-trigger` 값이 Action 문서의 `trigger`와 일치

#### 컬럼 동기화 (테이블 컴포넌트)
- [ ] Component 문서의 컬럼 정의와 HTML `<th>` 목록 일치
- [ ] 컬럼 순서 일치
- [ ] 컬럼 수 일치

### 2. 구조 검증

#### 레이아웃
- [ ] Wireframe MD의 `layout` 값에 맞는 CSS 클래스 사용
- [ ] `bo-standard` → `.wf-layout` (GNB + LNB + Content)
- [ ] `fo-app` → `.wf-app-layout`

#### CSS 참조
- [ ] `_wireframe/base.css` import 경로가 유효
- [ ] 인라인 스타일 최소화 (base.css 클래스 우선 사용)

#### JS 제약
- [ ] 인라인 JS 50줄 이내
- [ ] 외부 라이브러리 미사용
- [ ] fetch/API 호출 없음

### 3. Wireframe MD ↔ HTML 경로
- [ ] Wireframe MD의 `mockup_path`가 실제 HTML 파일 경로와 일치
- [ ] HTML 파일이 `05-wireframes/` 폴더에 존재

## 출력 형식

```
🔍 발행 검증: BO-PDM-040 (상품 목록 화면)

기획 ↔ HTML 일치성
  ✅ meta 태그 5/5 일치
  ❌ data-component: BO-PDM-021 — HTML에 누락
  ✅ data-action: 3/3 매핑 확인
  ⚠️ 컬럼 동기화: CMP 문서 8개 vs HTML <th> 7개 — "비고" 컬럼 누락

구조 검증
  ✅ 레이아웃: bo-standard → .wf-layout 정상
  ✅ CSS import 경로 유효
  ✅ 인라인 JS: 23줄 (50줄 이내)

---
결과: 통과 N / 실패 N / 경고 N
```
