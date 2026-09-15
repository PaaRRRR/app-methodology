# docs-work-naming — 회차 문서 명명 규칙

> `docs/work/{category}/{topic}-{yyyy-mm}/` 규칙.
> 브랜치 이름과 문서 폴더 이름 일대일.

---

## 1. 규칙

```
docs/work/{category}/{topic}-{yyyy-mm}/
```

- `category` — `feat` · `fix` · `chore` · `refactor` · `epic` · `ops` (git flow prefix 와 일대일)
- `topic` — kebab-case 짧게, 회차 주제만 (도메인·서수 명확히)
- `yyyy-mm` — 진입 월. 같은 topic 반복 시 `-{yyyy-mm-suffix}` (예: `matching-pool-rpc-2026-08-b`)

---

## 2. 예시

| 브랜치 | 문서 폴더 |
|---|---|
| `chore/env-setup-2026-06` | `docs/work/chore/env-setup-2026-06/` |
| `feature/d3-routing-2026-06` | `docs/work/feature/d3-routing-2026-06/` |
| `feature/d15-realtime-2026-07` | `docs/work/feature/d15-realtime-2026-07/` |
| `fix/room-timer-sync-2026-08` | `docs/work/fix/room-timer-sync-2026-08/` |
| `chore/week2-closeout-2026-07` | `docs/work/chore/week2-closeout-2026-07/` |
| `epic/brand-color-2026-09` | `docs/work/epic/brand-color-2026-09/` (umbrella) |
| `feature/brand-color-dual-mode-2026-09` | `docs/work/feature/brand-color-dual-mode-2026-09/` (sub) |

---

## 3. 폴더 안 파일 관행

**필수** — `plan.md`

**Optional (필요 시)**
- `checklist.md` — 진행 tracking 체크박스
- `context-notes.md` — 결정 배경 · 회차 도중 발견한 것. plan.md 안에 있으면 skip
- `pending.md` — 미완 · 후속 회차로 넘길 것
- 회차 산출물 — `inventory.md`, `spec.md`, `handoff/`, `logs/` 등 (성격에 맞게)

---

## 4. Day / Week 기반 회차

Day 번호는 topic 안에 살리고 카테고리 prefix 만 붙임.

- 초기 setup — `chore/d1-init-YYYY-MM`, `chore/d2-supabase-schema-YYYY-MM`
- 화면 만들기 — `feature/d3-routing-YYYY-MM`, `feature/d8-home-YYYY-MM`
- 기능 탑재 — `feature/d15-realtime-YYYY-MM`, `feature/d17-fcm-YYYY-MM`
- 마감 정리 — `chore/week2-closeout-YYYY-MM`

`05_roadmap.md` 의 Day/Week 는 주제 표기, 명명 규칙의 category 는 성격 표기. 둘이 충돌 없이 공존.

---

## 5. 과거 회차 rename 금지

- 과거 완료 회차 rename 은 blast radius 크고 git blame 깨짐
- 룰 #3 (surgical changes) — 오늘 이후 신규 회차만 신규 규칙 적용
- 과거 회차는 그대로 history 로 유지

---

## 6. 저장소별 판단

- **앱 저장소** — 이 규칙 그대로 적용
- **methodology 저장소** — 초기 단계는 회차 개념 최소 (main 직접 작업). 문서·회차 커지면 도입
- **논리 프로젝트가 여러 저장소 걸치면** — 각 저장소별 명명 · umbrella 문서로 통합
