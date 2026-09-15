# 03. 화면 명세

> 화면별 상세 명세 (라우트 · 상태 · UI 컴포넌트 · 엣지케이스).
> **주의** — 시각·동작·애니메이션은 프로토타입 (`design/prototype/{화면}.jsx`) 이 진실. 여기는 구조·상태·BLoC 연결·엣지케이스만 담당 (룰 #8).

**구조.** `docs/03_screens/{screen-name}.md` 개별 파일. 이 template 은 개별 화면 파일의 원형.

---

## {SCREEN_NAME} 화면

### 1. 라우트
- Path — `/{path}`
- 진입 — {진입 조건 · 이전 화면}
- 이탈 — {가능한 이탈 경로}

### 2. Bloc / Cubit
- 이름 — `{ScreenName}Cubit` (또는 `{ScreenName}Bloc`)
- 판단 — {단순 Cubit · 복잡 Bloc}
- 의존 — {Repository 목록}
- Realtime 구독 — {있음 · 없음 · 어떤 채널}

### 3. 상태 (State)
```dart
// {ScreenName}State 필드
class {ScreenName}State {
  final Loading/Loaded/Error 등 sealed 또는 Equatable
  // ...
}
```

### 4. UI 컴포넌트 (프로토타입 참조)
- 프로토타입 — `design/prototype/{screen-name}.jsx`
- 사용 SR 위젯 — {목록}
- 화면 전용 위젯 — `features/{screen-name}/widgets/` 안

### 5. 엣지케이스
- 빈 상태 — {UI · 카피}
- 로딩 상태 — {SR loading view · skeleton}
- 에러 상태 — {SR error view · 재시도}
- 오프라인 — {처리 방식}
- 권한 거부 — {처리 방식}
- {도메인 특수 케이스}

### 6. Analytics 이벤트
- 화면 진입 — `{screen_name}_viewed`
- 액션 — `{action_name}` (property)
- 상세 — `docs/work/amplitude-taxonomy/catalog.md` (배포 phase)

### 7. Deep link (해당 시)
- URL 패턴 — `https://{DOMAIN}/{path}?{params}`
- 처리 — `app_links` · `router.dart` redirect

### 8. 관련 결정 이력
- {D-XX 결정 링크}

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/03_screens/home.md`, `room.md`, `matched.md` 등. 화면별 개별 파일. 각 파일이 이 template 구조를 따름.
> 프로토타입 진실 원칙 (룰 #8) 은 StartRoom 에서도 엄격히 적용됨.
