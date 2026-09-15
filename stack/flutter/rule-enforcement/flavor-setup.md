# flavor-setup — 룰 #7 강제 asset

> 룰 #7 실현. dev/prod 환경 섞지 않음. flavor 로 완전 분리 · 한 기기 둘 다 설치 가능.

---

## 왜 이 룰이 있는가

- **dev 실험이 prod 에 오염** — 같은 env 쓰면 실수 시 사용자 데이터 손상
- **베타 테스터 혼란** — 앱 아이콘·이름 같으면 어느 env 인지 인식 못함
- **hotfix 사고** — 특정 env 값 누락 시 실기기 크래시 (StartRoom 1.0.2+11)

**해결.** `--dart-define-from-file=.env.{flavor}` · `AppEnv.{flavor}` 컴파일 타임 상수 · 진입점 (`entry.dev.dart` · `entry.prod.dart`) 완전 분리.

---

## 1. `AppEnv` (컴파일 타임 상수 컨테이너)

```dart
// lib/core/env.dart
//
// flavor별 환경 값을 모은 컴파일 타임 상수 컨테이너.
// 실제 값은 --dart-define-from-file=.env.{flavor} 로 빌드 시 주입한다.

class AppEnv {
  final String flavor;
  final String supabaseUrl;
  final String supabaseAnonKey;
  final String sentryDsn;
  final String amplitudeApiKey;

  const AppEnv._({
    required this.flavor,
    required this.supabaseUrl,
    required this.supabaseAnonKey,
    required this.sentryDsn,
    required this.amplitudeApiKey,
  });

  static const AppEnv dev = AppEnv._(
    flavor: 'dev',
    supabaseUrl: String.fromEnvironment('SUPABASE_URL'),
    supabaseAnonKey: String.fromEnvironment('SUPABASE_ANON_KEY'),
    sentryDsn: String.fromEnvironment('SENTRY_DSN'),
    amplitudeApiKey: String.fromEnvironment('AMPLITUDE_API_KEY'),
  );

  static const AppEnv prod = AppEnv._(
    flavor: 'prod',
    supabaseUrl: String.fromEnvironment('SUPABASE_URL'),
    supabaseAnonKey: String.fromEnvironment('SUPABASE_ANON_KEY'),
    sentryDsn: String.fromEnvironment('SENTRY_DSN'),
    amplitudeApiKey: String.fromEnvironment('AMPLITUDE_API_KEY'),
  );

  bool get isDev => flavor == 'dev';
  bool get isProd => flavor == 'prod';
}
```

---

## 2. 진입점 분리

```
lib/entry/
├── entry.dev.dart
└── entry.prod.dart
```

```dart
// lib/entry/entry.dev.dart
import '../core/bootstrap.dart';
import '../core/env.dart';

void main() => bootstrap(env: AppEnv.dev);
```

```dart
// lib/entry/entry.prod.dart
import '../core/bootstrap.dart';
import '../core/env.dart';

void main() => bootstrap(env: AppEnv.prod);
```

`lib/core/bootstrap.dart` 는 공통. env 만 다르게 주입.

---

## 3. `.env` 파일 (git 미포함)

```
# .env.dev
SUPABASE_URL=https://xxxxx-dev.supabase.co
SUPABASE_ANON_KEY=eyJ...
SENTRY_DSN=https://...
AMPLITUDE_API_KEY=...

# .env.prod
SUPABASE_URL=https://xxxxx-prod.supabase.co
SUPABASE_ANON_KEY=eyJ...
SENTRY_DSN=https://...
AMPLITUDE_API_KEY=...
```

**`.env.example` 는 커밋** — 필드 이름만.

---

## 4. 빌드 명령

```bash
# 실행 (개발)
flutter run --flavor dev -t lib/entry/entry.dev.dart --dart-define-from-file=.env.dev

# iOS prod (App Store)
flutter build ipa --flavor prod -t lib/entry/entry.prod.dart --release \
  --dart-define-from-file=.env.prod \
  --export-options-plist=ios/ExportOptions.plist

# iOS dev (Firebase Distribution)
flutter build ipa --flavor dev -t lib/entry/entry.dev.dart --release \
  --dart-define-from-file=.env.dev \
  --export-method=ad-hoc

# Android prod (Play Store)
flutter build appbundle --flavor prod -t lib/entry/entry.prod.dart --release \
  --dart-define-from-file=.env.prod

# Android dev (Firebase Distribution)
flutter build apk --flavor dev -t lib/entry/entry.dev.dart --release \
  --dart-define-from-file=.env.dev
```

> **필수.** `--dart-define-from-file=.env.{flavor}` 없이 빌드하면 env 값이 빈 문자열이 되어 `Supabase.initialize()` skip · 실기기 크래시. StartRoom 1.0.2+11 hotfix 사례.

---

## 5. iOS Xcode Configuration

**flavor 별 Configuration 4개.**
- Debug-dev, Release-dev, Debug-prod, Release-prod (Profile-dev/prod 도 필요 시)

**Bundle Identifier · 앱 이름.**
- dev — `{APP_ID}.dev` / `{APP_NAME} Dev`
- prod — `{APP_ID}` / `{APP_NAME}`

**Run Script — GoogleService-Info.plist 분기.**
```bash
case "${CONFIGURATION}" in
  "Debug-dev"|"Release-dev"|"Profile-dev" )
    cp -r "${PROJECT_DIR}/Runner/config/dev/GoogleService-Info.plist" \
          "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/GoogleService-Info.plist" ;;
  "Debug-prod"|"Release-prod"|"Profile-prod"|"Debug"|"Release"|"Profile" )
    cp -r "${PROJECT_DIR}/Runner/config/prod/GoogleService-Info.plist" \
          "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/GoogleService-Info.plist" ;;
  *) echo "warning: no Firebase config for CONFIGURATION=${CONFIGURATION}, skipping" ;;
esac
```

> **안전망.** Runner target 기본 `Debug/Release/Profile` config (Xcode UI 에서 Runner scheme 그대로 Archive 시 사용) 도 prod Firebase 로 fallback. 1.0.2+11 사고 재발 방지.

---

## 6. Android

`android/app/src/dev/` · `android/app/src/prod/` 에 각각 `google-services.json` — productFlavor 자동 인식.

**build.gradle**
```gradle
android {
  flavorDimensions "env"
  productFlavors {
    dev {
      dimension "env"
      applicationIdSuffix ".dev"
      resValue "string", "app_name", "StartRoom Dev"
    }
    prod {
      dimension "env"
      resValue "string", "app_name", "StartRoom"
    }
  }
}
```

---

## 7. 검증

- 두 flavor 동시 설치 · 아이콘·이름 구분 확인
- dev 데이터 · prod 데이터 완전 격리 확인
- Sentry event 에 `environment` = `dev` / `production` 정합

---

## 8. 원칙

- 베타 테스터는 prod 만 받음 · dev 데이터 미노출
- dev 는 마음껏 데이터 리셋 · 마이그레이션 불필요
- prod 는 실험·테스트 금지 (룰 · 절대 금지 #2)

---

## 관련 룰

- 룰 #7 — dev/prod 섞지 않음 (개념)
- 룰 · 절대 금지 #2 — prod 실험 금지
