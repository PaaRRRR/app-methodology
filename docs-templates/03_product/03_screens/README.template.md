# 03. 화면 아키텍처 (Screen Architecture)

> **성격 — Phase 3 화면 목록·관계·flow 확정본.** 개별 화면 상세는 `{screen}.md` 파일. 이 README 는 앱 전체 화면 구조 조망 자리.
> Phase 0 (기획 · `docs/planning/plan.md`) 에서 도출한 화면 목록을 Phase 1 (와이어프레임 · `docs/planning/wireframe.md`) · Phase 2 (디자인 · `design/prototype/*.jsx`) 확정 후 여기서 정합화. 개발 착수 (Phase 4) 진입 진실.

---

## 1. 화면 목록

| # | 화면 | 라우트 | 목적 (한 줄) | 핵심 상태 | MVP |
|---|---|---|---|---|---|
| 1 | Onboarding | `/onboarding` | 첫 진입 · 가치 소개 · 권한 요청 | intro · permission · ready | ✅ |
| 2 | Home | `/` | 앱 진입점 · 주요 액션 CTA | empty · active · loading | ✅ |
| 3 | {화면} | `/{route}` | {목적} | {상태들} | {✅/V2} |
| ... | ... | ... | ... | ... | ... |

**MVP 컬럼 규칙 —**
- `✅` — V1 (MVP) 필수
- `V2` — 이연 (`06_revenue_model.md` 참조)

---

## 2. IA · Navigation

화면 간 관계 트리. 어떤 화면에서 어디로 갈 수 있는지 조망.

```
Onboarding (첫 진입 1회)
   ↓
Home ─┬─→ {주 flow 진입 화면}
      │      ↓
      │   {다음 화면} ─→ {완료 화면}
      │
      ├─→ Settings ─→ Notification / Account / About
      ├─→ Notifications
      └─→ {2차 flow 진입 화면}
```

**Tab / Bottom nav 구조 (있으면)** —
- Tab 1 — Home
- Tab 2 — {스크린}
- Tab 3 — {스크린}

**Modal / Sheet** — 별도 라우트 아님 · 진입 화면 위에 얹힘.
- `{sheet_name}` — 진입: {어느 화면 · 어떤 액션}
- `{modal_name}` — 진입: {…}

---

## 3. 주요 User Flow

핵심 시나리오별 화면 순서. Analytics funnel 정의의 밑바탕.

### 3-A. 신규 사용자 온보딩

```
[앱 최초 실행]
   ↓
Onboarding (가치 · 알림 권한 · 마이크 권한 · 준비 완료)
   ↓
Home (empty 상태)
   ↓
{첫 액션 CTA} → {주 flow 진입 화면}
```

### 3-B. 정상 사용 (반복 사용자)

```
[알림 클릭 or 앱 열기]
   ↓
Home (active 상태)
   ↓
{주 flow 진입} → ... → 완료 화면
   ↓
Home (다음 액션 제안)
```

### 3-C. {도메인 특화 flow · 예: 예약 · 초대 · 매칭}

```
{시나리오 순서}
```

### 3-D. 이탈 · 회복 시나리오

- **첫 화면 이탈** — {어떤 상황 · 어디로}
- **flow 중 이탈** — {어떤 상황 · 다음 진입 시 처리}
- **에러 회복** — {네트워크 끊김 · 권한 거부 등}

---

## 4. 상태 정의 원칙 (전 화면 공통)

각 화면 §4 상태 목록의 공통 규칙. 화면별 재정의 안 해도 되는 baseline.

| 상태 | 정의 | UI 원칙 |
|---|---|---|
| `loading` | 데이터 대기 · 500ms 이상 | Skeleton · Progress indicator (spinner 지양) |
| `empty` | 데이터 없음 · 최초 진입 | 그림/아이콘 + 카피 + 첫 액션 CTA |
| `error` | 요청 실패 · 조건 불충족 | 원인 카피 + 재시도 CTA |
| `content` / `loaded` | 정상 데이터 표시 | 화면 본연 UI |
| `lock` / `disabled` | 조건 불충족 · 권한 없음 | 잠금 UI + 안내 카피 (닫기 CTA 지양 · [x] 만) |

**전환 규칙 —**
- `loading` → `content` / `empty` / `error` 중 하나
- `error` → 재시도 → `loading` → ...
- `content` 에서 다시 refresh 시 → `loading` skip · silent update (있는 데이터 유지)

**도메인 특수 상태** — 위 외 화면 특유 상태 (예: `waiting` · `ready` · `live` · `completed`) 는 개별 `{screen}.md` §4 에서 정의.

---

## 5. 개별 화면 상세

각 화면 명세 — 별도 md 파일 (화면당 1개 · `screen.template.md` 원형 사용).

- [Onboarding](./onboarding.md)
- [Home](./home.md)
- [{화면}](./{screen}.md)
- ...

---

## 6. 관련 결정 이력

화면 아키텍처 · IA · flow 관련 큰 결정.
- {D-XX. 화면 X 통합/분리 결정 · 배경}
- {D-XX. Tab vs Stack navigation 선택}

상세 — `07_decisions_log.md`

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/03_screens/` (16개 화면 · IA · 즉시/예약 flow).
>
> StartRoom Phase 0-1 원본 참고 예시 — `docs-templates/examples/00_planning/startroom-{plan,guide,wireframe}.md`. 신규 앱은 `docs/planning/plan.md` · `docs/planning/wireframe.md` 위치 권장 (git 이력 · 코드 저장소와 함께).
>
> Phase 3 에서 Phase 0-1 원본과 Phase 2 프로토타입이 어떻게 정합화되어 개별 `{screen}.md` 로 확정됐는지 대조 참고.
