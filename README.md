# app-methodology

> StartRoom 제작에서 검증된 knowledge (기획·디자인·개발·툴·워크플로) 를 다음 앱들에서 재사용할 template 로 문서화한 저장소.
> 사용자 실 작업 흐름을 그대로 재현 가능한 형태.

---

## 한눈에 (Phase × 산출물 × 참조)

| Phase | 성격 | 신규 앱 산출물 위치 | 이 저장소 참조 |
|---|---|---|---|
| **0. 기획** | 자유 · 탐색 | `docs/planning/plan.md` | `docs-templates/00_planning/plan.template.md` |
| **1. 와이어프레임** | 자유 · 탐색 | `docs/planning/wireframe.md` | `docs-templates/00_planning/wireframe.template.md` |
| **2. 디자인** | MCP iteration → 확정 시 export | `design/prototype/{screen}.jsx` × N | `docs-templates/02_design/README.template.md` |
| **3. Docs 정합화** | 확정 진실 | `docs/00~08.md` + `docs/03_screens/` + `CLAUDE.md` | `docs-templates/03_product/*` + `CLAUDE.md.template` |
| **4. Code** | 3주 실행 | `lib/` | `docs-templates/03_product/05_roadmap.template.md` + `stack/flutter/*` |
| **5. Launch** | 배포 준비 | 스토어 · 심사 · legal | `docs-templates/05_launch/*` + `docs-templates/legal/*` |
| **6. Ops** | 배포 후 필수 | `docs/work/...` + 대시보드 | `docs-templates/06_ops/*` |

**매 회차 (Phase 3~ 진입 시)** — [`workflow/session-workflow.md`](./workflow/session-workflow.md) 따라 새 브랜치 → plan.md → 수행 → analyze/test → 사용자 OK → 의미 단위 commit → 머지.

**Skip 가능** — 이미 완료된 phase 는 건너뜀 (예: 기획+디자인 완료 시 Phase 3 부터). Phase 별 상세 진입 순서 = [`milestone-order.md`](./milestone-order.md).

---

## 이 저장소 사용법 (다음 앱 시작 시)

> ⚠️ **Template 은 참조 자료 · 그대로 복사해서 쓰지 X.** 각 template 의 **공통 원칙** (앱 무관 · WCAG · 아키텍처 등) 은 유지하되 **placeholder · 예시 값 · StartRoom 특화 자리** 는 반드시 새 앱 정체성·도메인·화면에 맞게 재작성. 그대로 두면 새 앱이 StartRoom clone 이 됨.

**핵심 사고 —** docs-template 은 "확정 진실 압축본" 자리. 기획·와이어프레임·디자인은 **자유 형식** 으로 먼저 하고, 확정된 것만 docs-template 으로 정합화 → 개발 착수.

### Phase → 산출물 위치 (신규 앱 저장소 기준)

```
Phase 0    기획          → docs/planning/plan.md              (자유 · 원본)
Phase 1    와이어프레임   → docs/planning/wireframe.md          (자유 · 원본)
Phase 2    디자인         → design/prototype/{screen}.jsx      (Claude 디자인 MCP · Hybrid · 확정 시 export)
Phase 3    Docs 정합화    → docs/00~08.md + docs/03_screens/    (docs-template · 개발 진실)
Phase 4    Code          → lib/                                (Claude Code · 로드맵대로)
Phase 5    Launch        → 스토어 등록 · 심사                    (05_launch/ 참조)
Phase 6    Ops           → docs/work/... + 대시보드              (06_ops/ 참조 · 배포 후 필수)
```

- **Phase 0~2** = 자유 탐색 · 형식 자유 · docs-template scaffold 없음 (안내 README 만)
- **Phase 3** = 정합화 · docs-template 참조해서 채움
- **Phase 4~6** = 실행 · Phase 3 문서 참조

### 진입 순서

