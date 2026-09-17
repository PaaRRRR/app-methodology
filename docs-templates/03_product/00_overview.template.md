# 00. {APP_NAME} 프로젝트 개요

> {ONE_LINER}

이 문서는 {APP_NAME} 의 제품 정의, 핵심 가설, MVP 범위, 그리고 개발 착수 전 확정한 모든 핵심 결정사항을 담는다. 다른 모든 문서의 기준점이 되는 단일 진실 공급원 (source of truth).

> **성격 —** Phase 0 (기획) · Phase 1 (와이어프레임) · Phase 2 (디자인) 확정 후 채우는 **진실 요약** 자리. 탐색 문서 X · 확정본 O.
> Phase 0-1 자유 형식 원본은 `docs/planning/plan.md` · `docs/planning/wireframe.md` · Phase 2 프로토타입은 `design/prototype/*.jsx`. 이 문서는 그것들에서 확정된 것만 정합화.

---

## 1. 제품 정의

### 한 줄 정의
{APP_NAME} 은 {WHO} 가 {WHAT} 하고, {OUTCOME} 하는 앱이다.

### 핵심 포지셔닝
{APP_NAME} 의 본질은 {ESSENCE}.

| 서비스 | 포지션 |
|---|---|
| {COMPETITOR_A} | {POSITION_A} |
| {COMPETITOR_B} | {POSITION_B} |
| **{APP_NAME}** | **{OUR_POSITION}** |

### 피칭 문장
{PITCH_SENTENCE}

---

## 2. 핵심 문제

{TARGET_USER} 는 구조적으로 {PROBLEM_ROOT_CAUSE}. {CONTEXT}.

핵심 문제 (2~3개):
- **{PROBLEM_1}** — {EXPLAIN}
- **{PROBLEM_2}** — {EXPLAIN}
- **{PROBLEM_3}** — {EXPLAIN}

---

## 3. 핵심 가설

> "{CORE_HYPOTHESIS}"

**검증 가설 2~3개.**
1. **{HYPOTHESIS_1}** — {측정 방식}
2. **{HYPOTHESIS_2}** — {측정 방식}

---

## 4. 와우모먼트 (앱마다 개수 다름 · 2~4개)

와우모먼트는 한 번이 아니라 연속된 감각으로 이루어진다.
1. **{WOW_1}** — "{USER_FEELING}"
2. **{WOW_2}** — "{USER_FEELING}"

가장 강력한 바이럴 순간은 {SHARE_TRIGGER} 일 때다. {WHY_VIRAL}.

---

## 5. 북극성 지표 (North Star)

**{METRIC_NAME}** — {한 줄 설명}

{정의 · 카운트 방식 · 왜 이 지표인가}

| 목표 | 값 |
|---|---|
| {STAGE_1} | {VALUE_1} |
| {STAGE_2} | {VALUE_2} |

---

## 6. MVP 범위 (V1)

**포함** — {IN_SCOPE}
**제외 (V2+)** — {OUT_OF_SCOPE}

---

## 7. V2+ 로드맵 방향

- **V2** — {V2_THEME} (예: 구독)
- **V3** — {V3_THEME} (예: 광고 · 데이터 기반 결정)
- 상세 — `06_revenue_model.md`

---

## 8. 화면 목록

| 화면 | 라우트 | 핵심 상태 |
|---|---|---|
| Onboarding | `/onboarding` | 3단계 (가치 · 권한 · 준비 완료) |
| Home | `/` | {empty / active / ...} |
| {화면} | `/{route}` | {상태} |
| Settings | `/settings` | 프로필 · 알림 · 계정 |

화면 아키텍처 · IA · flow — `03_screens/README.md`
개별 화면 상세 — `03_screens/{화면}.md`

---

## 9. 확정된 핵심 결정사항

> 상세 내용은 각 전용 문서 참조. 변경 이력은 `07_decisions_log.md`.

### 기술 스택

| 영역 | 결정 |
|---|---|
| Framework | Flutter {버전} |
| State | BLoC + Cubit 혼용 |
| Routing | go_router |
| Models | freezed + json_serializable |
| Backend | Supabase (+ MCP 연동) |
| Auth | Supabase 익명 인증 |
| Push | FCM 직접 구현 |
| Analytics | Amplitude |
| Crash | Sentry (Day 1 부터) |
| Deep Link | app_links + Universal Link |
| Domain | {DOMAIN} |

### 제품 결정

