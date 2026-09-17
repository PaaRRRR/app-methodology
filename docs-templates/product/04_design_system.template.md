# 04. 디자인 시스템

> 컬러·타이포·간격 토큰과 공통 컴포넌트 (`{PREFIX}*`) 스펙.
> **이 문서의 모든 값은 디자인 프로토타입에서 추출한 실측값이다.** 프로토타입이 진실 (룰 #8).
> D2 (디자인 시스템 구축일) 에 `theme.dart` 로 옮긴다. Week 1 에서 가장 중요한 산출물.

---

## 0. 디자인 철학 (앱별)

> {DESIGN_PHILOSOPHY_ONE_LINER}

각 앱의 정체성 · 사용자 감정 · 브랜드 톤을 한 줄로 요약. 예 — "sharp, focused, productive" · "warm, meditative, quiet" · "playful, bold, spontaneous". 이 한 줄이 아래 색·타이포·간격·컴포넌트 결정의 기준.

---

## 1. 확정 결정 (V1) · 앱별 값

| 항목 | 값 | 비고 |
|---|---|---|
| 팔레트 | 다크 + 라이트 이중 + accent `{ACCENT_HEX}` | accent 단일 유지 |
| 모드 | 다크 + 라이트 (사용자 선택) | 기본 = 시스템. 3택 |
| 폰트 | {FONT_FAMILY} | display/body 동일 (별개 폰트는 V2+) |
| {DOMAIN_KEY_FIXED} | {값} | 관련 결정 |

### 1-1. 배경 · 이중 모드

**공통 원칙 (앱 무관).**
- 다크 + 라이트 이중 · `MaterialApp.themeMode` 표준 스위치. 사용자 선택권 (다크/라이트/시스템 3택)
- 배경 색조 결정은 앱 정체성 (§0) 에 따름 — 뉴트럴 · warm · cool 등 자유

**앱별 값.**
- 다크 bg — `{DARK_BG_HEX}`
- 라이트 bg — `{LIGHT_BG_HEX}`

### 1-2. Accent (fill · ink 페어 분리)

**공통 원칙 (WCAG AA).**
- 라이트 mode 에서 accent fill 을 텍스트/border 에 그대로 쓰면 흰 배경 위 WCAG AA 실패
- **fill (`accent`)** — 배경·CTA 채움. 다크/라이트 동일
- **ink (`accentInk`)** — 텍스트·stroke. 라이트만 dark 톤 down
- **`onAccent`** — accent fill 위 텍스트 · mode 무관 고정 (예측 가능성 우선)
- **`accentSoft`** — 카드 활성 배경 tint. 라이트에서 더 진하게 (배경 흰색이라 옅으면 안 보임)

**앱별 값.**
- `accent` — `{ACCENT_FILL_HEX}` (다크/라이트 동일)
- `accentInk` — 다크 `{ACCENT_INK_DARK}` / 라이트 `{ACCENT_INK_LIGHT}`
- `onAccent` — `{ON_ACCENT_HEX}` (mode 무관 고정)
- `accentSoft` — 다크 `{ACCENT_SOFT_DARK}` / 라이트 `{ACCENT_SOFT_LIGHT}`

### 1-3. 다크 고정 예외 (최소화 원칙)

**공통 원칙.**
- **이중 모드 표준이 default · 예외는 정말 필요한 자리만.** 세션 몰입 화면 · 콘텐츠 화면 등을 "몰입 = 다크 강제" 로 고정하지 X — 사용자 mode 선택권 침해
- 예외 자리 판단 기준 — (1) SNS 외부 오브젝트 (브랜드 일관성) · (2) OS/Material 컨벤션 (카메라 chrome 등) · (3) 내부 dev 도구

**앱별.** 필요 자리에 `Palette.dark.*` 정적 접근으로 fix. 예시 자리는 §8 참조.

### 1-4. 워드마크

**공통 원칙 — 워드마크 색은 텍스트 토큰 재사용** (`context.colors.text`). 별도 색 도입 X · 이중 모드에서 자동 대비.

**앱별.**
- 폰트 — {FONT_FOR_WORDMARK}
- 이유 — {브랜드 정체성 + 폰트 근거}

### 1-5. 앱 아이콘 · 스플래시

**공통 원칙 · 반드시 유의.**
- **앱 내부 accent 와 앱 아이콘 색은 의도적 분리 가능.** 아이콘 = 스토어 진열대 impact 최우선 (한 단계 밝게 등) · 앱 내부 = 장시간 피로 없어야 함. 무조건 sync X
- **스플래시 bg 와 아이콘 bg 는 다를 수 있음.** 아이콘 = 로고 대비 최적화 · 스플래시 = 앱 chrome 정합. OS 애니메이션이 두 프레임 직접 비교 회피

**앱별 값.**
- 아이콘 bg — `{ICON_BG_HEX}`
- 스플래시 bg — `{SPLASH_BG_HEX}`
- iOS AppIcon 18 사이즈 · Android mipmap/adaptive 5 밀도 (자동 생성 · `flutter_launcher_icons`)

**tip — Android `colors.xml` 두 값 명확 구분.** 이름 겹치지만 다른 요소 지시:
- `ic_launcher_background` = **아이콘** bg (`{ICON_BG_HEX}`)
- `launch_background` = **스플래시** bg (`{SPLASH_BG_HEX}`)

---

## 2. 컬러 토큰 (Palette ThemeExtension)

`Palette extends ThemeExtension<Palette>` — 다크/라이트 두 벌 · `context.colors` extension.

상세 코드 skeleton — `/stack/flutter/rule-enforcement/design-tokens.md`.

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

### 4-A. 애니메이션 우선순위

프로토타입 keyframes 를 Flutter (`AnimationController` · `TweenAnimationBuilder` · implicit 애니메이션) 로 옮길 때 **전부 구현할 필요 없음** — 우선순위 분류.

| 우선순위 | 성격 | 예 |
|---|---|---|
| ★ **필수** | 핵심 와우 · 신뢰 신호 | LIVE pulse · 카운트다운 flip-in · 완주 pop · 스티커 pop |
| ☆ **권장** | 전환 부드러움 · 감각 강화 | rise (등장) · ripple · band-pulse |
| ✕ **V2+** | 부가 · 대체 가능 | shimmer · shake · 음성 wave |

**햅틱은 애니메이션과 짝** — Ready 누름 · 리액션 전송 · 완주 도달 순간 가벼운 햅틱.

**시스템 무음 모드 존중** — 사운드 자동 OFF · 햅틱은 유지 (원칙).

---

## 5. 공통 컴포넌트 (`{PREFIX}*`)

목록 · 각 위젯 상세는 `/stack/flutter/rule-enforcement/sr-widgets.md` 참조.

**필수 기본 세트 (D2 first sitting).**
- `{PREFIX}Button` (5종 × 3크기)
- `{PREFIX}Card` (lift/padding/radius)
- `{PREFIX}Input`
- `{PREFIX}IconButton`
- `{PREFIX}Tag`
- `{PREFIX}ScreenHeader` (44px · left/title/right slot)
- `{PREFIX}SheetHandle`
- `{PREFIX}ScreenSkeleton` · `{PREFIX}LoadingView` · `{PREFIX}EmptyView` · `{PREFIX}ErrorView`

### 5-A. 사용률 정책

코드 리뷰 시 Material 원시 위젯 대신 `{PREFIX}*` 우선 사용 확인.

- **`{PREFIX}Button` 표준** — Material `ElevatedButton`/`TextButton` 대신
- **`{PREFIX}IconButton` 표준** — Material `IconButton` 대신
- **`{PREFIX}Card`** — `Container + BoxDecoration + border` 중복 자리는 통일
- **`{PREFIX}Input`** — 단순 `TextField` 는 직접 OK · 복잡 (placeholder/border/style) 통일 자리만 `{PREFIX}Input`
- **닫기 아이콘 통일** — 항상 `{PREFIX}Icons.x` (Phosphor). **`Icons.close` (Material) 직접 사용 금지** — 브랜딩 정합

### 5-B. 헤더 leadingIcon 규칙

사용자 동일 UX 신호. 닫기/뒤로 자리 정합.

| 자리 | leadingIcon | 예 |
|---|---|---|
| **하위 화면** | `{PREFIX}Icons.caretLeft` (뒤로) | Create · Room · Finish · Join |
| **모달 시트 / 결과 화면** | `{PREFIX}Icons.x` (닫기) | 바텀시트 · 완료 화면 |
| **루트** | leading `null` | Home · Settings · Stats · Onboarding |

### 5-C. AppBar vs {PREFIX}ScreenHeader 패턴

프로토타입 정합 = **{PREFIX}ScreenHeader 통일** (프로토타입 헤더 = body 안 첫 줄 · AppBar 개념 X).

- **AppBar** — Material 표준 · 자동 SafeArea · status bar 처리. leading icon 이 status bar + AppBar 영역 아래로 나옴
- **{PREFIX}ScreenHeader** — body 안 첫 줄. leading icon 이 body 상단에서 시작

**혼합 사용 시 사용자 시점에서 leading icon 절대 거리 차이 발생.** V1 안전 우선 혼합 유지도 가능 · V2 통일 권장.

**좌우 padding 정합** — AppBar leading/actions padding + {PREFIX}ScreenHeader padding 동일 값 (예: 4px).

### 5-D. 컴포넌트 카탈로그 화면

`/dev/components` 라우트 (dev flavor only) — 모든 위젯 눈으로 확인. D2 산출물 · 프로토타입 대비 시각 검증.

---

## 6. 토큰 사용 규칙 (CLAUDE.md 연동)

이 문서의 실행 계약. 신규 코드 작성 · 코드 리뷰 시 정합.

1. **색 · 폰트 · 간격 · 라운드 · 모션은 반드시 토큰 참조** (룰 #2). `Color(0xFF...)` · 매직 넘버를 위젯에 직접 쓰지 X → `Palette` (context.colors) · `AppTextStyles` · `AppSpacing` · `AppRadius` · `AppDurations` · `AppCurves`
2. **accent 블록 4개 (`accent` · `accentInk` · `accentSoft` · `onAccent`) 만 갈아끼우면 브랜드 색 전체 교체.** 중립 섀시 · 기능색은 손대지 X. 브랜드 리브랜딩 시 blast radius 최소
3. **컴포넌트는 `{PREFIX}*` 재사용 우선** (룰 #1). 화면 전용 위젯은 해당 feature 의 `widgets/` 폴더에
4. **이중 모드 (다크/라이트) 표준** — `context.colors.*` 로 자동 분기. 다크 고정 예외는 §1-3 자리만 · 정말 필요할 때만 `Palette.dark.*` 정적 접근
5. **매직넘버 예외** — 토큰에 없는 값 · 1회 사용 · prototype 직접 색 등은 widget-local 상수 허용 (룰 #2 유연화 자리)

**검증 팁 —** 코드 리뷰 시 `grep -rnF "Color(0xFF" lib/` · `grep -rnF "Icons.close" lib/` 등으로 위반 자리 발견.

---

## 7. Copy Policy

앱 안 카피의 자리별 톤 · 마침표 · 호칭 정책. 신규 카피 작성 시 정합.

### 7-A. 자리 × 톤 매트릭스 (baseline · 앱별 조정)

각 앱은 자기 화면 구조에 맞게 자리 추가/삭제/톤 조정.

| 자리 | 톤 | 예시 |
|---|---|---|
| 화면 · 섹션 헤더 | 정중 | "알림" · "통계" · "설정" |
| 빈 상태 title | 친근 | "아직 알림이 없어요." |
| 빈 상태 sub | 격려 + 정보 | "{조건}을 하면 알림을 받을 수 있어요." |
| 카드 headline | 격려 | "같이 시작해요." |
| 카드 sub | 정보 | "{조건}이 도착하면 알려드릴게요." |
| 시스템 메시지 (sys) | 정보 + 격려 | "모두 준비됐어요. 곧 시작할게요." |
| 상태 안내 (status pill) | 정보 | "{조건}이 오고 있어요." |
| CTA 버튼 라벨 | 명령형 | "준비 완료" · "마치기" · "만들기" |
| placeholder (hint) | 정보 | "메시지 입력…" |
| 빠른 칩 · quick reply | 친근 + 격려 | "좋아요, 가봅시다" |
| 알림 title | 정보 (조건 발생) | "{누가}가 도착했어요" |
| Onboarding 헤드라인 | 격려 | "{한 줄 격려 헤드라인}" |

### 7-B. 마침표 룰 (한국어 baseline)

| 사용 | 자리 |
|---|---|
| **마침표 O** | 시스템 메시지 · 상태 안내 · 빈 상태 title+sub · 카드 headline+sub · SnackBar · 카드 desc (한 문장 이상) |
| **마침표 X** | CTA 버튼 라벨 · 빠른 칩 · placeholder (`…` · `(선택)` 자체 종결) · 메트릭 라벨 · 시간 그룹 라벨 |

### 7-C. 호칭 룰 (한국어 baseline · 앱별 조정)

- **인앱 sys / live phase** — `fallback` "{역할명}" (-님 없음). nickname 도착 시 그대로. 친근·짧음
- **알림 (push + in-app feed)** — "{name} 님" (-님 O). 정중·외부 톤
- **본인 호칭** — "나" (avatar label) / "내" (소유 · "내 차례예요")

### 7-D. 종결 어미 · 톤 키워드 (앱별)

- **정중 어미** — "-해요" · "-할게요" · "-요" (한국어 baseline)
- **명령형 어미** — "-기" · "-하기" (CTA · 한국어 baseline)
- **격려 키워드 (앱별)** — 브랜드 정체성에 맞는 3-5개 선정. 예 — {키워드_1} · {키워드_2} · {키워드_3}

### 7-E. 알림 카피 형식 (특수 룰)

- 모든 알림 = "{name} 님" 정중 톤 (§7-C)
- body 시점/콘텐츠 구분 = "·" (예: "개발 시작방 · 오전 9:00 · 지우 님과 함께")
- **알림 title/body 마침표 X** (간결 외부 톤). 축하/액션 권유 `!` `?` 는 OK
- 인용 카피 (스티커·회고 등 body) = `"..."` 형식

---

## 8. 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/04_design_system.md`.

**디자인 철학 —** "sharp, focused, productive. NOT meditative or warm." + 중립 섀시 + 여백 넉넉 + 띠 또렷 (StartRoom 브랜드 성격).

**색 · 폰트 —** 라임 accent (`#b6e63a`) · 다크 bg (`#0a0a0b`) · 라이트 bg (`#f2f2f0`) · 라이트 accentInk (`#4f7317`) tone down · `onAccent` = 니어블랙 (`#131806`) 고정 · 아바타 이니셜도 니어블랙 재사용 · Pretendard 폰트 · SF Rounded Heavy 워드마크.

**로고 · 아이콘 —** M-별 (spark 변형 · 4방향 star burst · 베지어 곡선) · 앱 아이콘 spark 은 `#c6f04a` (앱 내부 lime 보다 한 단계 밝게 · 스토어 impact).

**다크 고정 예외 4자리 —** 공유 카드 · QR 스캔 chrome · Dev 스크린 · 아바타 helper.

**Room 도메인 자리 (특화 · 참조용) —**
- **SrBandRing intensity 맵** — 화면별 방 띠 강도 (waiting/session/chatscrum/retro 5단계)
- **스티커 세트 5종** — 회고 완료 시 파트너에 전송 · 세션 중 파트너가 보낸 것 표시

**shareCardHues 6색 (실 값 · SNS 공유 카드 자리) —** 살구 · 앤티크 골드 · 라벤더 · 틸 · 더스티 로즈 · 더스티 블루.

**avatarPalette 8색 (실 값 · WCAG AAA 니어블랙 대비 7:1+) —** 더스티 블루 · 라벤더 · 세이지 · 애프리콧 · 로즈 · 틸 · 앤티크 골드 · 모브.

**격려 키워드 (§7-D 예) —** "괜찮아요" · "함께" · "오늘도" · "다시" · "잘 하셨어요"