1. **`milestone-order.md` 먼저 읽기** — 시간순 이정표 (Phase 0~6 상세) · skip 가능 (완료된 phase 는 건너뜀)
2. **Phase 0-2 (기획·와이어프레임·디자인)** — Phase 0-1 은 scaffold (`docs-templates/00_planning/plan.template.md` · `wireframe.template.md`) 채우거나 자유 형식 · Phase 2 는 Claude 디자인 MCP (Hybrid 안내 `docs-templates/02_design/README.template.md`)
3. **Phase 3 (Docs 정합화)** — `CLAUDE.md.template` 복사 → 새 앱 저장소 root 에 `CLAUDE.md` · placeholder 치환. `docs-templates/03_product/*.template.md` 복사 → 새 앱 `docs/` 에 재작성
4. **Phase 4-6 (Code · Launch · Ops)** — 각 phase 참조 template 사용
5. **매 회차 진입 시 `workflow/session-workflow.md` 참조** — plan → 진입 commit → 수행 → analyze+test → 사용자 OK → 의미 단위 commit → 머지

---

## 폴더 구조

```
├── milestone-order.md              # 시간순 이정표 (여기서 시작 · Phase 0~6)
├── CLAUDE.md.template              # 프로젝트 CLAUDE.md 원형 (11개 절대 규칙)
├── docs-templates/
│   ├── 00_planning/                # (Phase 0-1) README + plan/wireframe scaffold + 유연성 (자유 형식·skip 가능)
│   ├── 02_design/                  # (Phase 2) Claude 디자인 MCP · Hybrid 안내 · README 만
│   ├── 03_product/                 # (Phase 3) Docs 정합화 — 00~08 templates
│   │   └── 03_screens/             #   README (화면 아키텍처) + screen (개별 명세)
│   ├── 05_launch/                  # (Phase 5) 배포 준비 (qa-test · store-listing · prod-deploy-guide)
│   ├── 06_ops/                     # (Phase 6) 배포 후 — amplitude · beta-observations
│   ├── legal/                      # 개인정보·약관 (cross-cutting · 번호 없음)
│   └── examples/                   # StartRoom 실 예시 (phase 별 하위 · 참조 전용 · fork 참고)
│       ├── 00_planning/            #   startroom-plan · guide · wireframe
│       ├── 05_launch/              #   startroom-prod-deploy-guide-{회차}.md
│       └── 06_ops/                 #   startroom-catalog.md
├── workflow/                       # 회차 workflow · git flow · 명명 규칙
└── stack/flutter/                  # (Phase 4) Flutter 스택 특화
    ├── rule-enforcement/           # 절대 규칙 실현 코드 예시 (룰별)
    └── config-baseline.md          # pubspec · analysis_options · launcher_icons
```

**폴더 = Phase 매핑 —**
- `docs-templates/00_planning/` = Phase 0-1 (README + plan/wireframe scaffold · 유연성 삼중 지원)
- `docs-templates/02_design/` = Phase 2 (Hybrid · README 만)
- `docs-templates/03_product/` = Phase 3 (확정 진실 · scaffold 있음)
- `stack/flutter/` = Phase 4 (스택 특화 · docs-templates/ 밖)
- `docs-templates/05_launch/` = Phase 5
- `docs-templates/06_ops/` = Phase 6

---

## 원칙 (지도)

1. **무겁지 않게** — 폴더·문서 수 최소. 넣을지 뺄지 애매하면 뺀다
2. **헷갈릴 소지 없게** — 사용법·역할이 한눈에 명확한 것만 남긴다
3. **실 사용된 것만 문서화** — StartRoom 에서 실제로 활용된 자산만. "쓰면 좋을 것" 은 필요해질 때 추가 (YAGNI)
4. **StartRoom = reference 구현체** — 이 저장소가 다루지 않는 상세는 startroom_app 직접 참고

---

## 진행 상황

- **v0.1.0** — 회차 2 이관 (2026-09~)
- **v0.2.0** — 회차 3 workflow 시각화 (2026-09-17) · 폴더 phase-numbered (00_planning · 02_design · 03_product · 05_launch · 06_ops) · 7-phase 재구성 · plan/wireframe scaffold + StartRoom examples
- **v1.0.0** — 신규 앱 dry-run 반영 back-port 완료 후

---

## 관련 저장소

- **StartRoom (reference 구현체)** — `/Users/haeincho/project/app/startroom_app`
- 다음 신규 앱 — 회차 3 dry-run 대상 (미정)
