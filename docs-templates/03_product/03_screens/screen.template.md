# 03. 화면 명세 · {SCREEN_NAME}

> **성격 — 개발 handoff 확정본 (Phase 3 산출물).** 기획·와이어프레임·디자인 phase 확정 후 채운다. 와이어프레임 X · 디자인 스케치 X · **상태 · 인터랙션 · 데이터 · BLoC 연결 · 엣지케이스** 개발 진실 자리.
> 파일 위치 — `docs/03_screens/{screen-name}.md` (화면당 1개). 화면 목록·IA·flow 는 `docs/03_screens/README.md`.
> 시각·동작 진실 — `design/prototype/{screen-name}.jsx` (Phase 2 Claude 디자인 산출물). md 는 md 담당 파트만 · 시각은 프로토타입 참조.

---

### 1. 화면 목적 · 사용자 goal

한 문장으로 — {WHO} 가 {WHAT} 을 하기 위한 화면.

세부 —
- 사용자가 이 화면에서 달성하려는 것 (task) 1~3개
- 이 화면이 없으면 어떤 문제가 생기는지

### 2. 진입 조건 · 이탈 조건

**진입.**
- 어느 화면에서 · 어떤 사용자 액션 · 어떤 조건에서 이 화면 도달
- 딥링크·푸시 진입 여부 (있으면 URL 패턴)

**이탈.**
- 정상 이탈 (완료 · 다음 화면 이동)
- 취소 이탈 (뒤로 · X · dismiss)
- 자동 이탈 (타임아웃 · 조건 변경)

### 3. 사용자 액션 목록

이 화면에서 사용자가 할 수 있는 액션 나열. 액션마다 —
- 액션 이름 (버튼 라벨 등)
- 액션 후 결과 (다음 상태 · 다음 화면 · 서버 호출 여부)
- 액션 조건 (활성 · 비활성 조건)

### 4. 상태 목록

이 화면이 가질 수 있는 상태.
- **loading** — 무엇을 기다리는가 · 예상 소요
- **empty** — 데이터 없을 때 UI · 카피
- **error** — 어떤 에러 유형 · 재시도 여부
- **success/loaded** — 정상 데이터 있을 때
- **{도메인 특수 상태}** — 예: waiting · ready · live · completed

각 상태 전환 조건 · 트리거 명시.

### 5. 엣지케이스

- **권한** — 필요 권한 · 거부 시 처리 · 재요청 flow
- **오프라인** — 네트워크 끊김 시 처리
- **데이터 없음** — 최초 진입 · 이전 삭제
- **동시 진행** — 다른 유저 · 다른 세션 영향
- **{도메인 특수 케이스}** — 예: 파트너 이탈 · 세션 만료

### 6. 화면 간 관계

이 화면과 연결된 다른 화면 목록.
- 이 화면 이전 화면들 (어떤 진입 경로들이 있는가)
- 이 화면 다음 화면들 (어떤 이탈 경로들이 있는가)
- 관련 sheet · modal · popup

### 7. 카피 톤 방향

이 화면 카피의 톤 · 지침.
- 감정 톤 (친근 · 진중 · 활기 · 절제 등)
- 사용자 호칭 (2인칭 vs 3인칭)
- 브랜드 카피 원칙 (있으면)
- 피해야 할 표현

### 8. 참고 competitor (있으면)

비슷한 목적 다른 앱 화면 · 링크 · 무엇을 참고할지·무엇은 다를지.

### 9. Analytics 이벤트 (초안 · 배포 phase 확정)

- 화면 진입 — `{screen_name}__view`
- 주요 액션 — `{screen_name}__click__{object}`
- 상세 — `docs/work/amplitude-taxonomy/catalog.md` (배포 phase)

### 10. 관련 결정 이력

- {D-XX 결정 링크 · 결정 근거}

---

## 개발 완료 후 sync (Phase 4 이후 채움)

- **프로토타입 파일** — `design/prototype/{screen-name}.jsx` (Phase 2 산출물 · Phase 3 진입 시 이미 존재 · 링크만)
- **UI 컴포넌트 목록** — 프로토타입 실측 기반 `{PREFIX}` 위젯 목록
- **BLoC/Cubit** — `lib/features/{screen-name}/` 개발 후 파일 링크
- **State dart 코드** — 개발 후 sealed class 링크

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/03_screens/home.md`, `room.md`, `matched.md` 등. 화면당 개별 파일.
> StartRoom 은 디자인·개발 완료 상태라 프로토타입·Bloc·State 세부까지 sync 되어 있음. 신규 앱은 Phase 3 에서 §1~10 만 먼저 채우고 §sync 는 Phase 4 후 갱신.
