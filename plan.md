# app-methodology — 상위 구상 (초기)

> 이 저장소 = StartRoom 제작에서 검증된 knowledge (기획·디자인·개발·툴·워크플로) 를 다음 앱들에서 재사용 가능한 template 으로 문서화한 것.
> 첫 문서 (본 plan.md) 는 지도 원칙 · 로드맵 · 회차 계획.
> 진입 — 2026-09-15

---

## §0 결정

### 저장 형식
- GitHub repo 하나 (`app-methodology`). 코드 boilerplate repo 별도 안 만듦
- 이유 — 다음 앱 제작 시 Claude Code 가 문서 참고해서 새로 스캐폴딩하면 충분. 실 코드 repo 는 stale 위험, 문서는 재활용 유연
- stack 종속 자산 — `stack/{name}/` 하위에 코드 블록 형태 문서 (예: `stack/flutter/`). 다음 앱이 다른 스택이면 새 폴더 추가
- **StartRoom 저장소는 손대지 않음** — reference 구현체로만 존재. 인벤토리 회차에서 read-only 스캔 대상

### 지도 원칙
1. **무겁지 않게** — 폴더·문서 수 최소. 넣을지 뺄지 애매하면 뺀다
2. **헷갈릴 소지 없게** — 사용법·역할이 한눈에 명확한 것만 남긴다
3. **실 사용된 것만 문서화** — StartRoom 에서 실제로 활용된 자산만 승격. "쓰면 좋을 것" 은 필요해질 때 추가 (YAGNI)
4. **브랜치 최소** — 초기 단계는 main 직접 작업. 문서가 커지고 회차가 명확해지면 그 때 chore 브랜치 도입. epic 개념은 정말 필요할 때만

### 실 사용 현황 (StartRoom 지금 시점 파악, 회차 1 에서 정밀 인벤토리)
- **실 사용** — CLAUDE.md 절대 규칙 11개, `docs/00~08.md` 기획 문서, `docs/work/{회차}/` workflow, 프로토타입 → Flutter 포팅, Supabase MCP (dev), Sentry MCP, git flow (chore/feat/fix/refactor/epic + sub-work), Amplitude catalog, 메모리 시스템
- **미사용** — Claude Code Skill (slash command). 회차 진입은 대화 기반
- **부분 사용** — Firebase App Distribution (설정만), Amplitude funnel 대시보드 (raw event 만 박음)
- → **미사용 자산은 template 에서 제외.** 필요해지면 그때 추가

---

## §1 로드맵 (회차 4개)

각 회차는 별도 세션. 이 저장소 초기 단계 = main 직접 작업, 회차 성숙하면 chore 브랜치.

### 회차 1 — 인벤토리
- StartRoom `lib/`, `docs/`, `supabase/`, `.github/`, root 설정 파일 전수 read-only 스캔
- 각 자산에 대해 3분류 — **승격** (다음 앱에도 유효) / **제외** (StartRoom 도메인) / **판단 유보** (일반화 검토 필요)
- 결과물 — 이 저장소 안 `inventory.md` (분류표 + 승격 시 필요한 일반화 작업 노트)
- **lynch-pin 회차** — 이후 모든 회차의 재료

### 회차 2 — 승격 이관
- 회차 1 인벤토리의 **승격** 항목만 이 저장소로 이관. **판단 유보** 는 회차 3 에서 필요해지면 그때
- 폴더 구조는 인벤토리 결과에 맞춰 최소로 (§0 원칙 1·2·3 적용)
- 각 문서에 StartRoom 사례를 예시 블록으로 첨부 (추상 설명만은 재현 어려움)
- README.md — "다음 앱 시작 순서" 명시 · v0.1.0 tag

### 회차 3 — dry-run (다음 신규 앱)
- 다음 신규 앱 하나를 methodology 기반 시범 착수 (다음 앱 저장소에서 진행, 이 저장소와 별개)
- 목적 = "다음 앱 완성" 이 아니라 "template gap 발견"
- 착수 D1~D3 정도 · 놓친 문서·룰·자동화 목록 (`gaps.md`) 을 이 저장소에 수집

### 회차 4 — back-port + v1.0.0
- 회차 3 gap 을 이 저장소에 back-port · v1.0.0 정식 tag
- (선택) methodology v1.0.0 확정 후 startroom_app CLAUDE.md·docs 를 최신본 대비 sync — 필요해지면 그 시점에 startroom_app 저장소에서 별개 세션으로 처리

---

## §2 폴더 구조

**지금 그리지 않음.** 회차 1 인벤토리 결과 기반으로 회차 2 에서 확정.

이유 — 미리 그리면 (a) 그 그림 채우려는 관성 발생 (b) 실 사용 검증 안 된 폴더 추가 위험 (c) §0 원칙 1·2 (무겁지 않게 · 헷갈릴 소지 X) 위배.

---

## §3 이 저장소 초기 세팅 (본 세션 액션)

- [x] `~/project/app-methodology/` 폴더 생성
- [x] 본 plan.md 작성
- [ ] 사용자 최종 확인
- [ ] `git init` · 첫 commit (`plan.md`)
- [ ] GitHub push 는 사용자 원할 때 (지금은 로컬만)

**다음 세션 = 회차 1 인벤토리.** startroom_app 을 read-only 로 스캔하고 결과물은 이 저장소 `inventory.md` 로.
