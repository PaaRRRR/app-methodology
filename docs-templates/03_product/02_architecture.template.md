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

## 2. 데이터 층

### 2-A. 필요한 테이블 (placeholder)

> 실 컬럼·타입·제약은 `supabase/migrations/` 와 실제 DB 가 진실. 여기서는 어떤 테이블이 왜 필요한지와 관계만 잡는다.

| 테이블 | 목적 | 비고 |
|---|---|---|
| `profiles` | {목적 · 사용자 프로필} | 익명 인증 · 자동 닉네임 |
| `{TABLE_2}` | {목적} | {특이사항} |
| ... | ... | ... |

**핵심 관계 —**
```
profiles ──< {ENTITY_1} >── ...
```

**tip —** 알림·사용자 preferences 는 `profiles.notification_settings jsonb default '{}'::jsonb` 로. 새 키 추가 시 컬럼 추가 없이 확장. 기본값은 앱별 정의.

### 2-B. 데이터 접근 원칙 (앱 무관)

- **모든 테이블 RLS 활성화** — DB 레벨 권한 강제. `auth.uid()` 기준 (익명 인증에서도 정상 작동)
- **협업 · 방 기반 앱은 `is_{scope}_member(id)` SECURITY DEFINER 헬퍼** — `{scope}_participants` 정책 재귀 회피 · `authenticated` 만 EXECUTE 허용 · `anon` REVOKE
- **모델은 freezed** — JSON 직렬화 필요 · `copyWith` 필수. 시간 필드는 UTC 저장 · 표시할 때만 KST 변환 (룰 #4)
- **FK ON DELETE 정책** — 기본 `CASCADE` (본인 데이터 삭제 시 관련 데이터도) · 예외 `SET NULL` (기록 보존 필요한 자리). SET NULL 자리에 남는 PII 는 BEFORE UPDATE 트리거로 마스킹 (PIPA/GDPR 정합)
- **soft delete 우선** — 즉시 delete 대신 `left_at IS NULL` / `deleted_at IS NULL` / `state='canceled'` 필터. 감사·복구·"이미 취소된 자원" 우아한 에러 처리 가능
- **GDPR / PIPA delete-account edge function** — 사용자 요청 시 hard delete 는 JWT 검증 → `service_role` CASCADE 삭제 edge function 으로. 앱에서 직접 삭제 X (RLS 우회 필요 + 여러 테이블 원자적 처리)

### 2-C. Client Identity — 3층 분리

각 목적이 달라 저장소·수명이 다름.

| 층 | 위치 | 목적 | 수명 |
|---|---|---|---|
| **`profiles.id`** (auth.uid) | Supabase Auth SDK (iOS Keychain / Android EncryptedSharedPreferences) | 서버 진실 · 익명 세션 | 재설치 시 wipe |
| **`StableDeviceId`** | `lib/core/identity/stable_device_id.dart` · Keychain first_unlock | Analytics user_id · 사용자 재설치에도 유지 시도 | iOS 정책에 따라 잔존 · Android = 재설치 시 wipe |
| **`InstallId`** | `lib/core/identity/install_id.dart` · SharedPreferences UUID | 재설치 감지 마커 (`isFirstRun` → FCM token 강제 재발급) | 재설치 시 wipe |

`isFirstRun=true` 시 push token 강제 재발급 (APNs `deleteToken` + `getToken`). 재설치 후 오래된 token 이 서버에 남는 이슈 방지.

### 2-D. Android Backup 정책

Android 기본 `allowBackup=true` = Google Drive Auto Backup 이 SharedPreferences 복원. iOS 와 재설치 동작 divergence (iOS = 모두 wipe · Android = SharedPreferences 복원) → `install_id` 같은 재설치 초기화 대상이 잘못 복원.

**정책** — `AndroidManifest.xml` 3속성:
- `android:allowBackup="false"` — Google Drive Auto Backup 차단
- `android:fullBackupContent="false"` — 백워드 호환 명시
- `android:dataExtractionRules="@xml/data_extraction_rules"` — Android 12+ D2D 이관도 차단

결과 — Android 도 재설치 시 wipe (iOS 정합) · `install_id` 정상 초기화.

### 2-E. 자동화 (트리거)

**원칙 — 정확성 중요한 자동화는 DB 트리거 (서버) 안전.** 클라이언트 계산은 race condition · 앱 버전 차이로 어긋남.

**4가지 유형 —**

| 유형 | 시점 | 용도 | 실현 |
|---|---|---|---|
| **INSERT 자동 세팅** | `auth.users` insert 시 | 익명 profile row + 자동 닉네임 부여 | 10회 retry 로 닉네임 충돌 해소 |
| **BEFORE INSERT 코드 부여** | 신규 row 삽입 전 | UNIQUE 코드 · slug 등 자동 생성 | 5회 retry + `SECURITY DEFINER` (RLS 우회 + 전체 UNIQUE 체크) |
| **BEFORE UPDATE PII 마스킹** | 사용자 삭제 시 (FK SET NULL) | 남는 이벤트 행의 텍스트를 `"[삭제됨]"` 마스킹 | SET NULL 로 기록 보존 + PIPA 탈퇴 동시 충족 |
| **AFTER UPDATE 파생 데이터 갱신** | 특정 컬럼 채워질 때 | streak · total_count · last_completed_date 등 집계 | **`WHEN` 절로 멱등 보장** (중복 발화 시 두 번 안 오름) · `FOR UPDATE` 락으로 동시성 안전 |

**보안 tips —** SECURITY DEFINER 함수는 `SET search_path = public, pg_temp` 명시 (Advisor 지적). helper 함수 (`is_{scope}_member` 등) 는 `anon` EXECUTE REVOKE.

### 2-F. Cron (pg_cron + pg_net)

주기 작업 (알림 · stale row 정리 · streak daily · weekly summary 등) = `pg_cron` + `pg_net` 활성 후 Edge Function 호출.

**등록 자리 —** Dashboard SQL Editor 직접 실행 (service_role key 노출 · migration 파일에 박지 않음 · placeholder 치환 방식).

```sql
select cron.schedule(
  'notif-tick-60s',
  '* * * * *',
  $$ select net.http_post(
       url := '<PROJECT_URL>/functions/v1/{edge-function}',
       headers := jsonb_build_object(
         'Authorization', 'Bearer <SERVICE_ROLE_KEY>',
         'Content-Type', 'application/json'
       )
     ) $$
);
```

또는 DB RPC 직접 호출 (`$$ select public.{rpc_name}() $$`) — key 노출 회피 · 권장.

**확인 —** `select jobname, schedule, command, active from cron.job;`

### 2-G. 마이그레이션 운영 (원칙)

- **신규 = 항상 다음 번호로 append.** 과거 파일 수정 금지 (이력 보존)
- **dev = MCP `apply_migration` 단일 채널** · Dashboard SQL Editor 는 SELECT/EXPLAIN 전용 (DDL 치면 `supabase_migrations` 메타 어긋남)
- **prod = Supabase CLI `db push`** (수동)
- **스키마 변경 시 §2-A 표 · 관련 문서 sync**

상세 tips (표기명 규칙 · `list_migrations` 누락 · placeholder 있는 migration 예외) — `08_dev_ops.md` §4-A/B.

---

## 3. Repository 인터페이스

- **UI 는 Supabase 직접 호출 금지** (룰 #3). Repository 를 통한다
- **abstract + impl 쌍** — 테스트 mock 용이 · 다른 백엔드로 교체 유연

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

### 3-A. DI 패턴 — `RepositoryProvider` 최상단

- `flutter_bloc` 의 `RepositoryProvider` 를 `App` 최상단에 두고 화면은 `context.read<{Domain}Repository>()` 로 받는다
- **`get_it` 미도입** — V1 규모에 무거움 · BLoC 컨텍스트 밖 호출 가능해져 "UI → Bloc → Repo" 단방향 (룰 #3) 흐림. `RepositoryProvider` 는 이미 flutter_bloc 안 포함 · 신규 의존성 0

### 3-B. 에러 처리 — expected vs unexpected

두 갈래 분류.

- **expected** — dispose 시 leave 재호출 · 사용자 취소 등. `catch (e)` + dev 콘솔만 (Sentry X · 노이즈 회피)
- **unexpected** — 원인 불명 · 데이터 무결성 위반 등. `catch (e, st)` + `Sentry.captureException(e, stackTrace: st)`
- **`catch (_)` 는 0건 유지** — 룰 #6 (에러 삼키지 않음) 자기복제되기 쉬움 · 코드 리뷰 시 grep

**Postgrest 예외 매핑 —** `wrapPostgrest()` 헬퍼로 `PostgrestException` → `NetworkError`/`NotFoundError`/`UnauthorizedError` 변환. UI 는 `AppError` 만 catch. `Result<T>` 는 만들지 않음 (unwrap 반복 + Sentry 품질↓).

### 3-C. `.order()` 기본값 주의 (postgrest-dart)

**`postgrest-dart` (v2.7+) `.order()` default = DESCENDING** (Supabase JS 와 반대).

```dart
.order('created_at')                      // ❌ 위험 · descending
.order('created_at', ascending: true)     // ✅ 정합
```

Repository 안 모든 `.order()` 호출 = **explicit** `ascending: true` / `false`. Client-side sort safety net 은 realtime jitter · 패키지 upgrade 방어용 optional.

---

## 4. BLoC 패턴

- **State** — `Equatable` 상속 (또는 freezed sealed)
- **복잡한 화면** — full Bloc (Event → State · 상태 머신 명시적)
- **단순한 화면** — Cubit (가볍고 빠름)
- **에러** — Bloc 안 `try/catch` · `emit(errorState)` + `rethrow` (BlocObserver 가 Sentry 자동 보고 · 룰 #6)

### 4-A. 모델 vs State 도구 분리

| 대상 | 도구 | 이유 |
|---|---|---|
| **데이터 모델** (Profile · Room · ...) | **freezed** | JSON 직렬화 필수 · `copyWith` 자주 사용 |
| **BLoC State / Event** | **Equatable** | 직렬화 불필요 · 비교만. 개발 중 자주 바뀌므로 build_runner 안 거치는 게 속도 유리 |

**State 에 필드 추가 시 반드시 `copyWith` + `props` 양쪽 동시 갱신.** 빼먹으면 비교/갱신 버그.

### 4-B. Realtime → Bloc 연결

Realtime stream 은 Bloc 안 `StreamSubscription` 으로 구독 · `add(Event)` 로 변환. **`close()` 오버라이드에서 `cancel()` 필수** (룰 #5 강제).

상세 코드 스켈레톤 — `/stack/flutter/rule-enforcement/bloc-vs-cubit.md`.

---

## 5. Realtime 채널

- `lib/data/supabase/channels.dart` — 채널 factory 한 곳
- 화면 진입 시 subscribe · dispose 에서 unsubscribe (룰 #5)
- publication 등록된 테이블만 실 realtime 동작 (`alter publication supabase_realtime add table {table}`)
- **`user_id=me` 같은 filter 매칭 시 `alter table {table} replica identity full` 필수** — 안 하면 filter 안 걸려서 다른 사용자 이벤트까지 옴

### 5-A. 채널 명명 규칙 · 설계 (placeholder)

**패턴 —** `{scope}:{id}-{purpose}` (예: `room:{roomId}-events` · `user:{userId}-notifications`)

| 채널 | 쓰임 | 구독 시점 |
|---|---|---|
| `{scope}:{id}-{purpose_1}` | {용도} | 화면 진입 · dispose |
| `{scope}:{id}-{purpose_2}` | {용도} | 앱 시작 · 로그아웃까지 유지 |

### 5-B. 헬퍼별 채널 분리 vs 통합

**초기 = 헬퍼별 분리** (예: `room:{id}-events` · `-participants` · `-room` 각각). 화면 1개 진입 = 채널 N 개.

- **장점** — 화면 1 = 채널 N 로 누수 검증 set size 로 명확 · `supabase_flutter` 한 채널 다중 `onPostgresChanges` 이슈 회피
- **단점** — connection 개수 = MAU × 채널 수. Free tier 한도 (500 동시 · 2M msg/mo) 근접 시 통합 재평가
- **결정 지연 원칙** — 초기엔 분리 · 실 사용 시점에 매트릭스 (헬퍼당 채널 · 피크 시나리오 · MAU 상한) 만들고 재평가

### 5-C. Silent trigger 패턴

알림 vs Realtime 신호 분리. `_silent` 플래그 · silent trigger 로 UI cubit 만 신호받고 알림은 안 보냄. 필요 자리는 앱마다 상이 · 도메인 판단.

**예 —** 방 나가기 · 방 취소 등 "다른 참가자 화면은 즉시 반영해야 하지만 push 알림은 안 보내야 하는" 이벤트. Realtime broadcast 는 발화 · notifications 트리거는 `_silent` 필터로 skip.

---

## 6. 도메인 상수 = DB 와 짝

- `lib/core/constants.dart` 에 정의된 값 (예: 방 코드 길이 · 매칭 윈도우 · streak 리셋 시각) 은 DB migration · Edge Function 과 정합 유지
- 코드에서 하드코딩 X · 상수 참조

---

## 7. Theme · ThemeExtension

- `Palette extends ThemeExtension<Palette>` — 다크/라이트 두 벌 · `context.colors` extension
- 상세 원칙 (Fill/Ink 페어 · 다크 고정 예외) — `04_design_system.md` §2
- 코드 스켈레톤 — `/stack/flutter/rule-enforcement/design-tokens.md`

---

## 8. 환경 · flavor

- dev / prod flavor 분리 (룰 #7)
- 진입점 — `lib/entry/entry.dev.dart` · `entry.prod.dart`
- `--dart-define-from-file=.env.{flavor}` 필수 (누락 시 실기기 크래시)
- 상세 코드 — `/stack/flutter/rule-enforcement/flavor-setup.md`

---

## 9. 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/02_architecture.md`.

**규모 —** 8개 테이블 (profiles · rooms · room_participants · room_events · tasks · retrospectives · match_queue · notifications) · 12쌍 Repository · Room 화면만 full Bloc · 나머지 Cubit · Realtime 3채널 (room · participants · events).

**주요 자리 —**
- **자동화 트리거 4종** — `handle_new_user` (익명 profile + 닉네임 10회 retry) · `assign_room_code` (BEFORE INSERT · 5회 retry · SECURITY DEFINER · UNIQUE 충돌 해소) · `scrub_room_event_pii` (BEFORE UPDATE · user_id SET NULL 시 message 마스킹) · `on_participant_finished` (`WHEN` 절 멱등 + `FOR UPDATE` 락 · streak/total_rooms/last_completed_date 갱신)
- **Cron 4자리** — match-tick (매칭 큐 60초) · notif-tick (before_1h · before_10m 60초) · streak-daily (20:00 KST) · weekly-summary (일요일 21:00 KST)
- **`rooms.state='canceled'` 유지** — delete 미채택 이유 = 운영 지표·디버깅 흔적 · 잘못된 코드 접근 시 "이미 다른 파트너와 시작했어요" 우아한 처리
- **retrospectives (room_id, user_id) UNIQUE upsert** — null 인자는 기존 값 보존 (부분 갱신 안전)
- **매칭 알고리즘** — "각자 만들고 매칭 시 합치기" · 결정적 룰 (created_at 빠른 쪽 · 같으면 id 사전순) · 살아남지 못한 방 = canceled · 자세히는 원문 §9
- **GDPR delete-account** — `supabase/functions/delete-account/index.ts` · JWT 검증 → service_role CASCADE 삭제
