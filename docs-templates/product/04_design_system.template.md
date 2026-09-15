# 04. 디자인 시스템

> 컬러·타이포·간격 토큰과 공통 컴포넌트 (`{PREFIX}*`) 스펙.
> **이 문서의 모든 값은 디자인 프로토타입에서 추출한 실측값이다.** 프로토타입이 진실 (룰 #8).
> D2 (디자인 시스템 구축일) 에 `theme.dart` 로 옮긴다. Week 1 에서 가장 중요한 산출물.

---

## 0. 디자인 철학

> {DESIGN_PHILOSOPHY_ONE_LINER} (예: sharp, focused, productive. NOT meditative or warm.)

**공통 원칙 (앱 무관 · 대부분 그대로).**
- **중립 섀시 + 기능색 분리.** 배경·표면은 완전 중립 (색조 없음). 색은 기능이 있는 곳 (버튼 · 활성 상태 · 강조 요소) 에만
- **Accent 는 갈아끼우는 변수.** 팔레트 여러 종이 같은 중립 섀시를 공유하고 accent 만 다름. 브랜드 색 변경 시 나머지 흔들리지 않음
- **여백 넉넉하게 · 말은 직접적으로.** 타이포 정제·넉넉하게 · 카피는 활기차고 직접적
- **띠·border 는 또렷한 선.** 흐릿한 후광 X

---

## 1. 확정 결정 (V1) · 앱별 값

| 항목 | 값 | 비고 |
|---|---|---|
| 팔레트 | 다크 + 라이트 이중 + accent `{ACCENT_HEX}` | accent 단일 유지 |
| 모드 | 다크 + 라이트 (사용자 선택) | 기본 = 시스템. 3택 |
| 폰트 | {FONT_FAMILY} | display/body 동일 (별개 폰트는 V2+) |
| {DOMAIN_KEY_FIXED} | {값} | 관련 결정 |

### 1-1. 배경색 (다크 + 라이트 이중)

**공통 원칙 · 앱 무관.**
- 배경은 뉴트럴 (색조 없음). 무드는 accent 가 결정
- 다크 + 라이트 이중 · MaterialApp.themeMode 표준 스위치

**앱별 값.**
- 다크 bg — `{DARK_BG_HEX}` (예: `#0a0a0b`)
- 라이트 bg — `{LIGHT_BG_HEX}` (예: `#f2f2f0`)

### 1-2. Accent (fill · ink 페어 분리)

**공통 원칙.**
- 라이트 mode 에서 accent fill 을 텍스트/border 에 그대로 쓰면 흰 배경 위 WCAG AA 실패
- **fill (`accent`)** — 배경·CTA 채움. 다크/라이트 동일
- **ink (`accentInk`)** — 텍스트·stroke. 라이트만 dark 톤 down
- **`onAccent`** — accent fill 위 텍스트 · mode 무관 니어블랙 고정 (예측 가능성 우선)
- **`accentSoft`** — 카드 활성 배경 tint. 라이트에서 더 진하게 (배경이 흰색이라 옅으면 안 보임)

**앱별 값.**
- `accent` — `{ACCENT_FILL_HEX}` (다크/라이트 동일)
- `accentInk` — 다크 `{ACCENT_INK_DARK}` / 라이트 `{ACCENT_INK_LIGHT}`
- `onAccent` — `{ON_ACCENT_HEX}` (니어블랙 · 고정)
- `accentSoft` — 다크 `{ACCENT_SOFT_DARK}` / 라이트 `{ACCENT_SOFT_LIGHT}`

### 1-3. 다크 고정 예외

**공통 원칙 — mode 반응 안 하고 항상 다크로 렌더링해야 하는 자리.**
- **공유 카드** (SNS 외부 · 브랜드 일관성 · 라이트 배경 위 accent fill 눈부심 회피)
- **QR 스캔 chrome** (카메라 UI 관례 · Material 컨벤션)
- **Dev 스크린** (내부 도구 · hex literal 하드코딩 유지)
- **아바타 helper** (색 계산 로직에서 정적 팔레트 참조)

**앱별.** 필요 자리에 `Palette.dark.*` 정적 접근으로 fix.

### 1-4. 워드마크

**공통 원칙 — 워드마크 색은 텍스트 토큰 재사용** (`context.colors.text`). 별도 색 도입 X · 이중 모드에서 자동 대비.

**앱별.**
- 폰트 — {FONT_FOR_WORDMARK} (예: SF Rounded Heavy)
- 이유 — {브랜드 정체성 + 폰트 근거}

### 1-5. 로고 아이콘

**공통 원칙.**
- 어수선하지 않지만 너무 단순하지 않게 · 유니크함 · 부드러운 느낌 지향
- 4방향 or 8방향 star · burst 형태 참조
- 베지어 곡선 포인트로 부드러운 흐름

**앱별.**
- 파일 — `assets/icons/{name}.svg` (UI 24px) · `assets/branding/app_icon.svg` (아이콘 1024px)
- 색 · 형태 이유 — {근거}

### 1-6. 앱 아이콘 · 스플래시

**공통 원칙 · 반드시 유의.**
- **앱 내부 accent 와 앱 아이콘 색은 의도적 분리 가능.** 아이콘 = 스토어 진열대 impact 최우선 (한 단계 밝게 등) · 앱 내부 = 장시간 피로 없어야 함. 무조건 sync X
- **스플래시 bg 와 아이콘 bg 는 다를 수 있음.** 아이콘 = spark 대비 최적화 · 스플래시 = 앱 chrome 정합. OS 애니메이션이 두 프레임 직접 비교 회피

**앱별 값.**
- 아이콘 bg — `{ICON_BG_HEX}`
- 스플래시 bg — `{SPLASH_BG_HEX}`
- iOS AppIcon 18 사이즈 · Android mipmap/adaptive 5 밀도 (자동 생성 · `flutter_launcher_icons`)

---

## 2. 컬러 토큰 (Palette ThemeExtension)

`Palette extends ThemeExtension<Palette>` — 다크/라이트 두 벌 · `context.colors` extension.

상세 코드 skeleton — `stack/flutter/rule-enforcement/design-tokens.md`.

---

## 3. 타이포 (AppTextStyles)

**공통 원칙.**
- display · headline · title · body · label · caption 6단계 baseline
- 컬러는 사용처에서 `.copyWith(color: context.colors.text)` 주입

**앱별 값 (프로토타입 실측 반영).**

| 스타일 | fontSize | fontWeight | height |
|---|---|---|---|
| display | {SIZE} | {WEIGHT} | {HEIGHT} |
| headline | {SIZE} | {WEIGHT} | {HEIGHT} |
| title | {SIZE} | {WEIGHT} | {HEIGHT} |
| body | {SIZE} | {WEIGHT} | {HEIGHT} |
| label | {SIZE} | {WEIGHT} | {HEIGHT} |
| caption | {SIZE} | {WEIGHT} | {HEIGHT} |

---

## 4. 간격 · Radius · Motion 토큰

**공통 baseline.**

```dart
class AppSpacing { xs=4, sm=8, md=12, lg=16, xl=24, xxl=32 }
class AppRadius  { sm=4, md=8, lg=16, pill=999 }
class AppDurations { fast=120ms, standard=240ms, slow=420ms, pop=400ms, pulse=2600ms, band=3000ms }
class AppCurves    { standard=Cubic(0.2,0.8,0.2,1), pop=Cubic(0.34,1.56,0.64,1), fade=easeInOut }
```

프로토타입 실측 값과 차이 나면 프로토타입 우선.

---

## 5. 공통 컴포넌트 (`{PREFIX}*`)

목록 · 각 위젯 상세는 `stack/flutter/rule-enforcement/sr-widgets.md` 참조.

**필수 기본 세트 (D2 first sitting).**
- `{PREFIX}Button` (5종 × 3크기)
- `{PREFIX}Card` (lift/padding/radius)
- `{PREFIX}Input`
- `{PREFIX}IconButton`
- `{PREFIX}Tag`
- `{PREFIX}ScreenHeader` (44px · left/title/right slot)
- `{PREFIX}SheetHandle`
- `{PREFIX}ScreenSkeleton` · `{PREFIX}LoadingView` · `{PREFIX}EmptyView` · `{PREFIX}ErrorView`

---

## 6. 컴포넌트 카탈로그 화면

`/dev/components` 라우트 (dev flavor only) — 모든 위젯 눈으로 확인. D2 산출물 · 프로토타입 대비 시각 검증.

---

## 7. 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/04_design_system.md`.
> 라임 accent (`#b6e63a`) · 다크 bg (`#0a0a0b`) · 라이트 bg (`#f2f2f0`) · 라이트 accentInk (`#4f7317`) tone down · 다크 고정 예외 4자리 (공유 카드 · QR · 아바타 helper · Dev 스크린) · 앱 아이콘 spark 은 `#c6f04a` (앱 내부 lime 보다 한 단계 밝게 · 스토어 impact) · Pretendard 폰트 · SF Rounded Heavy 워드마크 · M-별 (spark 변형) 로고.
