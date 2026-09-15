# git-flow — 브랜치 · 커밋 · 머지 정책

> 룰 #10 실현. 회차 워크플로 (`session-workflow.md`) 의 git 파트 상세.

---

## 1. 브랜치 네이밍

| prefix | 용도 | 예시 |
|---|---|---|
| `chore/` | 설정 · 스캐폴딩 · 문서 · 릴리스 준비 | `chore/env-setup` · `chore/release-1.0.3` |
| `feature/` | 새 기능 · 새 화면 | `feature/home-screen` · `feature/matching-pool-rpc` |
| `fix/` | 버그 수정 | `fix/room-timer-sync` · `fix/d16-notif-followup` |
| `refactor/` | 리팩토링 (동작 변경 없음) | `refactor/repository-signatures` |
| `epic/` | 통합 브랜치 (2+ 회차 함께 배포) · main 파생 | `epic/brand-color` |

**모두 kebab-case.** 회차명은 `{prefix}/{topic}-{yyyy-mm}` 권장 (예: `feature/matching-pool-rpc-2026-08`).

---

## 2. 회차별 단일 브랜치 원칙

- 한 이슈 여러 fix 시도 → 새 브랜치 파지 않음. 한 브랜치에 누적 후 해결 시점 commit + 머지
- QA 회차에서 여러 다른 이슈 fix → 이슈별 브랜치 X · 한 fix 브랜치에 이슈별 commit 누적
- 회차 마무리 시점에 의미 단위 분할 commit

---

## 3. Epic + sub-work (multi-session topic)

**언제** — 2+ 회차 함께 배포해야 정합인 topic.

**구조**
```
main
 └── epic/{topic}                         (main 파생)
      ├── feature/{topic}-{sub1}          (epic 파생)
      ├── feature/{topic}-{sub2}
      └── ...
```

**흐름**
1. `git checkout main && git checkout -b epic/{topic}`
2. `git checkout epic/{topic} && git checkout -b feature/{topic}-{sub1}`
3. sub 완료 → sub → epic 머지 (`--no-ff` 권장, 통합 검증 여지)
4. 모든 sub 완료 · epic 에서 통합 검증
5. epic → main 머지 (한 번에)
6. epic · sub 브랜치 삭제

**epic 은 long-lived 가능** — 문서 작업 · 배포 사이클 안에서만 유지.

**여러 저장소 걸치는 topic** — git branch 는 저장소 단위. 저장소별 epic 판단. 논리적 통합은 umbrella 문서로.

---

## 4. Commit 메시지 (Conventional Commits)

```
feat: {새 기능}                예: feat: 홈 화면 예약 카드 추가
fix: {버그 수정}               예: fix: room timer sync race condition
refactor: {리팩토링}           예: refactor: repository interface 단일화
chore: {설정 · 잡일}           예: chore: pubspec upgrade
docs: {문서}                   예: docs: 07_decisions_log D-19 추가
test: {테스트}                 예: test: room bloc entry event 커버
```

**Scope 선택** — 여러 영역 걸치면 생략, 하나면 명시 (예: `feat(home): ...`).

**본문 · 한국어** — 왜 이 변경인지 · 어떤 영향인지 · 관련 이슈/결정 링크.

**Co-Authored-By** — AI 활용 시 하단에 태그.

---

## 5. 커밋 시점

- **의미 단위** — 한 문장 요약 가능한 완결 변경
- **회차 § 사이 X** — §2 → §3 → §4 다단계는 마지막에 몰아 분할 commit
- **UI 회귀 반복** — 통과 시점까지 누적
- **진입 commit** — plan.md 만 담는 첫 commit 은 별개 (`chore(scope): {topic} plan`)

---

## 6. Merge 정책

- **sub → epic** — `--no-ff` 권장 (통합 검증 이력 남김)
- **epic → main / feature → main** — fast-forward or `--no-ff` 프로젝트 정책 따름
- **삭제** — 머지 후 `git branch -D {branch}` (원격 있으면 `git push origin --delete {branch}`)

---

## 7. 절대 금지

- main 직접 commit 금지
- `--no-verify` (hook skip) 금지 — 사용자 명시 요청 없으면
- prod 관련 브랜치 force push 금지
- `.env*` commit 금지
- 통과 안 한 코드 (analyze/test 실패) commit 금지
