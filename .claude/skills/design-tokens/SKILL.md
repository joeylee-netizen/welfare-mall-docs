# /design-tokens — 디자인 토큰 카탈로그

## 설명
프로젝트의 디자인 토큰(색상, 간격, 타이포그래피, 컴포넌트 클래스)을 조회한다.
`_wireframe/base.css`를 기준으로 사용 가능한 토큰 목록을 제공한다.

## 사용법
```
/design-tokens                     # 전체 토큰 카탈로그 출력
/design-tokens color               # 색상 토큰만 조회
/design-tokens component           # 컴포넌트 클래스 조회
/design-tokens badge               # 뱃지 상태 클래스 조회
```

## 카탈로그 범위

토큰 소스: `_wireframe/base.css`

### 1. 색상

#### 브랜드
| 용도 | 값 | 사용 클래스 |
|------|---|------------|
| 주요 액션 | `#2563eb` | `.wf-btn--primary` |
| 위험 액션 | `#dc2626` | `.wf-btn--danger` |
| 링크 | `#4075f3` | `a` 기본 |
| 컴포넌트 라벨 | `#4075f3` | `[data-component]::before` |
| 액션 라벨 | `#f95729` | `[data-action]::after` |

#### 뉴트럴
| 용도 | 값 | 사용 클래스 |
|------|---|------------|
| 본문 텍스트 | `#333` | body 기본 |
| 보조 텍스트 | `#999` | `.wf-text-muted` |
| 테이블 헤더 배경 | `#f7f8fa` | `.wf-table th` |
| 페이지 배경 | `#f5f5f5` | body 기본 |
| 보더 | `#ddd`, `#e5e5e5`, `#eee` | 각종 border |

### 2. 뱃지 상태

| 상태 | 클래스 | 배경 | 텍스트 |
|------|--------|------|--------|
| 준비 중 | `.wf-badge--preparing` | `#fff3cd` | `#856404` |
| 준비 완료 | `.wf-badge--ready` | `#d1ecf1` | `#0c5460` |
| 완료 | `.wf-badge--completed` | `#d4edda` | `#155724` |
| 취소 | `.wf-badge--cancelled` | `#f8d7da` | `#721c24` |

### 3. 컴포넌트 클래스

| 컴포넌트 | 클래스 | 변형 |
|---------|--------|------|
| 버튼 | `.wf-btn` | `--primary`, `--danger`, `--text`, `--sm`, `--lg` |
| 입력 | `.wf-input` | - |
| 셀렉트 | `.wf-select` | - |
| 테이블 | `.wf-table` | - |
| 카드 | `.wf-card` | - |
| 뱃지 | `.wf-badge` | `--preparing`, `--ready`, `--completed`, `--cancelled` |
| 페이지네이션 | `.wf-pagination` | `.wf-pagination-btn`, `.active` |
| 필터 행 | `.wf-filter-row` | - |
| 탭 | `.wf-tabs`, `.wf-tab` | `.active` |
| 플레이스홀더 | `.wf-placeholder` | - |

### 4. 레이아웃 클래스

| 레이아웃 | 클래스 | 용도 |
|---------|--------|------|
| BO 표준 | `.wf-layout` | GNB + LNB + Content |
| BO GNB | `.wf-gnb` | 상단 네비게이션 |
| BO LNB | `.wf-lnb` | 좌측 사이드바 |
| BO 콘텐츠 | `.wf-content` | 메인 영역 |
| FO 앱 | `.wf-app-layout` | 모바일 레이아웃 |
| Spec 레이아웃 | `.spec-layout` | 와이어프레임 + 설명 패널 |
| 설명 패널 | `.spec-desc-panel` | 기능 명세 패널 |

### 5. 유틸리티 클래스

| 클래스 | 용도 |
|--------|------|
| `.wf-flex` | flex + align-center |
| `.wf-flex-col` | flex column |
| `.wf-flex-between` | flex + space-between |
| `.wf-gap` | gap 8px |
| `.wf-gap-lg` | gap 16px |
| `.wf-mt` | margin-top 16px |
| `.wf-mb` | margin-bottom 16px |
| `.wf-ml-auto` | margin-left auto |
| `.wf-text-center` | 중앙 정렬 |
| `.wf-text-right` | 우측 정렬 |
| `.wf-text-muted` | 보조 텍스트 (#999) |
| `.wf-text-sm` | 작은 글씨 (12px) |
| `.wf-divider` | 구분선 |
| `.wf-sr-only` | 스크린리더 전용 |

## 실행 절차

1. `_wireframe/base.css` 파싱
2. 요청된 카테고리 필터링
3. 카탈로그 테이블 형식으로 출력
