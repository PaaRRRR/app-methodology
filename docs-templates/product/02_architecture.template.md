# 02. 아키텍처

> 폴더 구조 + 데이터 모델 방향성 + BLoC 패턴 + Repository 인터페이스.
> Week 1 D3~D4 작업 기준. 며칠 안에 이 골격까지 짜두면 그 다음은 화면 채우기 작업이 된다.

> **주의** — 구체적인 SQL (테이블 정의 · RLS · 트리거) 은 이 문서에 박지 않는다. 스키마는 개발 중 가장 자주 바뀌므로 **실제 DB + `supabase/migrations/` 가 단일 진실 공급원**. 이 문서는 "무엇이 왜 필요한지" 와 "지켜야 할 원칙" 만 다룬다.

---

## 1. 프로젝트 폴더 구조 (feature-first)

```
{APP_SLUG}/
├── lib/
│   ├── app/
│   │   ├── app.dart                 # MaterialApp + MultiRepositoryProvider + ThemeCubit wiring
│   │   ├── router.dart              # go_router config (라우트 정의)
│   │   ├── theme.dart               # Palette extends ThemeExtension (다크/라이트 두 벌) · context.colors 확장
│   │   ├── theme_cubit.dart         # ThemeMode 상태 · SharedPreferences 저장
│   │   ├── motion.dart              # AppDurations · AppCurves 토큰
│   │   └── bloc_observer.dart       # 전역 Bloc 라이프사이클 로깅 + onError Sentry 보고
│   ├── core/
│   │   ├── env.dart                 # AppEnv (dart-define 컴파일 상수)
│   │   ├── bootstrap.dart           # Firebase/Supabase/Amplitude/Push 초기화 순서
│   │   ├── constants.dart           # 도메인 상수 (DB 와 짝)
│   │   ├── datetime/                # clock (appNow) · kst (toKst) · format_duration (룰 #11)
│   │   ├── errors/                  # sealed AppError + postgrest_wrap
│   │   ├── identity/                # install_id · stable_device_id
│   │   ├── push/                    # push_service (FCM) · local_notif
│   │   ├── permissions/             # camera · notif 등
│   │   ├── settings/                # settings_prefs (SharedPreferences)
│   │   └── utils/                   # avatar_color 등 순수 헬퍼
│   ├── data/
│   │   ├── models/                  # freezed 모델 (Profile, Room, ...)
│   │   ├── repositories/            # abstract + impl 쌍 (AuthRepo, RoomRepo, StatsRepo, ...)
│   │   ├── services/                # 도메인 서비스 (매칭 라우팅 · 통계 집계 · 캐시)
│   │   └── supabase/                # client · channels · queries
│   ├── entry/
│   │   ├── entry.dev.dart           # dev flavor 진입점 · main() { bootstrap(env: AppEnv.dev); }
│   │   └── entry.prod.dart          # prod flavor 진입점
│   ├── features/                    # 화면별 폴더
│   │   └── {feature}/
│   │       ├── {feature}_screen.dart
│   │       ├── {feature}_bloc.dart  (또는 _cubit)
│   │       ├── {feature}_state.dart
│   │       ├── {feature}_event.dart (Bloc 만)
│   │       └── widgets/             # 이 화면 전용 위젯
│   ├── shared/
│   │   ├── widgets/                 # {PREFIX}Button, {PREFIX}Card, ...
│   │   ├── icons/                   # phosphor 매핑 · SR 아이콘 의미 이름
│   │   └── analytics.dart           # Amplitude wrapper (track · identify · setUserId)
│   └── l10n/                        # ko 우선, en 차후
├── supabase/
│   ├── migrations/                  # 실제 스키마의 진실 · 001, 002, ... 순차 파일
│   └── functions/                   # Edge Functions
├── assets/
├── test/
├── pubspec.yaml
├── CLAUDE.md
├── analysis_options.yaml
├── flutter_launcher_icons.yaml
└── .env.dev / .env.prod / .env.example
```

