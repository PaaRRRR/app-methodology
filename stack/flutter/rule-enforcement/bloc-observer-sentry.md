# bloc-observer-sentry — 룰 #6 강제 asset

> 룰 #6 실현. 에러 삼키지 않음. Bloc 안 throw · try-catch 놓친 것도 Sentry 로 상시 보고.

---

## 왜 이 룰이 있는가

- **Bloc 안 throw 는 UI 도달 전** — 화면에 보이지 않아서 사용자·개발자 인지 불가
- **prod 에서 조용히 실패** — try-catch 놓친 에러가 무증상으로 축적
- **회귀 늦게 발견** — 대시보드·로그 없이는 원인 파악 어려움

**해결.** `BlocObserver` 로 모든 Bloc onError 를 Sentry 로 보고 + `sealed AppError` 로 UI 계층 예외 처리 통일.

---

## 1. `AppBlocObserver`

```dart
// lib/app/bloc_observer.dart
//
// 전역 Bloc/Cubit 라이프사이클 로깅 + 미처리 에러를 Sentry 로 보고하는 옵저버.
//
// 정책 —
//   - onChange / onTransition: kDebugMode 콘솔만 (prod 콘솔 오염 방지)
//   - onError: Sentry 상시 보고 + kDebugMode 콘솔
//   - Bloc 안 throw 는 화면 도달 전이라 Sentry 가 유일한 가시성 채널
//
// 등록 위치 — lib/core/bootstrap.dart 의 `Bloc.observer = AppBlocObserver()`
// Sentry 미초기화 환경(SENTRY_DSN 빈값)에서는 captureException 이 NoOpHub 로 no-op

import 'package:flutter/foundation.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:sentry_flutter/sentry_flutter.dart';

class AppBlocObserver extends BlocObserver {
  const AppBlocObserver();

  @override
  void onChange(BlocBase<dynamic> bloc, Change<dynamic> change) {
    super.onChange(bloc, change);
    if (kDebugMode) {
      debugPrint('[Bloc] ${bloc.runtimeType} change: '
          '${change.currentState.runtimeType} → ${change.nextState.runtimeType}');
    }
  }

  @override
  void onTransition(
    Bloc<dynamic, dynamic> bloc,
    Transition<dynamic, dynamic> transition,
  ) {
    super.onTransition(bloc, transition);
    if (kDebugMode) {
      debugPrint('[Bloc] ${bloc.runtimeType} transition: '
          '${transition.event.runtimeType} → ${transition.nextState.runtimeType}');
    }
  }

  @override
  void onError(BlocBase<dynamic> bloc, Object error, StackTrace stackTrace) {
    if (kDebugMode) {
      debugPrint('[Bloc] ${bloc.runtimeType} error: $error');
    }
    Sentry.captureException(
      error,
      stackTrace: stackTrace,
      hint: Hint.withMap({'bloc': bloc.runtimeType.toString()}),
    );
    super.onError(bloc, error, stackTrace);
  }
}
```

---

## 2. `AppError` (sealed base)

```dart
// lib/core/errors/app_error.dart
//
// Repository·서비스 계층에서 throw 할 공통 에러 베이스 + 흔한 서브타입.
//
// 패턴: throw + Bloc try/catch (Result<T> 안 만듦).
// UI는 `AppError` 만 catch 해서 분기. Sentry 보고는 상위 try/catch 에서 처리.

sealed class AppError implements Exception {
  final String message;
  final Object? cause;
  final StackTrace? stackTrace;

  const AppError(this.message, {this.cause, this.stackTrace});

  @override
  String toString() => '$runtimeType($message)';
}

class NetworkError extends AppError {
  const NetworkError(super.message, {super.cause, super.stackTrace});
}

class NotFoundError extends AppError {
  const NotFoundError(super.message, {super.cause, super.stackTrace});
}

class UnauthorizedError extends AppError {
  const UnauthorizedError(super.message, {super.cause, super.stackTrace});
}

class UnknownError extends AppError {
  const UnknownError(super.message, {super.cause, super.stackTrace});
}
```

---

## 3. Postgrest 예외 매핑 (Supabase 사용 시)

```dart
// lib/core/errors/postgrest_wrap.dart
import 'package:supabase_flutter/supabase_flutter.dart';
import 'app_error.dart';

/// Repository 안 supabase 호출을 감싸서 AppError 로 변환
Future<T> wrapPostgrest<T>(Future<T> Function() fn) async {
  try {
    return await fn();
  } on PostgrestException catch (e, st) {
    if (e.code == 'PGRST116') {
      throw NotFoundError(e.message, cause: e, stackTrace: st);
    }
    if (e.code == '42501') {
      throw UnauthorizedError(e.message, cause: e, stackTrace: st);
    }
    throw NetworkError(e.message, cause: e, stackTrace: st);
  }
}
```

**사용 (Repository 안).**
```dart
Future<Profile> getProfile(String id) {
  return wrapPostgrest(() async {
    final row = await _client.from('profiles').select().eq('id', id).single();
    return Profile.fromJson(row);
  });
}
```

---

## 4. Bloc 안 try-catch 패턴

```dart
class HomeCubit extends Cubit<HomeState> {
  HomeCubit(this._repo) : super(HomeState.loading());
  final ProfileRepository _repo;

  Future<void> load(String id) async {
    try {
      final profile = await _repo.getProfile(id);
      emit(HomeState.loaded(profile));
    } on NotFoundError {
      emit(HomeState.notFound());
    } on NetworkError catch (e) {
      emit(HomeState.error('네트워크 오류'));
      // 상세 원인은 BlocObserver 가 Sentry 로 자동 보고
      rethrow;
    }
  }
}
```

---

## 5. Sentry init

```dart
// lib/core/bootstrap.dart
await SentryFlutter.init(
  (options) {
    options.dsn = env.sentryDsn;
    options.environment = env.flavor;                       // dev · prod
    options.release = '${appInfo.version}+${appInfo.buildNumber}';
    options.tracesSampleRate = env.isProd ? 0.1 : 1.0;
  },
);
Bloc.observer = const AppBlocObserver();
```

---

## 6. 검증

- Day 1 부터 통합 · test error 로 도착 확인
- 신규 화면·Bloc 추가 시 try-catch 없으면 review 지적
- prod 배포 후 Sentry 대시보드 · 새 이슈 유형 없는지 관측

---

## 관련 룰

- 룰 #6 — 에러 삼키지 않음 (개념)
- `mcp/sentry.md` — issue triage flow
