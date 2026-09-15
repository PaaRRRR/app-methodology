# Amplitude Dashboard Spec — Beta Health

> Beta 단계 앱 건강도 대시보드 spec.
> 실 대시보드 = Amplitude 안. Chart 개별 정의는 Amplitude description 이 진실. 이 파일 = **layout · 원칙 · 결정 이력**.

---

## 목적 · 가설

**앱 가설.** "{CORE_HYPOTHESIS}"

**대시보드 목적.** 이 가설이 실 사용자에게 유효한가를 **6~7개 지표로 검증**.

**포함/미포함 원칙.**
- **포함** — Volume (DAU) · {가설 검증 핵심} · Activation (funnel) · Retention
- **미포함** — 초대/알림 세부 · UX 지표 · Growth · 안정성 (Sentry 자리)

**"지표는 적을수록 잘하는 것"** 원칙 — Beta 상한 = 6~7개. 이 이상은 안 봄.

---

## Layout — 4 rows

```
Row 1 (Volume):
  [ DAU ]                                    (wide · 헤드라인)

Row 2 (가설 검증 핵심):
  [ 핵심 mechanic 지표 ] [ 핵심 mechanic funnel ]

Row 3 (Activation Funnels):
  [ Onboarding Funnel ] [ 핵심 flow Complete Funnel ]

Row 4 (Retention):
  [ Retention — App Open ] [ Retention — Session Complete ]
```

**Row 배치 원칙.**
- Row 1 = DAU wide (전체 볼륨 · 헤드라인)
- Row 2 = 가설 검증 강조 (시선 유도)
- Row 3 = Funnel 2개 (이탈 지점 진단)
- Row 4 = Retention 2개 나란히 (App Open vs Session Complete 격차 = 핵심 인사이트)

---

## Chart Spec (6~7개)

### 1. DAU
- Type — Segmentation
- Event — `app_opened`
- Measured — Uniques · Daily · Last 30 days
- Style — Line chart
- Benchmark — Beta 1주차 {A} DAU → 4주차 {B} DAU

### 2. {가설 검증 핵심 지표}
- Type — Segmentation
- Event — `{핵심 event}`
- Measured — Uniques · Daily
- Group by — `{key property}`
- Benchmark — {목표 값}

### 3. Onboarding Funnel
- Type — Funnel · N steps
- Steps —
  1. `onboarding__view`
  2. `onboarding__view__noti_permission`
  3. `onboarding__view__ready`
  4. `{첫 핵심 flow event}`
  5. `{완주 event}`
- Measured — Conversion (Uniques per User) · Conversion window 7 days
- Benchmark — 최종 15%+ · 각 단계 60%+

### 4. {핵심 flow Complete Funnel}
- Steps 정의 · Benchmark

### 5-6. Retention (2개)
- Type — Retention chart
- Event 1 — `app_opened` (재방문 하한)
- Event 2 — `{핵심 완주 event}` (진짜 재방문)
- Measured — D1 / D7 / D30
- 두 chart 격차 = 핵심 인사이트

---

## 관찰 실행 · 판단 트리는 `../beta-observations/protocol.template.md`

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/work/amplitude-taxonomy/dashboard-spec.md`. 7개 chart (DAU · 실 파트너 매칭 비율 · Partner Type Complete Funnel · Onboarding · Session Complete · Retention App Open · Retention Session Complete).
