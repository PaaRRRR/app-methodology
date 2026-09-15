# StartRoom → app-methodology 인벤토리 (회차 1 산출)

> 스캔 일자 — 2026-09-15
> 스캔 범위 — `/Users/haeincho/project/app/startroom_app` 전수
> 방향성 — 사용자 확정 (2026-09-15) — template = **시간순 이정표 + 문서 template (placeholder)** 중심. 코드 승격은 부차적. lib 는 룰 실현 asset 만 축소, supabase 는 원칙+tips 문서 1개, root 는 CLAUDE.md/README.md + 설정 baseline
> 원칙 — 무겁지 않게 · 헷갈릴 소지 X · 실 사용된 것만 · YAGNI

---

## §1 최종 template 그림 (회차 2 이관 대상)

```
app-methodology/
├── README.md                                    # 저장소 사용법 · 다음 앱 시작 순서
├── milestone-order.md                           # ★ 시간순 이정표 (Docs → Design → Code → 배포 → 운영)
├── CLAUDE.md.template                           # 프로젝트 CLAUDE.md 원형 (11개 절대 규칙 · placeholder)
├── docs-templates/
│   ├── product/                                 # (앱 개발 전) Docs-first
│   │   ├── 00_overview.template.md
│   │   ├── 01_tech_stack.template.md
│   │   ├── 02_architecture.template.md
│   │   ├── 03_screens.template.md
│   │   ├── 04_design_system.template.md
│   │   ├── 05_roadmap.template.md
│   │   ├── 06_revenue_model.template.md
│   │   ├── 07_decisions_log.template.md
│   │   └── 08_dev_ops.template.md
│   ├── launch/                                  # (배포 준비)
│   │   ├── store-listing.template.md
│   │   ├── qa-test.template.md
│   │   ├── release-checklist.template.md
│   │   └── prod-deploy-guide.template.md
│   ├── ops/                                     # ★ (배포 후 · 강조)
│   │   ├── amplitude-taxonomy/
│   │   │   ├── catalog.template.md
│   │   │   └── dashboard-spec.template.md
│   │   └── beta-observations/
│   │       ├── protocol.template.md
│   │       └── log-template.template.md
│   └── legal/
│       ├── privacy.template.md
│       └── terms.template.md
├── workflow/                                    # 작업 방식
│   ├── session-workflow.md                      # docs/work/{회차}/ 진행 방식
│   ├── git-flow.md                              # chore/feat/fix/refactor/epic + sub-work
│   └── docs-work-naming.md                      # {category}/{topic}-{yyyy-mm} 규칙
├── mcp/                                         # MCP 활용
│   ├── claude-design.md                         # 프로토타입 (React jsx) → Flutter 포팅 flow
│   ├── supabase-dev-vs-prod.md                  # ★ dev=MCP · prod=CLI + tips 브레인스토밍
│   └── sentry.md
└── stack/flutter/                               # Flutter 스택 특화 (코드 예시 문서)
    ├── rule-enforcement/                        # 절대 규칙 실현 asset (룰별 6개)
    │   ├── clock-and-kst.md                     # 룰 #11 (시간 단일 진실)
    │   ├── design-tokens.md                     # 룰 #2 (하드코딩 금지)
    │   ├── sr-widgets.md                        # 룰 #1 (컴포넌트 재사용 · SR 접두사)
    │   ├── bloc-observer-sentry.md              # 룰 #6 (에러 삼키지 않음)
    │   ├── flavor-setup.md                      # 룰 #7 (dev/prod 분리)
    │   └── bloc-vs-cubit.md                     # 룰 #4/#5 (Realtime 구독 해제)
    └── config-baseline.md                       # pubspec + analysis_options + launcher_icons (편집 사항 · 코드 블록)
```

**총 문서 ~36개.** 폴더별 목적 명확 · 헷갈릴 소지 X · 룰 강제 asset 은 룰별 분리 lookup 편함.

---

## §2 대영역별 승격 매핑

### 2-A. docs/ → docs-templates/