**핵심 원칙.**
- **feature-first** — 화면별 폴더 · 공통은 `shared/`. layer-first (models/ · views/ · controllers/ 등) 보다 화면 개발·리팩토링 단위 명확
- **entry 분리** — `main.dart` 아닌 `lib/entry/entry.{flavor}.dart` (룰 #7)
- **core 세분** — 성격별 서브 폴더. 한 폴더에 100 파일 몰지 않음

---

## 2. 데이터 모델 (방향성)

> 실 컬럼·타입·제약은 `supabase/migrations/` 와 실제 DB 가 진실. 여기서는 어떤 테이블이 왜 필요한지와 관계만 잡는다.

### 필요한 테이블과 목적

| 테이블 | 목적 | 비고 |
|---|---|---|
| `profiles` | {목적 · 사용자 프로필} | 익명 인증 · 자동 닉네임 |
| `{TABLE_2}` | {목적} | {특이사항} |
| ... | ... | ... |

### 핵심 관계
```
profiles ──< {ENTITY_1} >── ...
```

---

## 3. BLoC 패턴

- **State** — `Equatable` 상속 (또는 freezed sealed)
- **복잡한 화면** — full Bloc (Event → State · 상태 머신 명시적)
- **단순한 화면** — Cubit (가볍고 빠름)
- **Realtime stream** — Bloc 안 `StreamSubscription` 으로 구독, `add(Event)` 로 변환. `close()` 오버라이드에서 `cancel()` 필수 (룰 #5 강제)
- **에러** — Bloc 안 `try/catch` · `emit(errorState)` + `rethrow` (BlocObserver 가 Sentry 자동 보고 · 룰 #6)

상세 코드 — `stack/flutter/rule-enforcement/bloc-vs-cubit.md`

---

## 4. Repository 인터페이스

- **UI 는 Supabase 직접 호출 금지** (룰 #3). Repository 를 통한다
- **abstract + impl 쌍** — 테스트 mock 용이 · 다른 백엔드 (예: REST → GraphQL) 로 교체 유연
- **Postgrest 예외 매핑** — `wrapPostgrest()` 헬퍼로 `PostgrestException` → `NetworkError`/`NotFoundError`/`UnauthorizedError` 변환. UI 는 `AppError` 만 catch

```dart
// lib/data/repositories/{domain}_repository.dart (abstract)
abstract class {Domain}Repository {
  Future<{Model}> get(String id);
  Stream<List<{Model}>> watchAll();
  Future<void> update(...);
}

// lib/data/repositories/{domain}_repository_impl.dart
class Supabase{Domain}Repository implements {Domain}Repository { ... }
```

---

## 5. 환경 · flavor

- dev / prod flavor 분리 (룰 #7)
- 진입점 — `lib/entry/entry.dev.dart` · `entry.prod.dart`
- `--dart-define-from-file=.env.{flavor}` 필수 (누락 시 실기기 크래시)
- 상세 코드 — `stack/flutter/rule-enforcement/flavor-setup.md`

---

## 6. Theme · ThemeExtension 패턴

- `Palette extends ThemeExtension<Palette>` — 다크/라이트 두 벌
- `context.colors` extension 으로 접근 (`context.colors.accent` · `context.colors.bg`)
- **Fill/Ink 페어 분리** — 라이트 mode 에서 accent fill 을 텍스트/border 로 쓰면 WCAG AA 실패 → fill · ink 분리
- **다크 고정 예외** — 공유 카드 · QR chrome · Dev 스크린 등 명시적 예외 (`Palette.dark.*` 정적 접근)
- 상세 — `stack/flutter/rule-enforcement/design-tokens.md`

---

## 7. 도메인 상수는 DB 와 짝

- `lib/core/constants.dart` 에 정의된 값 (예: 방 코드 길이 · 매칭 윈도우 · streak 리셋 시각) 은 DB migration · Edge Function 과 정합 유지
- 코드에서 하드코딩 X · 상수 참조

---

## 8. Realtime 채널 관리

- `lib/data/supabase/channels.dart` — 채널 factory 한 곳
- 화면 진입 시 subscribe · dispose 에서 unsubscribe (룰 #5)
- publication 등록된 테이블만 실 realtime 동작 (`alter publication supabase_realtime add table {table}`)
- silent trigger 패턴 (알림 vs realtime 신호 분리) — 필요 시 `mcp/supabase-dev-vs-prod.md` tips 참조

---

## 9. 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/02_architecture.md`.
> 8개 테이블 (profiles · rooms · room_participants · room_events · tasks · retrospectives · match_queue · notifications) · 12쌍 Repository · Room 화면만 full Bloc · 나머지 Cubit · Realtime 3채널 (room · participants · events).
