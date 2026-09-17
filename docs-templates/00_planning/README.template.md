# docs-templates/00_planning — Phase 0-1 (기획 · 와이어프레임)

> **성격 — 초심자 진입 낮추는 scaffold + 자유 형식 유연성 · 완료된 사용자는 skip 가능.**
> 폴더 안 `plan.template.md` · `wireframe.template.md` 를 신규 앱 저장소 `docs/planning/` 에 복사해 채우거나 · 참고만 하고 자유 형식으로 작성하거나 · 이미 있으면 이관 후 이어서.

---

## 사용 방식 (3가지 유연 · 원하는 방식 선택)

### 방식 A. Scaffold 채우기 (초심자 · 권장)

1. `plan.template.md` → 신규 앱 저장소 `docs/planning/plan.md` 복사
2. `wireframe.template.md` → `docs/planning/wireframe.md` 복사
3. § 구조 따라 placeholder 치환 · § 순서 자유 변경 · § 추가/삭제 OK
4. 각 § 끝 "예 (StartRoom)" 인용 참고 · 자기 앱 맞게 재작성

**장점 —** 놓칠 항목 없음 · 초심자 즉시 작성 시작 · § 구조 학습됨.

### 방식 B. 자유 형식 (숙련자)

Scaffold 무시하고 자유 markdown 으로 `docs/planning/plan.md` · `docs/planning/wireframe.md` 작성.

**장점 —** 사고 넓힘 · 앱 특화 자유 · 형식 강제 없음.
**단점 —** 놓칠 수도 있음 · Phase 3 정합화 시 항목 누락 발견 가능.

### 방식 C. 이미 있음 → 이관 후 skip

Phase 0-1 이 이미 다른 자리 (Notion · 별도 저장소 · Google Docs) 에 있으면 —
- `docs/planning/` 로 이관 (Markdown 화 · git 이력 시작)
- Phase 2 (디자인) 부터 진행

**이관하지 않고 참조만 유지도 OK** — 다만 유실 위험. Git 이관 권장.

---

## Phase 0 — 기획

**목적** — "무엇을 · 누구를 위해 · 왜" 확정. 제품 vision · 핵심 가설 · MVP 범위 · 수익 모델 방향 · 데이터 구조 초안.

**Scaffold** — `plan.template.md` (§0~11 · 각 § 안내 + StartRoom 인용 예시)

**주요 § 미리보기 —**
- §0 제품 정의 (한 줄 · 포지셔닝 · 피칭)
- §1 핵심 문제 (2-3개 · 붕괴 패턴)
- §2 결과값 · 와우모먼트
- §3 핵심 가설
- §4 전체 앱 흐름
- §5 화면 목록 (초안 · 개수·이름·MVP)
- §6 MVP 범위 (Must · Should · Not now)
- §7 수익 모델 방향
- §8 핵심 지표
- §9 데이터 구조 초안
- §10 리스크 · 불안요소
- §11 핵심 개발 원칙 (앱별 5-7개)

**완료 기준** — Phase 1 (와이어프레임) 착수 가능한 정도로 제품·flow 확정.

---

## Phase 1 — 와이어프레임

**목적** — Phase 0 기획을 화면 단위로 쪼갬. ASCII 스케치 · 손그림 · 다이어그램 무엇이든. 시각 정확성 X · 화면 목록·flow·상태 정의 O.

**Scaffold** — `wireframe.template.md` (화면 목록 · 공통 원칙 · 각 화면 · flow)

**주요 § 미리보기 —**
- 화면 목록 (표 · 라우트 초안 · 상태 · MVP 여부)
- 공통 원칙 (헤더 · CTA · 카피 톤)
- 각 화면 (목적 · 진입 · 상태 · ASCII · 인터랙션 · 엣지)
- 주요 user flow (신규 · 정상 · 이탈)

**완료 기준** — Phase 2 (디자인) 착수 가능한 정도로 화면·상태·flow 명확.

---

## 위치 권장 — `docs/planning/`

신규 앱 저장소 안 `docs/planning/plan.md` · `docs/planning/wireframe.md` 권장 이유 —

1. **git 이력** — 기획·와이어프레임 변경이 commit 으로 남음
2. **코드 저장소와 함께** — 개발자가 한 저장소만 열면 됨
3. **docs-template 과 자연 분리** — `docs/planning/` = 탐색 원본 · `docs/00~08.md` = 확정 정합화 (Phase 3)
4. **참조 쉬움** — Phase 3 정합화 시 `docs/planning/` 참조하며 `docs/00~08.md` 채움

**대안 (덜 권장)** —
- 별도 저장소 (`~/project/{app}-planning/`) — 두 저장소 관리 부담
- Notion · Obsidian — Git 밖 · 유실 위험 · Git 로 이관 권장

---

## 참고 원본 예시 (StartRoom)

두 층 참고 자료 —

### 층 1. Scaffold 안 짧은 인용

`plan.template.md` · `wireframe.template.md` 각 § 끝에 "예 (StartRoom): 3-5줄 인용" 이 이미 담김. 각 § 채울 때 바로 옆에서 참고.

### 층 2. examples/ 폴더 전문

Scaffold 인용만으론 서술 깊이·톤 전달 부족. `examples/` 안 실 완성된 문서 전문 열람 —

- `../examples/00_planning/startroom-plan.md` (본 기획서 v2.0 · 18KB) — 제품·문제·가설·와우·MVP·수익·지표·데이터·원칙 상세
- `../examples/00_planning/startroom-guide.md` (요약본 · 4KB) — 앱 개요·화면 목록·디자인 시스템 핵심 (첫 훑기용)
- `../examples/00_planning/startroom-wireframe.md` (v1.1 · 31KB) — 15개 화면 ASCII 상세

**⚠️ 참조 전용 · 그대로 복사 금지.**
- 각 예시 파일 첫 줄에 강력 경고 헤더.
- StartRoom 특화 도메인 (익명 매칭 · 시작방 · 30초 스크럼 · 10분 세션 · Guide 봇) 걷어내고 구조·톤·서술 깊이만 참고.
- 그대로 두면 StartRoom clone 이 됨. 자기 앱 사용자·문제·가설로 처음부터 새로 작성.

**왜 examples/ 를 저장소에 포함했나 —**
- Fork 시 참고 가능 (원래 StartRoom 원본은 `~/Downloads/` 로컬 자리 · fork 자에게 접근 불가).
- 각 § 짧은 인용만으론 "이 정도 깊이·톤으로 쓰는구나" 감 부족 · 완성된 문서 통째로 읽어야 감 옴.

---

## 다음 Phase 이동

Phase 0-1 확정 후 —
- **Phase 2** — `docs-templates/02_design/README.template.md` 읽고 Claude 디자인 MCP 진입
- 프로토타입 (React jsx) 생성 · 확정 시 `design/prototype/{screen}.jsx` export
