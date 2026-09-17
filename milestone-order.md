# milestone-order — 시간순 이정표

> 다음 앱을 만들 때 어떤 순서로 진행할지 이정표. StartRoom 실 작업 흐름 기반.
>
> **핵심 사고 —** docs-template 은 "확정 진실 압축본" 자리. 기획·와이어프레임·디자인은 **자유 형식** 으로 먼저 하고, 확정된 것만 docs-template 으로 정합화 → 개발 착수.
>
> ⚠️ **각 phase 의 참조 template 은 그대로 쓰지 X.** 공통 원칙 (앱 무관) 은 유지 · placeholder · 예시 값 · StartRoom 특화 자리는 새 앱 정체성·도메인·화면에 맞게 **재작성** 필수. 그대로 두면 새 앱이 StartRoom clone 이 됨.

---

## 유연한 진입 (skip 가능)

이미 완료한 phase 는 skip. 다음 phase 부터 진입.

| 시작 지점 | 진행 |
|---|---|
| 아무것도 없음 | Phase 0 → 1 → 2 → 3 → 4 → 5 → 6 |
| 기획 완료 | 기존 기획문서를 `docs/planning/` 로 이관하고 Phase 1 부터 |
| 기획+와이어프레임 완료 | Phase 2 부터 |
| 기획+와이어프레임+디자인 완료 | **Phase 3 부터** (흔한 케이스) |
| 개발 중 앱 회고 정리 | Phase 3 채워서 정합화 |

---

## Phase 0 — 기획 (자유 형식)

**목적** — "무엇을 · 누구를 위해 · 왜" 확정. 제품 vision · 핵심 가설 · MVP 범위 · 수익 모델 방향 · 데이터 구조 초안.

**형식** — 자유. 앱마다 다름. Template scaffold 없음.

**산출물 위치 (권장)** — 새 앱 저장소 `docs/planning/plan.md`
- 저장소 없으면 로컬 자유 (Notion · Obsidian 등) · 저장소 생기면 `docs/planning/` 로 이관
- 이유 — git 이력 · 코드 저장소와 함께 · docs-template 과 자연 분리 ("탐색 원본" vs "확정 정합화")

**참조 template** — `docs-templates/00_planning/README.template.md` (사용 방식 3가지) · `docs-templates/00_planning/plan.template.md` (Phase 0 scaffold · §0~11 + StartRoom 인용 예시)

**참고 예시 (StartRoom)** — 저장소 안 첨부:
- `docs-templates/examples/00_planning/startroom-plan.md` (본 기획서 v2.0 · 18KB)
- `docs-templates/examples/00_planning/startroom-guide.md` (요약본 · 4KB)

**완료 기준** — 다음 phase (와이어프레임) 에서 화면 스케치 착수 가능한 정도로 제품·flow 확정.

---

## Phase 1 — 와이어프레임 (자유 형식)

**목적** — Phase 0 기획을 화면 단위로 쪼갬. ASCII 스케치 · 손그림 · 다이어그램 무엇이든. 시각 정확성 X · 화면 목록·flow·상태 정의 O.

**형식** — 자유. Template scaffold 없음.

**산출물 위치 (권장)** — 새 앱 저장소 `docs/planning/wireframe.md`
- Phase 0 plan.md 와 같은 자리 · git 이력 · 코드 저장소와 함께

**참조 template** — `docs-templates/00_planning/README.template.md` (사용 방식 3가지) · `docs-templates/00_planning/wireframe.template.md` (Phase 1 scaffold · 화면 목록 · flow · StartRoom 인용 예시)

**참고 예시 (StartRoom)** — 저장소 안 첨부:
- `docs-templates/examples/00_planning/startroom-wireframe.md` (와이어프레임 v1.1 · 31KB · 15개 화면 · ASCII)

**완료 기준** — 각 화면의 목적·상태·인터랙션이 스케치 수준으로 명확. 디자인 phase 착수 가능.

---

## Phase 2 — 디자인 (Claude 디자인 MCP · 프로토타입)

**목적** — 화면 시각·동작을 프로토타입 (React jsx) 으로 완성. 이 프로토타입이 **시각·동작의 단일 진실.** Flutter 는 이 코드의 실측값을 읽어 포팅 (새로 디자인하지 않음).

**Hybrid 방식 — iteration 은 MCP · 확정 시점 local export.**
- **iteration 중** — MCP only · 빠르게 갈아엎기 · local 파일 없음 (부담 X)
- **확정 시점** — 새 앱 저장소 `design/prototype/{screen}.jsx` 로 export · git commit
- **Phase 3~ 이후** — Flutter · docs 는 local jsx 참조 · MCP 콜 안 필요
- **디자인 변경 필요 시** — MCP 재수정 → 재export → 새 commit ("이 시점 디자인 변경" 명시)

