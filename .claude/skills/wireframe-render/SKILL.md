# /wireframe-render — 와이어프레임 HTML 렌더링

## 설명
Wireframe MD 문서와 참조된 Component/Action 문서를 읽어 HTML 와이어프레임을 생성한다.
`_wireframe/template.html`과 `_wireframe/base.css`를 기반으로 표준화된 화면을 출력한다.

## 사용법
```
/wireframe-render BO-PDM-040       # 특정 와이어프레임 렌더링
/wireframe-render domains/BO/product/05-wireframes/BO-PDM-040_상품-목록-화면.md
```

## 렌더링 절차

### 1. 입력 수집
- 대상 Wireframe MD 파일 읽기
- frontmatter에서 `refs` 추출
- 참조된 Component 문서 → 컬럼 정의, UI 타입 추출
- 참조된 Action 문서 → 트리거, 엔드포인트 추출
- `_wireframe/template.html` 읽기

### 2. HTML 생성

#### meta 태그
```html
<meta name="wfr-id"      content="{id}">
<meta name="wfr-title"   content="{title}">
<meta name="wfr-domain"  content="{domain}">
<meta name="wfr-route"   content="{route}">
<meta name="wfr-version" content="{version}">
```

#### CSS import
```html
<link rel="stylesheet" href="../../_wireframe/base.css">
```

#### 컴포넌트 → HTML 변환
| ui_type | HTML 구조 |
|---------|----------|
| table | `.wf-table` + thead/tbody |
| form | `.wf-input`, `.wf-select` 그룹 |
| filter | `.wf-filter-row` |
| card | `.wf-card` |
| modal | `<dialog>` |
| list | 반복 `.wf-card` |
| detail | 라벨+값 레이아웃 |

#### 액션 → HTML 변환
```html
<button class="wf-btn wf-btn--primary"
        data-action="{action_id}"
        data-trigger="{trigger}">
  {action_title}
</button>
```

### 3. 레이아웃 선택
- `layout: bo-standard` → `.wf-layout` (GNB + LNB + Content)
- `layout: bo-full` → `.wf-layout` (LNB 없이 전체 너비)
- `layout: fo-app` → `.wf-app-layout` (모바일)
- `layout: fo-web` → `.wf-app-layout` (웹 너비)

### 4. 출력
- 파일: `{같은 폴더}/{ID}_{title-slug}.html`
- Wireframe MD의 `mockup_path` 필드 갱신

## 제약 사항
- 인라인 JS 최대 50줄
- 외부 라이브러리 금지
- fetch/API 호출 금지
- 허용 JS: 탭 전환, 모달 토글, LNB active, 토글 열기/닫기
