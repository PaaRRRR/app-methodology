# 02. 아키텍처

> 폴더 구조 + 데이터 모델 방향성 + BLoC 패턴 + Repository 인터페이스.
> Week 1 D3~D4 작업 기준. 며칠 안에 이 골격까지 짜두면 그 다음은 화면 채우기 작업이 된다.

> **주의** — 구체적인 SQL (테이블 정의 · RLS · 트리거) 은 이 문서에 박지 않는다. 스키마는 개발 중 가장 자주 바뀌므로 **실제 DB + `supabase/migrations/` 가 단일 진실 공급원**. 이 문서는 "무엇이 왜 필요한지" 와 "지켜야 할 원칙" 만 다룬다.

---

## 1. 프로젝트 폴더 구조 (feature-first)

```
{APP_SLUG}/
├── lib/
│   ├── main.dart
│   ├── app/
│   │   ├── app.dart                 # MaterialApp + MultiRepositoryProvider + ThemeCubit wiring
│   │   ├── router.dart              # go_router config
│   │   ├── theme.dart               # Palette (ThemeExtension · 다크/라이트) · context.colors 확장
│   │   ├── theme_cubit.dart         # ThemeMode 상태 · SharedPreferences 저장
│   │   └── bloc_observer.dart       # 전역 Bloc 로깅 + onError Sentry 보고
│   ├── core/
│   │   ├── env.dart                 # 환경 변수 접근 (dart-define)
│   │   ├── constants.dart           # 도메인 상수 (DB 와 짝)
│   │   ├── datetime/                # clock · kst · format_duration (룰 #11 강제)
│   │   └── errors/                  # sealed AppError + postgrest_wrap
│   ├── data/
│   │   ├── models/                  # freezed
│   │   ├── repositories/            # abstract + impl 쌍
│   │   └── supabase/                # client · channels · queries
│   ├── features/                    # 화면별 폴더
│   │   └── {feature}/
│   │       ├── {feature}_screen.dart
│   │       ├── {feature}_bloc.dart  (또는 _cubit)
│   │       ├── {feature}_state.dart
│   │       ├── {feature}_event.dart (Bloc 만)
│   │       └── widgets/             # 이 화면 전용 위젯
│   ├── shared/
│   │   ├── widgets/                 # {PREFIX}Button, {PREFIX}Card, ...
│   │   ├── icons/                   # phosphor 매핑
│   │   └── analytics.dart           # analytics wrapper
│   └── l10n/                        # ko 우선, en 차후
├── supabase/
│   ├── migrations/                  # 실제 스키마의 진실
│   └── functions/                   # Edge Functions
├── assets/
├── test/                            # repository + bloc 단위
├── pubspec.yaml
├── CLAUDE.md
└── .env.dev / .env.prod / .env.example
```

---

## 2. 데이터 모델 (방향성)

> 실 컬럼·타입·제약은 `supabase/migrations/` 와 실제 DB 가 진실. 여기서는 어떤 테이블이 왜 필요한지와 관계만 잡는다.

### 필요한 테이블과 목적

| 테이블 | 목적 | 비고 |
|---|---|---|
| `profiles` | {목적} | {특이사항} |
| `{TABLE_2}` | {목적} | {특이사항} |
| ... | ... | ... |

### 핵심 관계
```
profiles ──< {ENTITY_1} >── ...
```

---

## 3. BLoC 패턴

- **State** — `Equatable` 상속 (또는 freezed)
- **복잡한 화면** — full Bloc (Event → State)
- **단순한 화면** — Cubit
- **Realtime stream** — Bloc 안에서 `StreamSubscription` 구독, 이벤트로 변환. dispose 에서 해제 (룰 #5)

상세 코드 예시 — `stack/flutter/rule-enforcement/bloc-vs-cubit.md`

---

## 4. Repository 인터페이스

- UI 는 Supabase 직접 호출 금지 (룰 #3). Repository 를 통한다
- Repository = abstract + impl 쌍 (테스트 mock 용이)
- 필드 — `AuthRepo` · `{DOMAIN}Repo` · `NotificationRepo` · ...

---

## 5. 환경 · flavor

- dev / prod flavor 분리 (룰 #7)
- 진입점 — `lib/entry/entry.dev.dart` · `entry.prod.dart`
- `--dart-define-from-file=.env.{flavor}` 필수
- 상세 — `stack/flutter/rule-enforcement/flavor-setup.md`

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/02_architecture.md`. 8개 테이블 (profiles · rooms · room_participants · room_events · tasks · retrospectives · match_queue · notifications) + 12쌍 Repository.