이유 — git 이력 (디자인 결정 rollback·diff 가능) · 구현 시점 스냅샷 (사일런트 드리프트 방지) · MCP 의존 최소화 · 오프라인 grep 가능.

**순서**

1. Phase 1 와이어프레임 (`docs/planning/wireframe.md`) 기반 → Claude 디자인 MCP 에서 각 화면 프로토타입 생성
2. 프로토타입 iteration — 시각·상호작용·애니메이션 · MCP 안에서
3. **확정 → local export** `design/prototype/{screen}.jsx` × N · git commit
4. Phase 3 에서 `04_design_system.md` 확정 시 프로토타입 실측 팔레트·토큰 값 사용

**산출물 위치 (권장)** — 새 앱 저장소 `design/prototype/{screen}.jsx`

**참조 template** — `docs-templates/02_design/README.template.md` (Hybrid 방식 상세) · `CLAUDE.md.template` 룰 8 (디자인 진실)

**완료 기준** — 전 화면 프로토타입 완성 · local jsx export 완료 · 시각·동작 확정. Phase 3 docs 정합화 착수 가능.

---

## Phase 3 — Docs (확정 진실 정합화 · docs-template 채우기)

**목적** — Phase 0-2 확정된 것들을 개발 착수 전 **참조 진실 자리** 로 정합화. 코드 짜기 전에 제품·기술·화면·디자인·일정을 문서로 확정. "잘 쓴 문서로 시작하면 코드는 필요할 때 더 나은 방식으로 새로 작성 가능".

**순서 (권장)**

1. `docs/00_overview.md` — 제품 정의 · 핵심 가설 · MVP 범위 · 북극성 지표 · 확정 결정 요약
2. `docs/01_tech_stack.md` — 기술 선택 확정 (Framework · Backend · Auth · State · Analytics · Crash · Push · Deep Link)
3. `docs/02_architecture.md` — 폴더 구조 · DB 스키마 · Bloc/Cubit 패턴 · Repository 원칙
4. `docs/03_screens/README.md` — 화면 목록 · IA · flow · 상태 원칙 (전체 조망)
5. `docs/03_screens/{screen}.md × N` — 화면별 개발 handoff 확정 명세 (와이어프레임 X · 프로토타입 기반)
6. `docs/04_design_system.md` — 프로토타입 실측 토큰 · 컴포넌트 (`{PREFIX}` 접두사)
7. `docs/05_roadmap.md` — Week 1-3 마일스톤 · Day 단위 작업 가이드
8. `docs/06_revenue_model.md` — MVP → V2 (구독) → V3 (광고) staged rollout
9. `docs/07_decisions_log.md` — D-XX 결정 로그 (배경 · 결정 · 결과)
10. `docs/08_dev_ops.md` — Git · 테스트 · 환경 (dev/prod flavor) · 배포 · Supabase MCP
11. `CLAUDE.md` — 11개 절대 규칙 + 프로젝트 요약 + 문서 맵

**참조 template** — `docs-templates/03_product/00~08.template.md` + `docs-templates/03_product/03_screens/{README, screen}.template.md` + `CLAUDE.md.template`

**완료 기준** — 위 11개 문서 정합화 완료 · 개발 (Phase 4) 착수 가능.

---

## Phase 4 — Code (Claude Code · 로드맵대로 3주)

**목적** — 프로토타입 (Phase 2) + docs (Phase 3) 를 재료로 Claude Code 가 로드맵대로 앱 구현.

**Week 1 · 기반** — 환경 (dev/prod flavor · .env) · 디자인 시스템 (AppColors/AppSpacing/AppTextStyles) · 폴더 구조 · DB 스키마 (Supabase MCP dev) · Realtime 검증
**Week 2 · 뼈대·화면 완성** — mock 데이터로 모든 UI 구현
**Week 3 · 기능 탑재 + 베타** — Realtime · 매칭 · 푸시 (FCM) · 딥링크 (app_links) · 베타 배포 (TestFlight · Android Firebase App Distribution)

**참조 template** — `docs-templates/03_product/05_roadmap.template.md` · `workflow/session-workflow.md` · `stack/flutter/rule-enforcement/` · `stack/flutter/config-baseline.md` · `docs-templates/03_product/08_dev_ops.template.md` §4 (Supabase MCP + tips)

