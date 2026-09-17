# milestone-order — 시간순 이정표

> 다음 앱을 만들 때 어떤 순서로 진행할지 이정표.
> StartRoom 실 작업 흐름 기반 — 앱 코드 없이 docs 8개로 시작 → Claude 디자인 MCP → Claude Code 로드맵 → 배포 → 운영.
>
> ⚠️ **각 phase 의 참조 template 은 그대로 쓰지 X.** 공통 원칙 (앱 무관) 은 유지 · placeholder · 예시 값 · StartRoom 특화 자리는 새 앱 정체성·도메인·화면에 맞게 **재작성** 필수. 그대로 두면 새 앱이 StartRoom clone 이 됨.

---

## Phase 0 — Docs-first (앱 코드 없이 문서 먼저)

**목적** — 코드 짜기 전에 제품·기술·화면·디자인·일정을 문서로 확정. "잘 쓴 문서로 시작하면 코드는 필요할 때 더 나은 방식으로 새로 작성 가능".

**순서 (권장)**

1. `docs/00_overview.md` — 제품 정의 · 핵심 가설 · MVP 범위 · 북극성 지표
2. `docs/01_tech_stack.md` — 기술 선택 (Framework · Backend · Auth · State · Analytics · Crash · Push · Deep Link)
3. `docs/02_architecture.md` — 폴더 구조 · DB 스키마 · Bloc/Cubit 패턴 · Repository 원칙
4. `docs/03_screens/` — 화면별 명세 (라우트 · 상태 · UI 컴포넌트 · 엣지케이스)
5. `docs/04_design_system.md` — 컬러 · 폰트 · 간격 · 공통 컴포넌트 (SR 접두사)
6. `docs/05_roadmap.md` — Week 1-3 마일스톤 · Day 단위 작업 가이드
7. `docs/06_revenue_model.md` — MVP → V2 (구독) → V3 (광고) staged rollout
8. `docs/07_decisions_log.md` — D-XX 결정 로그 (배경 · 결정 · 결과)
9. `docs/08_dev_ops.md` — Git · 테스트 · 환경 (dev/prod flavor) · 배포 · Supabase MCP
10. `CLAUDE.md` — 11개 절대 규칙 + 프로젝트 요약 + 문서 맵

**참조 template** — `docs-templates/product/00~08.template.md` + `CLAUDE.md.template`

**완료 기준** — 위 10개 문서 초안 완성 · 다음 단계 (디자인) 착수 가능

---

## Phase 1 — Design (Claude 디자인 MCP · 프로토타입)

**목적** — 화면 시각·동작을 프로토타입 (React jsx) 으로 완성. 이 프로토타입이 **시각·동작의 단일 진실**. Flutter 는 이 코드의 실측값을 읽어 포팅 (새로 디자인하지 않음).

**순서**

1. `docs/`와 `docs/03_screens/` 기반 → Claude 디자인에서 각 화면 프로토타입 생성
2. Claude Code에서 MCP를 통해 Claude 디자인 산출물 체크
3. 프로토타입 iteration — 시각·상호작용·애니메이션 확정
4. `docs/04_design_system.md` 확정 — 프로토타입 실측 팔레트·토큰 값

**참조 template** — CLAUDE.md 룰 8 · `docs-templates/product/04_design_system.template.md` · `docs-templates/launch/qa-test.template.md`

**완료 기준** — 전 화면 프로토타입 완성 · 디자인 시스템 확정 · 다음 단계 (코드) 착수 가능

---

## Phase 2 — Code (Claude Code · 로드맵대로 3주)

**목적** — 프로토타입 + docs 를 재료로 Claude Code 가 로드맵대로 앱 구현.

**Week 1 · 기반** — 환경 (dev/prod flavor · .env) · 디자인 시스템 (AppColors/AppSpacing/AppTextStyles) · 폴더 구조 · DB 스키마 (Supabase MCP dev) · Realtime 검증
**Week 2 · 뼈대·화면 완성** — mock 데이터로 모든 UI 구현
**Week 3 · 기능 탑재 + 베타** — Realtime · 매칭 · 푸시 (FCM) · 딥링크 (app_links) · 베타 배포 (TestFlight · Android Firebase App Distribution)

**참조 template** — `docs-templates/product/05_roadmap.template.md` · `workflow/session-workflow.md` · `stack/flutter/rule-enforcement/` · `stack/flutter/config-baseline.md` · `docs-templates/product/08_dev_ops.template.md` §4 (Backend MCP + tips)

**완료 기준** — 앱 빌드 성공 · 핵심 flow 동작 · 베타 배포 가능 상태

---

## Phase 3 — Launch (배포 준비)

**목적** — 정식 출시. QA · 스토어 메타 · 심사 대응.

**순서**

1. **QA** — 프로토타입 vs Flutter 실측 비교 (`docs-templates/launch/qa-test.template.md`)
2. **store-listing 작성** — 앱스토어/플레이스토어 메타 (이름·부제·설명·키워드·스크린샷) (`docs-templates/launch/store-listing.template.md`)
3. **legal** — 개인정보처리방침 · 이용약관 (`docs-templates/legal/`)
4. **prod-deploy** — SQL prod migration (CLI) · 앱 빌드 · 심사 제출 · 롤아웃 (`docs-templates/launch/prod-deploy-guide.template.md`)

**완료 기준** — App Store · Play Store 심사 통과 · 출시 완료

---

## Phase 4 — Ops (배포 후 · 특히 중요)

**목적** — 배포 후 사용자 반응·이슈 관측 · 개선 loop 형성. **개발 시엔 그닥이었지만 배포 후엔 필수.**

**핵심 문서 (이 phase 진입 시 즉시 세팅)**

1. **amplitude-taxonomy** — 이벤트 카탈로그 (`docs-templates/ops/amplitude-taxonomy/catalog.template.md`) + 대시보드 spec (`docs-templates/ops/amplitude-taxonomy/dashboard-spec.template.md`)
   - Raw event 전수 박음 (funnel 정의는 대시보드 단계)
2. **beta-observations** — 관측 프로토콜 (`docs-templates/ops/beta-observations/protocol.template.md`) + 로그 template (`docs-templates/ops/beta-observations/log-template.template.md`)
   - 주기적 관측 · 트리아지 flow
3. **Sentry alerts · Firebase App Distribution · 사용자 피드백 분류** — 각각 setup

**참조 template** — `docs-templates/ops/*` · `docs-templates/product/08_dev_ops.template.md` §5 (Sentry + triage flow)

**완료 기준** — 지속적 관측·개선 loop 정착 (일회성 아님)

---

## Phase 전반 · 공통 (매 회차 진입 시)

- **`workflow/session-workflow.md`** — 회차 시작·수행·commit·머지 흐름
- **`workflow/git-flow.md`** — 브랜치 네이밍 (chore/feat/fix/refactor/epic + sub-work)
- **`workflow/docs-work-naming.md`** — `docs/work/{category}/{topic}-{yyyy-mm}/` 규칙

---

## 이정표 축약도

```
Phase 0                Phase 1              Phase 2               Phase 3           Phase 4
Docs-first             Design               Code                  Launch            Ops
(앱 코드 X)            (Claude 디자인 MCP)   (Claude Code · 3주)   (배포 준비)       (배포 후 · 필수)
   ↓                      ↓                    ↓                      ↓                 ↓
10개 문서              프로토타입 (React)    앱 빌드 · 베타         스토어 등록       amplitude · beta-obs
                                                                                     Sentry · 피드백
```
