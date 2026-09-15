# bloc-vs-cubit — 룰 #4/#5 강제 asset

> 룰 #3 (UI→Bloc→Repository 순서) · 룰 #5 (Realtime 구독 해제) 실현.
> Bloc vs Cubit 선택 원칙 + StreamSubscription 안전 관리.

---

## 왜 이 룰이 있는가

- **UI 에서 Supabase 직접 호출 시** — 테스트 어렵고 재사용 불가
- **복잡 상태 머신을 Cubit 으로** — event 추적 · 로깅 어려움
- **StreamSubscription 미해제** — 화면 dispose 후에도 콜백 실행 · setState after unmounted 크래시

**해결.** UI → Bloc → Repository → Supabase 순서 강제 · 복잡=Bloc · 단순=Cubit · Realtime 은 Bloc 안에서 `StreamSubscription` 관리 · `close()` 오버라이드에서 해제.

---

## 1. UI → Bloc → Repository → Supabase (룰 #3)

**금지.** UI 위젯에서 `Supabase.instance.client.from('...')` 직접 호출.

**허용.**
```
UI (screen/widget)
    ↓ context.read<Cubit>().action()
Cubit / Bloc
    ↓ _repo.method()
Repository (abstract)
    ↓ (impl)
RepositoryImpl
    ↓ _client.from('...')
Supabase
```

---

## 2. Bloc vs Cubit 선택

| 판단 기준 | 선택 |
|---|---|
| 복잡한 상태 머신 (여러 event → 여러 state 전환) | **Bloc** (event → state 명시적) |
| 단순 상태 (load · error · success 정도) | **Cubit** (가볍고 빠름) |
| Realtime stream 구독 있음 | **Bloc** 권장 (event 로 변환 후 처리) |
| 애니메이션 · 임시 UI 상태 | Cubit or ValueNotifier |

**예시.**
- StartRoom `RoomBloc` — Room 화면 (joining → waitingPartner → scrumChat → countdown → live → finishing 다단계) = Bloc
- StartRoom `HomeCubit` — Home 화면 (loading · loaded · empty) = Cubit

---

## 3. Cubit 예시 (단순 상태)

```dart
class HomeCubit extends Cubit<HomeState> {
  HomeCubit(this._repo) : super(HomeState.loading());
  final HomeRepository _repo;

  Future<void> load() async {
    try {
      final data = await _repo.getHome();
      emit(HomeState.loaded(data));
    } on NetworkError {
      emit(HomeState.error('네트워크 오류'));
      rethrow; // BlocObserver → Sentry
    }
  }
}
```

---

## 4. Bloc 예시 (복잡 · Realtime 구독)

```dart
// lib/features/room/room_event.dart
sealed class RoomEvent {}
class RoomEntered extends RoomEvent { final String roomId; RoomEntered(this.roomId); }
class RoomParticipantChanged extends RoomEvent { final List<Participant> ps; RoomParticipantChanged(this.ps); }
class RoomReady extends RoomEvent {}
class RoomLeft extends RoomEvent {}

// lib/features/room/room_bloc.dart
class RoomBloc extends Bloc<RoomEvent, RoomState> {
  RoomBloc(this._repo) : super(RoomState.initial()) {
    on<RoomEntered>(_onEntered);
    on<RoomParticipantChanged>(_onParticipantChanged);
    on<RoomReady>(_onReady);
    on<RoomLeft>(_onLeft);
  }

  final RoomRepository _repo;
  StreamSubscription<List<Participant>>? _participantSub;

  Future<void> _onEntered(RoomEntered event, Emitter<RoomState> emit) async {
    emit(state.copyWith(status: RoomStatus.joining, roomId: event.roomId));
    try {
      await _repo.joinRoom(event.roomId);
      emit(state.copyWith(status: RoomStatus.waitingPartner));

      // Realtime 구독 — 이벤트로 변환 (룰 #5)
      await _participantSub?.cancel();
      _participantSub = _repo.watchParticipants(event.roomId).listen(
        (participants) => add(RoomParticipantChanged(participants)),
        onError: (e, st) => addError(e, st),  // BlocObserver → Sentry
      );
    } on NetworkError {
      emit(state.copyWith(status: RoomStatus.error));
      rethrow;
    }
  }

  // ...

  @override
  Future<void> close() async {
    await _participantSub?.cancel();      // 룰 #5 강제 (dispose 시 해제)
    return super.close();
  }
}
```

**핵심.**
- Realtime stream = Bloc 안 `StreamSubscription` 으로 구독
- stream 이벤트 → `add(Event)` 로 변환 (Bloc 상태 흐름에 편입)
- `close()` 오버라이드에서 `cancel()` 필수 (누수 · setState after unmounted 방지)
- 에러 = `addError()` 로 Bloc onError 발동 · BlocObserver 가 Sentry 보고

---

## 5. Realtime 구독 안전 패턴 (룰 #5)

**금지.**
```dart
class _RoomScreenState extends State<RoomScreen> {
  @override
  void initState() {
    super.initState();
    _channel = supabase.channel('room:1')..subscribe();   // ❌ UI 에서 직접 구독
  }
  // dispose 에서 unsubscribe 놓치면 누수
}
```

**허용.**
```dart
// Bloc 안 구독 · dispose 자동 해제
class _RoomScreenState extends State<RoomScreen> {
  @override
  void initState() {
    super.initState();
    context.read<RoomBloc>().add(RoomEntered(widget.roomId));
  }
  // Bloc.close() 가 자동으로 cancel
}
```

---

## 6. Repository 인터페이스 패턴

```dart
// lib/data/repositories/room_repository.dart (abstract)
abstract class RoomRepository {
  Future<Room> getRoom(String id);
  Stream<List<Participant>> watchParticipants(String roomId);
  Future<void> joinRoom(String id);
  Future<void> leaveRoom(String id);
}

// lib/data/repositories/room_repository_impl.dart
class SupabaseRoomRepository implements RoomRepository {
  SupabaseRoomRepository(this._client);
  final SupabaseClient _client;

  @override
  Future<Room> getRoom(String id) {
    return wrapPostgrest(() async {
      final row = await _client.from('rooms').select().eq('id', id).single();
      return Room.fromJson(row);
    });
  }

  @override
  Stream<List<Participant>> watchParticipants(String roomId) {
    return _client
        .from('room_participants')
        .stream(primaryKey: ['id'])
        .eq('room_id', roomId)
        .map((rows) => rows.map(Participant.fromJson).toList());
  }
  // ...
}
```

**abstract 이유** — Bloc 테스트 시 mock 주입 용이. 다른 백엔드 (예: REST → GraphQL) 로 교체 유연성.

---

## 7. 검증

- `flutter test` — Bloc unit 테스트 (`bloc_test` 활용)
- Repository 테스트 = 실 Supabase dev 붙임 (mock/real 정합 · [[project-widget-test-supabase-issue]] 방지)
- Realtime 화면 QA — 진입·이탈·재진입 시 구독 정합 확인

---

## 관련 룰

- 룰 #3 — UI → Bloc → Repository 순서
- 룰 #5 — Realtime 구독 해제
- 룰 #6 — 에러 삼키지 않음 (`bloc-observer-sentry.md`)
