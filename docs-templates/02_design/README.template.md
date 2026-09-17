# docs-templates/02_design — Phase 2 (디자인 · Claude 디자인 MCP)

> **성격 — 디자인 phase 안내 자리.** Template 없음 · 실 산출물은 신규 앱 저장소 `design/prototype/{screen}.jsx`.
> 이 폴더는 "Phase 2 가 workflow 상 존재함" 을 저장소 구조로 보여주는 자리 + Hybrid 방식 안내.

---

## Phase 2 — 디자인 (Claude 디자인 MCP · 프로토타입)

**목적** — 화면 시각·동작을 프로토타입 (React jsx) 으로 완성. 이 프로토타입이 **시각·동작의 단일 진실.** Flutter 는 이 코드의 실측값을 읽어 포팅 (새로 디자인하지 않음).

**입력** — Phase 1 와이어프레임 (`docs/planning/wireframe.md` · 화면 목록 · 스케치 · 상태)
**출력** — `design/prototype/{screen}.jsx` × N (신규 앱 저장소)

---

## Hybrid 방식 (핵심)

**iteration 은 MCP · 확정 시점 local export.**

```
Phase 2 iteration      → Claude 디자인 MCP only · local 파일 없음 · 빠르게 갈아엎기
   ↓ (디자인 확정 시점)
Phase 2 완료 export    → design/prototype/{screen}.jsx 로 export · git commit
   ↓
Phase 3~ 이후          → Flutter · docs 는 local jsx 참조 · MCP 콜 안 필요
   ↓ (디자인 변경 필요 시)
MCP 재수정 · 재export  → 새 commit ("이 시점 디자인 변경" 명시)
```

**이 방식의 이유 —**

1. **git 이력** — 디자인 결정 (색·간격·컴포넌트 변경) 이 commit 으로 남음 · review 가능 · rollback · diff 지원
2. **구현 시점 스냅샷** — Flutter 코드는 특정 시점 디자인에 대응. MCP 가 다른 회차에서 변경돼도 local 은 그대로 · 사일런트 드리프트 방지
3. **MCP 의존 최소화** — Phase 4+ 개발 도중 MCP 콜 반복 X · Claude Code 가 local read 로 빠르게 진행
4. **오프라인 · grep** — MCP 콜 없이 확인. `grep AppColors design/prototype/` 같은 검색

**Iteration 부담 최소화 —**
- Phase 2 iteration 중엔 export 안 함 (MCP 만) · 부담 X
- 확정 시점에만 1회 export · 이후 sync 는 "명시적 변경 회차" 에만
- 자주 변경되면? = 아직 확정 안 됨 = Phase 2 미완료

---

## 순서 (권장)

1. **Phase 1 와이어프레임 read** — `docs/planning/wireframe.md` 화면 목록 · 상태 · flow
2. **Claude 디자인 MCP 에서 각 화면 프로토타입 생성** — 시각·상호작용·애니메이션 iteration
3. **디자인 확정 시점** — 각 화면 프로토타입 `design/prototype/{screen}.jsx` 로 export
4. **git commit** — 확정 프로토타입 스냅샷 저장 (예: `chore: design phase 2 확정 · N개 화면 프로토타입 export`)
5. **Phase 3 진입** — `docs-templates/03_product/` 참조하여 docs 정합화

---

## 위치 권장 — `design/prototype/`

신규 앱 저장소 안 `design/prototype/{screen}.jsx` 권장 이유 —

1. **저장소 root 아래 별도 폴더** — `docs/` (문서) 와 `lib/` (Flutter 코드) 와 대등 · 시각의 진실 자리
2. **파일당 화면 1개** — grep · diff 편리
3. **git 이력** — 위 Hybrid 이유 그대로

**파일 명명** — `{screen-name}.jsx` (kebab-case · 화면 이름 = docs/03_screens/{화면}.md 와 일치)

---

## 디자인 변경 정책

Phase 3+ 진입 이후 디자인 변경 필요 시 —

1. **MCP 재수정** — Claude 디자인 MCP 에서 iteration
2. **확정 → 재export** — 해당 `{screen}.jsx` 갱신
3. **git commit** — 변경 이유·범위 명시 (예: `refactor: home 화면 CTA 재배치 · 디자인 결정 D-XX`)
4. **관련 docs sync** — `docs/03_screens/{화면}.md` 상태·인터랙션·엣지 파트 갱신
5. **Flutter 코드 sync** — 실측값 재확인 · AppColors/AppSpacing 토큰 갱신 · 위젯 조정

**중요 —** 디자인 변경은 회차 단위로 진행. 사일런트 드리프트 (Flutter 는 안 바뀌었는데 프로토타입만 바뀜 · 또는 반대) 절대 X.

---

## `docs/03_screens/{화면}.md` 와 관계

- **`design/prototype/{화면}.jsx`** — 시각·동작 진실 (색·간격·애니메이션·상태 전환)
- **`docs/03_screens/{화면}.md`** — 개발 handoff 명세 (상태 목록 · 인터랙션 · 데이터 · BLoC 연결 · 엣지케이스)
- **충돌 시** — 시각·동작은 프로토타입 우선 · 동작 흐름·연결은 md 우선
- CLAUDE.md 룰 8 로 강제

---

## 대안 (덜 권장)

### MCP-only · local 없음

가능하지만 아래 단점 —
- git 디자인 이력 X (rollback · diff 불가)
- MCP 의존 (오프라인 · MCP 장애 시 blocking)
- 매번 MCP 콜 (개발 도중 반복 read)
- "이 Flutter 코드는 어느 디자인 버전에 대응?" 불명확

작은 앱 (5-10화면 이하) · 솔로 · MCP 안정적으로 계속 확보 가능하면 고려 가능.

---

## 참고 예시 (StartRoom)

StartRoom `design/prototype/` — 15개 화면 프로토타입. Claude 디자인 MCP 로 iteration 후 export. 이후 Flutter 포팅은 이 파일들의 실측값 기반. 변경 시점마다 git commit 으로 이력 유지.
