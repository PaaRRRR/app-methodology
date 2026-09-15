# config-baseline — Flutter 설정 파일 baseline

> `pubspec.yaml` + `analysis_options.yaml` + `flutter_launcher_icons.yaml` 편집 baseline.
> Flutter 생성 시 자동 파일 vs 편집 필요 파일 구분 · 각 편집 사항 명시.

---

## 1. `pubspec.yaml`

**Flutter 생성 시 최소.** 필요 패키지는 앱 도메인·기술 스택에 맞게 추가·크게 편집.

### Baseline dependencies (공통)

```yaml
name: {APP_SLUG}
description: "{ONE_LINER}"
publish_to: 'none'
version: 0.1.0+1

environment:
  sdk: ^3.9.2      # Flutter 안정 버전 · 개발 시작일 기준 확인

dependencies:
  flutter:
    sdk: flutter

  # ─── backend ───────────────────────────────
  supabase_flutter: ^2.12.4

  # ─── state & routing ───────────────────────
  flutter_bloc: ^8.1.6      # 또는 최신 (9.x)
  equatable: ^2.0.5
  go_router: ^14.6.2

  # ─── models ────────────────────────────────
  freezed_annotation: ^2.4.4
  json_annotation: ^4.9.0

  # ─── monitoring · analytics ────────────────
  sentry_flutter: ^8.14.2
  amplitude_flutter: ^4.5.0

  # ─── push · notifications ──────────────────
  firebase_core: ^4.9.0
  firebase_messaging: ^16.2.2
  flutter_local_notifications: ^18.0.1

  # ─── UI · icons ────────────────────────────
  cupertino_icons: ^1.0.8
  phosphor_flutter: ^2.1.0
  flutter_svg: ^2.0.16
  google_fonts: ^6.2.1

  # ─── utils ─────────────────────────────────
  intl: ^0.19.0
  shared_preferences: ^2.5.5
  flutter_secure_storage: ^10.3.1
  package_info_plus: ^8.1.2
  device_info_plus: ^12.4.0
  url_launcher: ^6.3.1
  http: ^1.6.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0

  # codegen for freezed models
  build_runner: ^2.4.13
  freezed: ^2.5.7
  json_serializable: ^6.8.0

  # Play/App Store 출시 · 런처 아이콘 자동 생성
  flutter_launcher_icons: ^0.14.4
```

### 도메인별 추가 (필요 시)

```yaml
  # QR 생성 / 스캔
  qr_flutter: ^4.1.0
  mobile_scanner: ^7.2.0

  # Deep link
  app_links: ^6.3.3

  # 미디어 공유
  share_plus: ^12.0.2
  image_gallery_saver: ^2.0.3

  # 오디오
  audioplayers: ^6.7.1

  # 권한
  permission_handler: ^11.3.1
  app_settings: ^7.0.0

  # 특수 UI
  sliver_tools: ^0.2.12       # iOS UITableView-style sticky 헤더
```

### assets · fonts

```yaml
flutter:
  uses-material-design: true

  assets:
    - assets/icons/
    - assets/sounds/
    - assets/branding/

  fonts:
    - family: Pretendard
      fonts:
        - asset: assets/fonts/Pretendard/Pretendard-Regular.otf
          weight: 400
        - asset: assets/fonts/Pretendard/Pretendard-Medium.otf
          weight: 500
        - asset: assets/fonts/Pretendard/Pretendard-SemiBold.otf
          weight: 600
        - asset: assets/fonts/Pretendard/Pretendard-Bold.otf
          weight: 700
        - asset: assets/fonts/Pretendard/Pretendard-ExtraBold.otf
          weight: 800
        - asset: assets/fonts/Pretendard/Pretendard-Black.otf
          weight: 900
```

> 한국 앱이면 Pretendard 권장. 라이선스 무료.

### dependency_overrides (필요 시)

로컬 fork 패키지 사용 시.
```yaml
dependency_overrides:
  image_gallery_saver:
    path: packages/image_gallery_saver
```

---

## 2. `analysis_options.yaml`

**Flutter 생성 시 `include: package:flutter_lints/flutter.yaml` 만.** 편집 필수.

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  errors:
    # freezed 2.x + json_serializable 6.x — JsonKey를 freezed 클래스의 named
    # parameter에 다는 표준 패턴을 analyzer가 false positive로 잡는다.
    # freezed 공식 권장: invalid_annotation_target은 ignore.
    invalid_annotation_target: ignore
  exclude:
    - "**/*.freezed.dart"
    - "**/*.g.dart"

linter:
  rules:
    # 프로젝트별 lint 규칙 추가·비활성 가능
    # avoid_print: false
    # prefer_single_quotes: true
```

**핵심 편집 사항.**
- `invalid_annotation_target: ignore` — freezed + json_serializable 호환
- `exclude: freezed/g.dart` — codegen 파일 분석 제외

---

## 3. `flutter_launcher_icons.yaml`

**Flutter 생성 시 없음.** `flutter_launcher_icons` 패키지 설치 후 직접 작성.

```yaml
# 런처 아이콘 생성 설정. 단일 1024 PNG → iOS Asset Catalog + Android legacy/adaptive.
# 실행 — `dart run flutter_launcher_icons`.
flutter_launcher_icons:
  android: "ic_launcher"
  ios: true
  image_path: "assets/branding/app_icon_1024.png"
  adaptive_icon_background: "#0e1319"     # 아이콘 bg · 앱 chrome 과 분리 가능
  adaptive_icon_foreground: "assets/branding/app_icon_foreground.png"
  min_sdk_android: 21
  remove_alpha_ios: true
```

**핵심 편집 사항.**
- `image_path` — 앱 아이콘 SVG → 1024 PNG 변환 후 지정
- `adaptive_icon_background` — Android adaptive bg 색 (앱 chrome 과 분리 가능)
- `adaptive_icon_foreground` — 별도 foreground PNG (viewBox 68% 등)
- 실행 — `dart run flutter_launcher_icons` · 1회 · 모든 iOS 18 사이즈 + Android 5 밀도 자동

---

## 4. Flutter 생성 시 자동 vs 편집됨 · 요약

| 파일 | 생성 시 자동 | 편집 여부 |
|---|---|---|
| `analysis_options.yaml` | flutter_lints 만 | ✅ 편집 (예외 규칙 · exclude) |
| `pubspec.yaml` | 최소 | ✅ 크게 편집 (필요 패키지) |
| `flutter_launcher_icons.yaml` | 없음 | ✅ 직접 작성 |
| `devtools_options.yaml` | Flutter 3.19+ 자동 · 비어있음 | ❌ 편집 안 함 |
| `analysis_options.yaml` linter 커스텀 | — | 앱별 판단 |

---

## 5. 검증

- `flutter pub get` — 의존성 해결 확인
- `flutter analyze` — analysis_options 정합
- `dart run flutter_launcher_icons` — 아이콘 생성 정합
- 첫 빌드 (`flutter run --flavor dev`) — 실기기 실행 확인

---

## 관련 문서

- `stack/flutter/rule-enforcement/flavor-setup.md` — `.env.dev/.env.prod` · 빌드 명령
- `docs-templates/product/01_tech_stack.template.md` — 스택 선택 · 근거
- `docs-templates/launch/prod-deploy-guide.template.md` — 배포 시 빌드