**완료 기준** — 앱 빌드 성공 · 핵심 flow 동작 · 베타 배포 가능 상태.

---

## Phase 5 — Launch (배포 준비)

**목적** — 정식 출시. QA · 스토어 메타 · 심사 대응.

**순서**

1. **QA** — 프로토타입 vs Flutter 실측 비교 (`docs-templates/05_launch/qa-test.template.md`)
2. **store-listing 작성** — 앱스토어/플레이스토어 메타 (이름·부제·설명·키워드·스크린샷) (`docs-templates/05_launch/store-listing.template.md`)
3. **legal** — 개인정보처리방침 · 이용약관 (`docs-templates/legal/`)
4. **prod-deploy** — SQL prod migration (CLI) · 앱 빌드 · 심사 제출 · 롤아웃 (`docs-templates/05_launch/prod-deploy-guide.template.md`)

**완료 기준** — App Store · Play Store 심사 통과 · 출시 완료.

---

## Phase 6 — Ops (배포 후 · 특히 중요)

**목적** — 배포 후 사용자 반응·이슈 관측 · 개선 loop 형성. **개발 시엔 그닥이었지만 배포 후엔 필수.**

**핵심 문서 (이 phase 진입 시 즉시 세팅)**

1. **amplitude-taxonomy** — 이벤트 카탈로그 (`docs-templates/06_ops/amplitude-taxonomy/catalog.template.md`) + 대시보드 spec (`docs-templates/06_ops/amplitude-taxonomy/dashboard-spec.template.md`)
   - Raw event 전수 박음 (funnel 정의는 대시보드 단계)
2. **beta-observations** — 관측 프로토콜 (`docs-templates/06_ops/beta-observations/protocol.template.md`) + 로그 template (`docs-templates/06_ops/beta-observations/log-template.template.md`)
   - 주기적 관측 · 트리아지 flow
3. **Sentry alerts · Firebase App Distribution · 사용자 피드백 분류** — 각각 setup

**참조 template** — `docs-templates/06_ops/*` · `docs-templates/03_product/08_dev_ops.template.md` §5 (Sentry + triage flow)

**완료 기준** — 지속적 관측·개선 loop 정착 (일회성 아님).

---

## Phase 전반 · 공통 (매 회차 진입 시)

- **`workflow/session-workflow.md`** — 회차 시작·수행·commit·머지 흐름
- **`workflow/git-flow.md`** — 브랜치 네이밍 (chore/feat/fix/refactor/epic + sub-work)
- **`workflow/docs-work-naming.md`** — `docs/work/{category}/{topic}-{yyyy-mm}/` 규칙

---

## 이정표 축약도

Phase → 산출물 위치 매핑 (신규 앱 저장소 기준) —

```
Phase 0        Phase 1              Phase 2                 Phase 3               Phase 4      Phase 5      Phase 6
기획           와이어프레임          디자인                   Docs 정합화            Code         Launch       Ops
자유           자유                 Claude 디자인 MCP        docs-template         Claude Code  배포 준비    배포 후 · 필수
   ↓              ↓                    ↓                      ↓                      ↓            ↓            ↓
docs/          docs/                design/prototype/       docs/00~08.md         lib/         스토어       docs/work/
planning/      planning/            {screen}.jsx × N        docs/03_screens/                    등록         amplitude · beta-obs
plan.md        wireframe.md         (MCP iteration →         README.md +                                     Sentry · 피드백
                                     확정 시 export)         {screen}.md × N
```

**참조 template 매핑 —**

```
Phase 0    docs-templates/00_planning/README.template.md + plan.template.md         (scaffold + 유연성)
Phase 1    docs-templates/00_planning/README.template.md + wireframe.template.md   (scaffold + 유연성)
Phase 2    docs-templates/02_design/README.template.md       (Hybrid 상세) + CLAUDE.md.template 룰 8
Phase 3    docs-templates/03_product/00~08.template.md + 03_screens/{README, screen}.template.md
Phase 4    docs-templates/03_product/05_roadmap.template.md + stack/flutter/*
Phase 5    docs-templates/05_launch/*
Phase 6    docs-templates/06_ops/*
```

- **Phase 0~2 = 자유 탐색** · 형식 자유 · Phase 2 는 MCP iteration 후 확정 시점 local export
- **Phase 3 = 정합화** · docs-template 채워 개발 진실 자리 확정
- **Phase 4~6 = 실행** · 참조는 Phase 3 문서
- **Skip 가능** — 완료된 phase 는 건너뜀 (예: 기획+디자인 이미 있으면 Phase 3 부터)