| StartRoom 원본 | app-methodology 이관 위치 | 일반화 작업 |
|---|---|---|
| `docs/00_overview.md` | `docs-templates/product/00_overview.template.md` | StartRoom 도메인 사례 → placeholder + "예시 블록" 으로 분리 |
| `docs/01_tech_stack.md` | `docs-templates/product/01_tech_stack.template.md` | 스택 선택 원칙 + StartRoom 사례 예시 |
| `docs/02_architecture.md` | `docs-templates/product/02_architecture.template.md` | 폴더 구조·BLoC/Repository 패턴 원칙. StartRoom 테이블은 예시 블록 |
| `docs/03_screens/*` | `docs-templates/product/03_screens.template.md` (골격만) | 화면 정의 template + 예시 1개 (StartRoom 화면 하나) |
| `docs/04_design_system.md` | `docs-templates/product/04_design_system.template.md` | 이중 모드·토큰 원칙. 팔레트 구체값은 skeleton |
| `docs/05_roadmap.md` | `docs-templates/product/05_roadmap.template.md` | Week 1-3 골격 (기반→뼈대→화면→연결→기능). Day 단위는 placeholder |
| `docs/06_revenue_model.md` | `docs-templates/product/06_revenue_model.template.md` | staged rollout (MVP→V2→V3) 패턴만 |
| `docs/07_decisions_log.md` | `docs-templates/product/07_decisions_log.template.md` | 의사결정 로그 포맷 (D-XX · 배경 · 결정 · 결과) + StartRoom 예시 2~3개 |
| `docs/08_dev_ops.md` | `docs-templates/product/08_dev_ops.template.md` | flavor 분리·GitHub Flow·.env 주입 원칙 |
| `docs/store-listing.md` | `docs-templates/launch/store-listing.template.md` | 앱스토어/플레이스토어 메타 필드 목록 (앱명·부제·설명·키워드·스크린샷 spec) |
| `docs/release-checklist.md` | `docs-templates/launch/release-checklist.template.md` | 출시 차단·권장·사용자 결정 3분류 골격 |
| `docs/work/home-ia-redesign/prod-deploy-guide.md` | `docs-templates/launch/prod-deploy-guide.template.md` | prod 배포 순서 (SQL 적용·앱 빌드·심사 제출·롤아웃) |
| `_qa/visual_qa.md` + `_qa/README.md` | `docs-templates/launch/qa-test.template.md` | 프로토타입 vs Flutter 실측 비교 QA 프로세스 |
| `docs/work/amplitude-taxonomy/catalog.md` | `docs-templates/ops/amplitude-taxonomy/catalog.template.md` | 이벤트 네이밍·property 표준·funnel 분리 원칙 |
| `docs/work/amplitude-taxonomy/dashboard-spec.md` | `docs-templates/ops/amplitude-taxonomy/dashboard-spec.template.md` | 대시보드 chart 유형·segment 정의 |
| `docs/work/beta-observations/protocol.md` | `docs-templates/ops/beta-observations/protocol.template.md` | 관측 프로토콜 (수집 대상·주기·트리아지 flow) |
| `docs/work/beta-observations/log-template.md` | `docs-templates/ops/beta-observations/log-template.template.md` | 관측 로그 표준 포맷 |
| `docs/legal/privacy.md` | `docs-templates/legal/privacy.template.md` | GDPR·PIPA 정합 구조 + 수집 항목 placeholder |
| `docs/legal/terms.md` | `docs-templates/legal/terms.template.md` | 약관 구조 + 서비스 특화 조항 placeholder |

**제외** — `backlog.md`, `docs/work/` 54개 회차 (history 로만).

### 2-B. lib/ → stack/flutter/rule-enforcement/ (룰별 6개 문서)

