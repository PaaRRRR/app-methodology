# amplitude-taxonomy — event catalog

> Amplitude 트래킹 단일 진실. 모든 event / user property / super property 정의 한 곳.
> 배포 후 유용성 극대화 · 개발 시엔 raw event 만 박고 funnel 정의는 대시보드에서.

---

## 컨벤션

- **Naming** — `{screen}__{action}__{object}` (double `__` 구분, 내부 single `_`)
- **Action 4종** — `view` / `click` / `close` / `input`
- **Length** — 최대 40자. 길면 줄여서 (allow_notif, notif_toggle 등)
- **Chars** — 영문 소문자 + 숫자 + underscore
- **Funnel** — Amplitude 대시보드에서 raw event 조합. 트래킹 코드는 raw event 전수

---

## `__view` umbrella 규칙

`__view` = "사용자가 의미 있게 화면/sheet 진입 또는 큰 상태 전환한 시점". 단순 step/phase 진행은 sub-event 만.

| 패턴 | fire 시점 | 적용 화면 |
|---|---|---|
| **mount 1회** | screen/sheet/cubit init 1회 | 대부분 화면 |
| **상태 전환마다** | umbrella 자체가 상태 추적용 | 홈 (state_type) · 다단계 UI |

### close → 재진입 시 재 fire 패턴

같은 session 안 `__close` 후 재진입 → 새 cubit → **mount `__view` + 진행된 sub-event 모두 재 fire**. 이건 raw event 진실 (사용자 이탈/재진입 신호).

분석 시 처리 —
- **총 진입 횟수** — 그대로 카운트
- **Unique 진입** — Amplitude chart "Unique by {id}" 옵션 또는 funnel `Uniques` mode
- **Funnel 이탈** — `Uniques per User` mode

---

## Cleanup 원칙

1. **상태 분기 = property** — `home__view__empty/reserved` 같은 상태 분기는 view 1개 + `state_type` property 로 통합
2. **애니메이션 = drop** — UI 애니메이션 drop
3. **정적 컨텐츠 = drop** — 자동 노출 정적 컨텐츠 drop · interaction object 만
4. **반복 slot = property** — `{screen}__input__task_0/1/2` → `{screen}__input__task` + `slot_index`
5. **키보드 이벤트 = drop** — 매 char drop · 입력 완료 (debounce 500ms) 시 1회
6. **모달/시트 = 별도 screen** — sheet 도 screen 취급
7. **Transit phase = drop** — 1초 이내 자동 transit drop · 사용자 의사결정 X
8. **Duplicate view = drop** — state property 와 중복 시 drop
9. **App lifecycle = drop** — Amplitude SDK 자동 session 분석 cover

---

## Table 1 — Screens

| screen | 한글 | 소스 | Bloc/Cubit |
|---|---|---|---|
| {screen_1} | {한글} | `features/{screen_1}/` | {Cubit_1} |
| ... | ... | ... | ... |

dev / debug 화면 — taxonomy 대상 X (flavor=prod 만 트래킹).

---

## Table 2 — Events

포맷 — `{screen}__{action}__{object}` (한 행 = 한 event).

| event | fire 시점 | property |
|---|---|---|
| `app_opened` | 앱 열림 (SDK 자동) | — |
| `{screen}__view` | screen mount | `state_type` (해당 시) |
| `{screen}__click__{object}` | interaction | `{context_property}` |
| `{screen}__close` | X · 뒤로가기 | — |
| ... | ... | ... |

---

## Table 3 — User Properties

- `user_id` — 익명 UUID (자동)
- `platform` — iOS / Android
- `install_date` — 최초 설치일
- `{도메인 property}` — 예: 완주 세션 수 · 연속 일수

---

## Table 4 — Super Properties (모든 event 자동 첨부)

- `app_version`
- `build_number`
- `flavor` — dev / prod
- `theme_mode` — dark / light / system

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/work/amplitude-taxonomy/catalog.md`. 16개 화면 · 143개 event · cleanup 9원칙 · close→재진입 재 fire 패턴 상세.
