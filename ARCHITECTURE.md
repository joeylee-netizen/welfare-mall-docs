# Architecture — 임직원 복지몰 기획 프로젝트

> 이 문서는 `welfare-mall-planning` 프로젝트의 구조, 규칙, 스키마를 정의한다.

---

## 1. 디렉토리 구조

```
welfare-mall-planning/
├── CLAUDE.md                  # 프로젝트 규칙 & AI 에이전트 가이드
├── ARCHITECTURE.md            # 이 문서
├── .gitignore
├── _schema/                   # 문서 타입별 스키마 정의
│   ├── policy.schema.md
│   ├── data.schema.md
│   ├── component.schema.md
│   ├── action.schema.md
│   ├── wireframe.schema.md
│   └── testcase.schema.md
├── _wireframe/                # 와이어프레임 공유 자산
│   ├── base.css               # 공통 CSS
│   └── template.html          # BO 레이아웃 템플릿
├── domains/                   # 도메인별 기획 문서
│   ├── BO/                    # Back Office (관리자)
│   │   ├── product/           # 상품 관리 (PDM)
│   │   │   ├── 01-policies/
│   │   │   ├── 02-data/
│   │   │   ├── 03-components/
│   │   │   ├── 04-actions/
│   │   │   ├── 05-wireframes/
│   │   │   └── 06-testcases/
│   │   └── claim/             # 클레임 관리 (CLM)
│   │       ├── 01-policies/
│   │       ├── 02-data/
│   │       ├── 03-components/
│   │       ├── 04-actions/
│   │       ├── 05-wireframes/
│   │       └── 06-testcases/
│   └── common/                # 공통 (CMN)
└── changelog/                 # 변경 이력
```

### 구조 설계 원칙

| 원칙 | 설명 |
|------|------|
| **도메인 분리** | `domains/BO/`, `domains/FO/`(예정) 하위에 도메인 폴더 — 독립적 확장 가능 |
| **6단계 폴더** | `01-policies/` ~ `06-testcases/` — 기획 흐름 순서대로 배치 |
| **언더스코어 접두사** | `_schema/`, `_wireframe/` — 메타 자산 시각적 구분, AI 파싱 우선 대상 |
| **changelog** | 분기별 설계 이력 — Git 로그보다 발견 가능성 높음 |

---

## 2. ID 네이밍 규칙

```
{FO|BO}-{DOMAIN}-{SEQ}
├─ 시스템: BO (Back Office) | FO (Front Office)
├─ DOMAIN: 3자리 대문자 약어
└─ SEQ: 3자리 제로패딩 (001 ~ 999)
```

### 예시

| ID | 의미 |
|----|------|
| `BO-PDM-001` | BO 상품 관리 정책 #1 |
| `BO-CLM-010` | BO 클레임 관리 데이터 #10 |
| `FO-PDM-040` | FO 상품 와이어프레임 #40 |

### ID 대역 가이드 (권장)

문서 타입별로 번호 대역을 나누어 관리한다 (강제 아님, 권장 사항).

| 타입 | 대역 | 예시 |
|------|------|------|
| Policy | 001 ~ 009 | BO-PDM-001 |
| Data | 010 ~ 019 | BO-PDM-010 |
| Component | 020 ~ 029 | BO-PDM-020 |
| Action | 030 ~ 039 | BO-PDM-030 |
| Wireframe | 040 ~ 049 | BO-PDM-040 |
| Test Case | 050 ~ 059 | BO-PDM-050 |

> 대역 초과 시 다음 대역 사용 (예: Policy 10개 이상이면 010부터 Data와 공유 관리).

---

## 3. 도메인 약어 표

| 도메인 | 약어 | 폴더명 | 설명 |
|--------|------|--------|------|
| 상품 관리 | `PDM` | `product` | 상품 등록, 수정, 카테고리, 재고 |
| 클레임 관리 | `CLM` | `claim` | 교환, 반품, 환불, 취소 |
| 공통 | `CMN` | `common` | 도메인 횡단 정책/컴포넌트 |

> 새 도메인 추가 시: 3자리 약어 등록 → 폴더 생성 → 이 표에 추가.

### 향후 확장 예정 도메인 (참고)

| 도메인 | 약어 | 비고 |
|--------|------|------|
| 주문 관리 | `ORD` | BO 추가 예정 |
| 회원 관리 | `MEM` | BO/FO 추가 예정 |
| 정산 관리 | `STL` | BO 추가 예정 |
| 포인트 관리 | `PNT` | BO/FO 추가 예정 |

---

## 4. Frontmatter 스키마

모든 기획 문서는 YAML Frontmatter를 포함해야 한다.