| 룰 | 관련 StartRoom 코드 | app-methodology 문서 |
|---|---|---|
| #1 컴포넌트 재사용 (Sr 접두사) | `lib/shared/widgets/{sr_button, sr_card, sr_input, sr_icon_button, sr_tag, sr_screen_header, sr_sheet_handle, sr_screen_skeleton, sr_loading_view, sr_empty_view, sr_error_view}.dart` | `stack/flutter/rule-enforcement/sr-widgets.md` — 기본 위젯 세트 목록 + 각 위젯 최소 예시 (button/card 상세, 나머지는 목록) |
| #2 하드코딩 금지 | `lib/app/theme.dart` (Palette, TextStyles, Spacing, Radius), `lib/app/motion.dart` | `stack/flutter/rule-enforcement/design-tokens.md` — 토큰 카테고리 (색·간격·radius·모션) 및 skeleton 코드 |
| #4 UI→Bloc→Repository · #5 Realtime 구독 해제 | `lib/app/bloc_observer.dart` + Bloc/Cubit 사용 관행 | `stack/flutter/rule-enforcement/bloc-vs-cubit.md` — 복잡=Bloc·단순=Cubit 원칙, StreamSubscription 해제 패턴 코드 |
| #6 에러 삼키지 않음 | `lib/app/bloc_observer.dart`, `lib/core/errors/app_error.dart` | `stack/flutter/rule-enforcement/bloc-observer-sentry.md` — Sentry 통합 + sealed AppError 코드 |
| #7 dev/prod 섞지 않음 | `lib/core/env.dart`, `lib/entry/entry.{dev,prod}.dart` | `stack/flutter/rule-enforcement/flavor-setup.md` — --dart-define-from-file · flavor 진입점 코드 |
| #11 시간 단일 진실 | `lib/core/datetime/{clock, kst, format_duration}.dart` | `stack/flutter/rule-enforcement/clock-and-kst.md` — appNow · toKst · kKstOffset 코드 |

