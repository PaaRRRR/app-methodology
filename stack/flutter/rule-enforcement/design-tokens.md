# design-tokens — 룰 #2 강제 asset

> 룰 #2 실현. 하드코딩 금지. 색·간격·폰트·radius·모션 모두 토큰 참조.

---

## 왜 이 룰이 있는가

- **일관성** — 화면마다 색·간격 값 다르면 브랜드 훼손
- **테마 변경 용이** — 다크/라이트 이중 모드 · accent 교체 시 토큰만 수정
- **접근성** — WCAG AA 준수 위해 fill/ink 페어 분리 등 체계 필요
- **자동 검증** — 하드코딩 grep 으로 회귀 방지 가능

**해결.** `AppColors` · `AppSpacing` · `AppTextStyles` · `AppRadius` · `AppDurations` · `AppCurves` 정적 클래스 + `Palette extends ThemeExtension<Palette>` (모드 반응).

---

## 1. `AppSpacing` · `AppRadius` (mode 무관)

```dart
class AppSpacing {
  AppSpacing._();
  static const double xs = 4;
  static const double sm = 8;
  static const double md = 12;
  static const double lg = 16;
  static const double xl = 24;
  static const double xxl = 32;
}

class AppRadius {
  AppRadius._();
  static const double sm = 4;
  static const double md = 8;
  static const double lg = 16;
  static const double pill = 999;
}
```

---

## 2. `AppDurations` · `AppCurves` (Motion 토큰)

```dart
import 'package:flutter/animation.dart';

class AppDurations {
  AppDurations._();
  static const Duration fast = Duration(milliseconds: 120);
  static const Duration standard = Duration(milliseconds: 240);
  static const Duration slow = Duration(milliseconds: 420);
  static const Duration pop = Duration(milliseconds: 400);
  static const Duration pulse = Duration(milliseconds: 2600);
  static const Duration band = Duration(milliseconds: 3000);
}

class AppCurves {
  AppCurves._();
  static const Curve standard = Cubic(0.2, 0.8, 0.2, 1);
  static const Curve pop = Cubic(0.34, 1.56, 0.64, 1);
  static const Curve fade = Curves.easeInOut;
}
```

---

## 3. `Palette` — mode 반응 (다크/라이트 이중)

```dart
class Palette extends ThemeExtension<Palette> {
  const Palette({
    required this.bg,
    required this.bgDeep,
    required this.surface,
    required this.text,
    required this.textFaint,
    required this.accent,
    required this.accentInk,
    required this.accentSoft,
    required this.onAccent,
    required this.success,
    required this.danger,
    required this.onDanger,
    required this.avatarPalette,
    // ...
  });

  final Color bg, bgDeep, surface;
  final Color text, textFaint;
  final Color accent, accentInk, accentSoft, onAccent;
  final Color success, danger, onDanger;
  final List<Color> avatarPalette;

  static const Palette dark = Palette(
    bg: Color(0xFF0a0a0b),
    bgDeep: Color(0xFF000000),
    surface: Color(0xFF141416),
    text: Color(0xFFf2f2f0),
    textFaint: Color(0xFF808080),
    accent: Color(0xFFb6e63a),
    accentInk: Color(0xFFb6e63a),
    accentSoft: Color(0x24b6e63a),
    onAccent: Color(0xFF131806),
    success: Color(0xFF3ad07e),
    danger: Color(0xFFff5a4d),
    onDanger: Color(0xFF131806),
    avatarPalette: [/* 8색 · 채도 30~45% 대역 */],
  );

  static const Palette light = Palette(
    bg: Color(0xFFf2f2f0),
    bgDeep: Color(0xFFffffff),
    surface: Color(0xFFffffff),
    text: Color(0xFF131806),
    textFaint: Color(0xFF6b7280),
    accent: Color(0xFFb6e63a),           // fill = dark 와 동일
    accentInk: Color(0xFF4f7317),        // ink = tone down (WCAG AA)
    accentSoft: Color(0x38b6e63a),
    onAccent: Color(0xFF131806),
    success: Color(0xFF0b7a50),
    danger: Color(0xFFc42a1d),
    onDanger: Color(0xFFffffff),
    avatarPalette: [/* light 대응 */],
  );

  @override
  Palette copyWith({...}) => Palette(...);

  @override
  Palette lerp(covariant ThemeExtension<Palette>? other, double t) => this;
}

extension PaletteX on BuildContext {
  Palette get colors => Theme.of(this).extension<Palette>()!;
}
```

**사용.** `context.colors.accent` · `context.colors.bg` · ... 위젯에서 `Color(0xFF...)` 하드코딩 금지.

---

## 4. `AppTextStyles`

```dart
class AppTextStyles {
  AppTextStyles._();
  static const TextStyle display = TextStyle(fontSize: 32, fontWeight: FontWeight.w700, height: 1.2);
  static const TextStyle headline = TextStyle(fontSize: 24, fontWeight: FontWeight.w700, height: 1.3);
  static const TextStyle title = TextStyle(fontSize: 18, fontWeight: FontWeight.w600, height: 1.4);
  static const TextStyle body = TextStyle(fontSize: 15, fontWeight: FontWeight.w400, height: 1.5);
  static const TextStyle label = TextStyle(fontSize: 13, fontWeight: FontWeight.w500, height: 1.4);
  static const TextStyle caption = TextStyle(fontSize: 12, fontWeight: FontWeight.w400, height: 1.4);
}
```

컬러는 사용처에서 `.copyWith(color: context.colors.text)` 로 주입.

---

## 5. Fill/Ink 페어 분리 (라이트 mode 대비)

라이트 mode 에서 `accent` fill 을 그대로 텍스트/border 로 쓰면 흰 배경 위 WCAG AA 실패 → fill 과 ink 분리.

- `accent` (fill) — 다크/라이트 동일. 배경·CTA 채움 색
- `accentInk` (ink) — 라이트만 dark 톤 (텍스트·border stroke 용)
- `onAccent` — accent fill 위 텍스트 (mode 무관 고정)

---

## 6. ThemeMode 관리

```dart
// lib/app/theme_cubit.dart
class ThemeCubit extends Cubit<ThemeMode> {
  ThemeCubit(this._prefs) : super(_load(_prefs));
  final SharedPreferences _prefs;

  static ThemeMode _load(SharedPreferences prefs) {
    final s = prefs.getString('theme_mode') ?? 'system';
    return {'dark': ThemeMode.dark, 'light': ThemeMode.light, 'system': ThemeMode.system}[s]!;
  }

  Future<void> setMode(ThemeMode mode) async {
    emit(mode);
    await _prefs.setString('theme_mode', mode.name);
  }
}
```

`MaterialApp.themeMode` = `context.watch<ThemeCubit>().state`.

---

## 7. 검증

```bash
# 하드코딩 색상 검사
grep -rn "Color(0xFF" lib/ | grep -v theme.dart | grep -v _test.dart

# 매직 넘버 padding
grep -rnE "padding: EdgeInsets\.(all|symmetric|only)\([0-9]" lib/
```

**나오면 즉시 토큰으로 교체.**

---

## 8. 관련 룰

- 룰 #2 — 하드코딩 금지 (개념)
- 룰 #8 — 프로토타입 실측값 사용 (토큰 값 근거)
