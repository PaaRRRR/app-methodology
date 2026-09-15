# 04. 디자인 시스템

> 컬러·타이포·간격 토큰과 공통 컴포넌트 (`{PREFIX}*`) 스펙.
> **이 문서의 모든 값은 디자인 프로토타입에서 추출한 실측값이다.** 기획서 초기 메모는 폐기됨. 프로토타입이 진실 (룰 #8).
> D2 (디자인 시스템 구축일) 에 `theme.dart` 로 옮긴다. Week 1 에서 가장 중요한 산출물.

---

## 0. 디자인 철학

> {DESIGN_PHILOSOPHY_ONE_LINER} (예: sharp, focused, productive. NOT meditative or warm.)

- **{원칙_1}** — {설명}
- **{원칙_2}** — {설명}
- **{원칙_3}** — {설명}

---

## 1. 확정 결정 (V1)

| 항목 | 값 | 비고 |
|---|---|---|
| 팔레트 | {DARK_LIGHT} + {ACCENT_COLOR} | accent 단일 유지 |
| 모드 | 다크 + 라이트 (사용자 선택) | 기본 = 시스템. 3택 |
| 폰트 | {FONT_FAMILY} | display/body |
| {DOMAIN_KEY_FIXED} | {값} | 관련 결정 |

### 1-1. 브랜드 컬러 근거
{배경색 · accent · fill/ink 페어 · onAccent · accentSoft 근거}

### 1-2. 워드마크
{폰트 · 색 · 이유}

### 1-3. 로고 아이콘
{경로 · 형태 · 이유}

### 1-4. 앱 아이콘 · 스플래시 (v{VERSION} 확정)
{iOS 사이즈 · Android 밀도 · 스토어 자산 · 앱 내부 accent 와 분리 여부}

---

## 2. 컬러 토큰 (Palette ThemeExtension)

```dart
// stack/flutter/rule-enforcement/design-tokens.md 코드 예시 참조
class Palette extends ThemeExtension<Palette> {
  static const Palette dark = Palette(...);
  static const Palette light = Palette(...);
  // context.colors.accent, .bg, .text ...
}
```

**Fill/Ink 페어 분리.** 라이트 모드에서 accent fill 을 텍스트/border 로 그대로 쓰면 WCAG AA 실패 → fill 과 ink 분리.

**다크 고정 예외.** {목록} (예: 공유 카드 · QR 스캔 chrome · Dev 스크린).

---

## 3. 타이포 (AppTextStyles)

```dart
// display · headline · title · body · label · caption ...
```

---

## 4. 간격 · Radius · Motion 토큰

```dart
class AppSpacing { static const xs, sm, md, lg, xl, xxl; }
class AppRadius { static const sm, md, lg, pill; }
// Motion 토큰 — fast/standard/slow/pop/pulse (Duration + Curve)
```

---

## 5. 공통 컴포넌트 (`{PREFIX}*`)

> 목록 · 각 위젯 상세는 `stack/flutter/rule-enforcement/sr-widgets.md` 참조.

**필수 기본 세트 (first sitting 에 만들기)**
- `{PREFIX}Button` (5종 × 3크기)
- `{PREFIX}Card` (lift/padding/radius)
- `{PREFIX}Input`
- `{PREFIX}IconButton`
- `{PREFIX}Tag`
- `{PREFIX}ScreenHeader` (44px · left/title/right slot)
- `{PREFIX}SheetHandle`
- `{PREFIX}ScreenSkeleton` · `{PREFIX}LoadingView` · `{PREFIX}EmptyView` · `{PREFIX}ErrorView`

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/04_design_system.md`. 라임 accent · 다크/라이트 이중 모드 · Fill/Ink 페어 · 다크 고정 예외 4자리 (공유 카드 · QR · 아바타 helper · Dev 스크린) 등 상세.
