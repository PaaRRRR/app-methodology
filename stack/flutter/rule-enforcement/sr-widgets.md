# sr-widgets — 룰 #1 강제 asset

> 룰 #1 실현. 컴포넌트 재사용 우선. 새 위젯 만들기 전 `lib/shared/widgets/` 확인.
> 기본 위젯 세트 = first sitting (D2) 에 만드는 최소 세트.

---

## 왜 이 룰이 있는가

- **일관성** — 같은 역할 위젯이 화면마다 다르면 UX 훼손
- **유지보수** — 브랜드 변경 시 shared 위젯만 수정
- **속도** — 화면 개발 시 기본 위젯 조합만으로 대부분 가능

**해결.** `{PREFIX}` (예: `Sr`) 접두사 · `lib/shared/widgets/` 안 기본 세트.

---

## 기본 위젯 세트 (first sitting D2 필수)

| 위젯 | 파일 | 역할 |
|---|---|---|
| `{PREFIX}Button` | `sr_button.dart` | 5종 (primary · secondary · ghost · ready · danger) × 3크기 (lg · md · sm) |
| `{PREFIX}Card` | `sr_card.dart` | 기본 카드 (lift/padding/radius 옵션) |
| `{PREFIX}Input` | `sr_input.dart` | 텍스트 입력 (상태별 스타일) |
| `{PREFIX}IconButton` | `sr_icon_button.dart` | 아이콘 버튼 (크기 선택) |
| `{PREFIX}Tag` | `sr_tag.dart` | 태그/칩 (variant) |
| `{PREFIX}ScreenHeader` | `sr_screen_header.dart` | 44px 헤더 · left/title/right slot |
| `{PREFIX}SheetHandle` | `sr_sheet_handle.dart` | 바텀시트 상단 드래그 핸들 |
| `{PREFIX}ScreenSkeleton` | `sr_screen_skeleton.dart` | 로딩 skeleton |
| `{PREFIX}LoadingView` | `sr_loading_view.dart` | 전체 로딩 뷰 |
| `{PREFIX}EmptyView` | `sr_empty_view.dart` | 전체 empty 뷰 |
| `{PREFIX}ErrorView` | `sr_error_view.dart` | 전체 에러 뷰 |

**추가 (필요 시)** — `{PREFIX}Avatar`, `{PREFIX}TimePicker`, `{PREFIX}Icons` (icons 매핑) 등.

---

## SrButton 예시 구조

```dart
enum SrButtonKind { primary, secondary, ghost, ready, danger }
enum SrButtonSize { lg, md, sm }

class SrButton extends StatefulWidget {
  const SrButton({
    super.key,
    required this.label,
    this.kind = SrButtonKind.primary,
    this.size = SrButtonSize.lg,
    this.onPressed,
    this.fullWidth = true,
    this.icon,
  });

  final String label;
  final SrButtonKind kind;
  final SrButtonSize size;
  final VoidCallback? onPressed;
  final bool fullWidth;
  final Widget? icon;

  @override
  State<SrButton> createState() => _SrButtonState();
}

class _SrButtonState extends State<SrButton> {
  bool _pressed = false;

  @override
  Widget build(BuildContext context) {
    final disabled = widget.onPressed == null;
    final kind = _kindSpec(widget.kind, context.colors);  // Palette 참조
    final size = _sizeSpec(widget.size);

    return GestureDetector(
      onTapDown: (_) => setState(() => _pressed = true),
      onTapUp: (_) => setState(() => _pressed = false),
      onTapCancel: () => setState(() => _pressed = false),
      onTap: widget.onPressed,
      child: AnimatedContainer(
        duration: AppDurations.fast,      // Motion 토큰
        curve: AppCurves.standard,
        height: size.height,
        width: widget.fullWidth ? double.infinity : null,
        padding: EdgeInsets.symmetric(horizontal: size.paddingX),
        decoration: BoxDecoration(
          color: _pressed ? kind.background.withOpacity(0.8) : kind.background,
          borderRadius: BorderRadius.circular(size.radius),
        ),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            if (widget.icon != null) ...[
              IconTheme(data: IconThemeData(color: kind.foreground), child: widget.icon!),
              const SizedBox(width: AppSpacing.sm),
            ],
            Text(widget.label, style: size.textStyle.copyWith(color: kind.foreground)),
          ],
        ),
      ),
    );
  }
}
```

**핵심 원칙.**
- 색·크기·radius = 토큰 참조 (`context.colors.*`, `AppSpacing.*`, `AppRadius.*`)
- 상태 (`_pressed` · `disabled`) = 시각 반응 명시
- 애니메이션 = Motion 토큰 (`AppDurations.fast`, `AppCurves.standard`)

---

## 컴포넌트 카탈로그 화면 (D2 산출물)

```dart
// lib/features/dev/components_catalog_screen.dart (dev flavor only)
class ComponentsCatalogScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ListView(
        children: [
          _Section('SrButton'),
          for (final kind in SrButtonKind.values)
            for (final size in SrButtonSize.values)
              SrButton(label: '$kind $size', kind: kind, size: size, onPressed: () {}),
          _Section('SrCard'),
          SrCard(child: Text('...')),
          // ...
        ],
      ),
    );
  }
}
```

카탈로그 = 모든 위젯 눈으로 확인 · 프로토타입 대비 시각 검증 자리.

---

## 검증 (룰 강제)

- 새 화면 개발 시 `lib/shared/widgets/` 안 위젯 먼저 검색
- `Container` 나 raw `Material` 위젯 반복 사용 발견 시 `{PREFIX}` 위젯으로 뽑기
- Palette · Spacing · Radius · Motion 토큰 100% 참조 (하드코딩 X)

---

## 관련 룰

- 룰 #1 — 컴포넌트 재사용 (개념)
- 룰 #2 — 하드코딩 금지 (`design-tokens.md`)
- 룰 #8 — 프로토타입 실측값 (시각 정의 근거)