**제외** — 나머지 lib/* 는 다음 앱 실 필요 시점에 startroom_app (reference 구현체) 참고해 새로 작성.

### 2-C. supabase/ → mcp/supabase-dev-vs-prod.md (문서 1개, tips 통합)

**이관 대상 = 문서 1개만.** SQL 파일 승격 안 함.

`mcp/supabase-dev-vs-prod.md` 구조:
1. 원칙 — dev = MCP `apply_migration` · prod = Supabase CLI `db push`
2. 신규 프로젝트 setup 순서 (dev/prod 각각)
3. **tips 섹션 (사용자 지시 = 회차 2 진입 시 실 케이스 브레인스토밍 후 정리)**
   - 브레인스토밍 소재 (StartRoom 경험) — migration 표기명 문제 · MCP list_migrations 누락 이슈 ([[reference_supabase_migration_channel]]) · SECURITY DEFINER search_path 고정 · anon EXECUTE 회수 · RLS 헬퍼 함수 (`is_room_member`) · soft delete (`left_at IS NULL`) · GDPR delete-account RPC · Realtime publication · edge function CORS · pg_net trigger

**제외** — `supabase/{config.toml, functions/*, migrations/*}` 파일 승격 안 함. 실 패턴 필요 시 startroom_app supabase/ 참고.

### 2-D. root → CLAUDE.md.template + README.md.template + stack/flutter/config-baseline.md

| StartRoom 원본 | app-methodology 이관 | 일반화 |
|---|---|---|
| `CLAUDE.md` | `CLAUDE.md.template` (root) | 11개 절대 규칙 유지. "프로젝트 한 줄 요약·현재 단계" placeholder |
| `README.md` | `README.md.template` (root) | 앱별 재작성 골격 (섹션 순서·badge 배치) |
| `pubspec.yaml` + `analysis_options.yaml` + `flutter_launcher_icons.yaml` | `stack/flutter/config-baseline.md` (통합 1개) | 각 파일별 섹션 + 코드 블록 + "이 부분 편집" 표시 |

**Flutter 생성 시 자동 vs 편집됨:**
- `analysis_options.yaml` — 생성 시 `flutter_lints` 만. freezed/json_serializable 예외 규칙 **편집**
- `pubspec.yaml` — 생성 시 최소. bloc·freezed·go_router·supabase·sentry·amplitude 등 **크게 편집**
- `flutter_launcher_icons.yaml` — 생성 시 없음. 패키지 설치 후 **직접 작성**
- `devtools_options.yaml` — Flutter 3.19+ 자동 · 비어있음 · 편집 안 함 → 제외

**제외** — `devtools_options.yaml`, `.claude/settings.local.json`, `.github/` (비어있음 확인), `scripts/`, `packages/`, `design/`, `assets/`, `screenshots/`, `test/`. 다음 앱 실 필요 시 startroom_app 참고.

### 2-E. workflow/ · mcp/ (신규 문서 · StartRoom 관행 정리)

StartRoom 특정 파일 대응 없고 관행 자체를 문서화.

| 문서 | 출처 |
|---|---|
| `workflow/session-workflow.md` | StartRoom 관행 — plan → 진입 commit → 수행 → analyze+test → 사용자 OK → 의미 단위 commit → 머지 |
| `workflow/git-flow.md` | CLAUDE.md 룰 #10 + [[feedback-multi-session-umbrella-branch]] |
| `workflow/docs-work-naming.md` | 오늘 확정 규칙 (`{category}/{topic}-{yyyy-mm}`) |
| `mcp/claude-design.md` | StartRoom 관행 — 프로토타입 (React jsx) 을 시각·동작 단일 진실로 삼고 Flutter 포팅 |
| `mcp/sentry.md` | StartRoom 관행 — issue triage flow |

---

## §3 회차 2 작업 노트

### 3-A. 이관 순서 (권장)

1. **README.md + milestone-order.md + CLAUDE.md.template** (root 3개) — 저장소 얼굴부터
2. **docs-templates/product/00~08** (9개) — Docs-first 흐름의 재료
3. **workflow/ + mcp/** (6개) — 작업 방식·MCP 활용 문서화
4. **docs-templates/launch/ + ops/ + legal/** (10개) — 배포·운영·법무
5. **stack/flutter/** (7개) — Flutter 스택 특화 코드 예시
6. **v0.1.0 tag**

### 3-B. 문서별 유의사항

- **모든 문서에 StartRoom 사례 예시 블록 첨부** — 추상 설명만은 재현 어려움. 실 사례 있어야 다음 앱에서 즉시 활용
- **milestone-order.md** — 사용자 실 작업 흐름 반영 (앱 코드 없이 docs/ 8개로 시작 → Claude 디자인 MCP → Claude Code 로드맵 진행 → 배포 → 운영)
- **ops/amplitude-taxonomy · beta-observations** — 사용자 강조. 배포 후 유용성 명시. 앱 개발 시엔 그닥이지만 배포 후 필수
- **mcp/supabase-dev-vs-prod.md** — 사용자 지시 = 실 케이스 브레인스토밍 후 tips 정리 (§2-C 브레인스토밍 소재 목록 활용)
- **stack/flutter/rule-enforcement/*.md** — 각 문서 첫 줄에 관련 룰 번호 명시. 코드 블록은 그대로 복붙 가능한 형태

### 3-C. Placeholder 규칙

- 앱 이름 — `{APP_NAME}`
- 앱 한 줄 요약 — `{ONE_LINER}`
- 저장소 이름 — `{REPO}`
- Supabase 프로젝트 — `{SUPABASE_PROJECT_ID}`
- 앱 도메인 특화 사례 — "예시 블록" (StartRoom 사례) 로 분리해 별도 인용

---

## §4 다음 단계 (회차 2)

- `app-methodology` 저장소에 §1 그림대로 폴더·파일 생성 · 승격 이관
- 각 문서에 §3 유의사항 반영
- v0.1.0 tag
- 이후 회차 3 (신규 앱 dry-run) 에서 template gap 발견 · 회차 4 back-port · v1.0.0

**유보 (지금 안 넣음)** — 회차 3 dry-run 에서 실 필요 확인 후 판단. 미리 넣지 않음 (§0 원칙 3 · YAGNI).