- **{핵심 mechanic}** — {결정 요약} (V2 이연 여부)
- **인증** — 익명 자동 (닉네임 자동 생성 — "{닉네임 패턴}")
- **{화면 A}** — {결정 요약} ({관련 D-XX})
- **디자인** — accent `{ACCENT_HEX}` + {FONT_FAMILY} · {mode 정책}
- **{도메인 특수 결정}** — {요약} ({관련 D-XX})
- **디자인 진실** — 프로토타입 (`design/prototype/{화면}.jsx`) 이 시각·동작의 단일 진실. md 는 구조·상태·연결 담당

### 수익모델

| 단계 | 시점 | 내용 |
|---|---|---|
| V1 (MVP) | 지금 | 구독·광고 없음 (DB 자리만) |
| V2 | 출시 1-2개월 후 | 구독만 (월 ₩{MONTHLY} / 연 ₩{YEARLY}) |
| V3 | 출시 2-3개월 후 | 광고 추가 (배너 + Rewarded) |

무료 한도 (V2) — {한도}.

### 개발 운영

- **환경 분리** — dev / prod flavor
- **테스트** — Repository / BLoC 단위 테스트만
- **Git** — chore/feature/fix/refactor/epic + sub-work + Conventional Commits (룰 #10)
- **베타 배포** — TestFlight (iOS prod) + Firebase App Distribution (Android)
- **에러 모니터링** — Day 1 부터 Sentry

---

## 10. 개발 철학

공사 짓듯 단계적으로 쌓아 올린다.

```
기반 → 뼈대 → 화면 → 연결 → 기능 → 고도화
```

- **Week 1** — 기반 구축 (환경 · 디자인 시스템 · 폴더 구조 · DB · Realtime 검증)
- **Week 2** — 화면 완성 (mock 데이터로 모든 UI + 풀 플로우)
- **Week 3** — 기능 탑재 + 베타 출시 (Realtime · 매칭 · 푸시 · 딥링크)

일정 운영 원칙 —

- 일별 일정은 **가이드** (유연하게 운영, 초과 시 다음 날로)
- 주별 마일스톤은 **약속** (절대 미루지 않음)
- **일정보다 품질 우선**

코드 원칙 —

- 코딩 컨벤션 일관성 유지
- 디자인 토큰 (변수) 적극 활용
- 컴포넌트 재사용 우선 (중복 코드 방지)
- 클린 아키텍처 (feature-first · 기능/화면별 독립)

---

## 11. 핵심 개발 원칙 (앱별 자리)

앱 정체성에서 파생되는 원칙 목록. 룰 #9 (앱별 도메인 룰) 의 근거. 코드·카피·기능 결정의 최종 판단 기준.

1. {원칙 1 · 예 · "사용자를 오래 붙잡지 말고, 실제 행동으로 보내라"}
2. {원칙 2}
3. {원칙 3}
4. {원칙 4}
5. {원칙 5}

**작성 팁.** 5-7개 권장. 서로 상충 가능 (예: "사용자를 오래 붙잡지 않음" vs "충분한 가치 제공") — 원칙 간 균형이 판단 기준. 앱 특화 표현 (도메인 명사 · 사용자 감정 등) 유지.

---

## 문서 맵

| 문서 | 내용 |
|---|---|
| `00_overview.md` | 이 문서 — 전체 개요 + 결정 요약 |
| `01_tech_stack.md` | 기술 스택 상세 + pubspec.yaml + .env |
| `02_architecture.md` | 폴더 구조 + DB 스키마 + BLoC 패턴 |
| `03_screens/README.md` | 화면 아키텍처 · IA · user flow · 상태 원칙 |
| `03_screens/{화면}.md` | 화면별 개발 handoff 명세 (상태 · 인터랙션 · 데이터 · BLoC 연결) |
| `04_design_system.md` | 디자인 토큰 + 컴포넌트 |
| `05_roadmap.md` | Week 1-3 로드맵 + 출시 후 1주 |
| `06_revenue_model.md` | V1/V2/V3 수익모델 |
| `07_decisions_log.md` | 결정 변경 이력 |
| `08_dev_ops.md` | Git · 테스트 · 환경 · MCP · 모니터링 |
| `CLAUDE.md` | Claude Code 컨텍스트 (프로젝트 루트) |

---

## 예시 (StartRoom)

> 이 template 을 실제로 채운 예 — StartRoom `docs/00_overview.md`.
> 참고 시 도메인 특화 값 (익명 파트너 매칭 · 시작방 · 10분 세션 등) 은 걷어내고 구조만 참고.