### 4.1 공통 필드

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `id` | string | Y | `{BO\|FO}-{DOMAIN}-{SEQ}` 형식 |
| `title` | string | Y | 문서 제목 (한국어) |
| `type` | enum | Y | `policy` \| `data` \| `component` \| `action` \| `wireframe` \| `testcase` |
| `domain` | string | Y | 폴더명과 일치 (product, claim, common) |
| `status` | enum | Y | `draft` \| `review` \| `approved` \| `deprecated` |
| `version` | string | Y | 시맨틱 버전 (예: `1.0`) |
| `created` | date | Y | `YYYY-MM-DD` |
| `updated` | date | Y | `YYYY-MM-DD` (≥ `created`) |
| `author` | string | Y | 작성자명 |
| `refs` | string[] | N | 참조 문서 ID 배열 |
| `tags` | string[] | N | 검색용 태그 |

### 4.2 타입별 확장 필드

#### Policy

| 필드 | 타입 | 설명 |
|------|------|------|
| `priority` | enum | `high` \| `medium` \| `low` |
| `effective_date` | date | 정책 시행일 |
| `conditions` | string[] | 적용 조건 목록 |

#### Data

| 필드 | 타입 | 설명 |
|------|------|------|
| `entity_name` | string | 영문 엔티티명 (PascalCase) |
| `source_system` | string | 데이터 출처 시스템 |

#### Component

| 필드 | 타입 | 설명 |
|------|------|------|
| `ui_type` | enum | `table` \| `form` \| `modal` \| `card` \| `filter` \| `list` \| `detail` \| `dashboard` |
| `design_tokens` | object | 디자인 토큰 (색상, 간격 등) |

#### Action

| 필드 | 타입 | 설명 |
|------|------|------|
| `trigger` | string | 트리거 이벤트 (click, submit 등) |
| `method` | enum | `GET` \| `POST` \| `PUT` \| `PATCH` \| `DELETE` |
| `endpoint` | string | API 엔드포인트 경로 |

#### Wireframe

| 필드 | 타입 | 설명 |
|------|------|------|
| `route` | string | 화면 URL 경로 |
| `layout` | enum | `bo-standard` \| `bo-full` \| `fo-app` \| `fo-web` |
| `mockup_type` | enum | `html` \| `image` \| `figma` |
| `mockup_path` | string | 목업 파일 상대 경로 |

#### Test Case

| 필드 | 타입 | 설명 |
|------|------|------|
| `test_target` | string | 테스트 대상 문서 ID |
| `test_type` | enum | `functional` \| `ui` \| `api` \| `e2e` \| `regression` |
| `excel_path` | string | Excel TC 파일 상대 경로 |

---

## 5. 문서 간 참조 전략

2가지 참조 방식을 병행한다.

### 5.1 구조적 참조 — Frontmatter `refs`

```yaml
refs:
  - "BO-PDM-001"
  - "BO-PDM-010"
```

- 머신 파싱 가능, 자동 검증 대상
- 이 문서가 **의존하는** 문서 ID 나열

### 5.2 맥락적 참조 — Markdown 링크

```markdown
상품 등록 정책은 [BO-PDM-001](../01-policies/BO-PDM-001_상품-등록-정책.md)에 따른다.
```

- **왜** 참조하는지 문맥 설명 포함
- 섹션 참조 가능: `[BO-PDM-001#규칙-1](path#규칙-1)`

---

## 6. 기획 흐름

```
Policy → Data → Component → Action → Wireframe → Test Case
  (1)     (2)      (3)        (4)       (5)         (6)
```

| 단계 | 산출물 | 입력 | 설명 |
|------|--------|------|------|
| 1. Policy | 정책 문서 | 요구사항 | 비즈니스 규칙 정의 |
| 2. Data | 데이터 문서 | Policy | 엔티티, 필드, 관계 정의 |
| 3. Component | 컴포넌트 문서 | Data | UI 구조, 컬럼, 상태 정의 |
| 4. Action | 액션 문서 | Policy + Component | API, 검증, 후속 처리 |
| 5. Wireframe | 화면 설계 | Component + Action | 레이아웃, 인터랙션 |
| 6. Test Case | 테스트 케이스 | 전체 | 기능/UI/API 테스트 |

### 참조 관계도

```
Policy ──→ Data ──→ Component ──→ Wireframe
  │                    │              ↑
  └──→ Action ─────────┘──────────────┘
                                      │
                              Test Case ←──┘
```

---

## 7. 파일 명명 규칙

```
{ID}_{title-slug}.md
```

- `ID`: Frontmatter의 `id`와 동일
- `title-slug`: 한국어 제목의 하이픈 연결 (띄어쓰기 → `-`)
- 예: `BO-PDM-001_상품-등록-정책.md`

HTML 와이어프레임:
```
{ID}_{title-slug}.html
```

Excel 테스트케이스:
```
{ID}_{title-slug}.xlsx
```

---

## 8. 확장성

| 확장 유형 | 방법 |
|----------|------|
| 새 도메인 추가 | `domains/{BO\|FO}/{folder}/` 생성 + 도메인 약어 표 등록 |
| FO 추가 | `domains/FO/` 하위에 도메인 폴더 생성, ID 접두사 `FO-` 사용 |
| 새 문서 타입 | `_schema/` 스키마 추가 + 폴더 번호 배정 |
| 다국어 | 파일 접미사 로케일 (`.ko.md`, `.en.md`) |
