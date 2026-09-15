# session-workflow — 회차 진행 방식

> 매 작업 회차 (session) 시작·수행·commit·머지 흐름.
> 회차 = 하나의 논리적 완결 단위 (예: "홈 화면 UI 완성", "매칭 알고리즘 튜닝").

---

## 1. 회차 진입 (Start)

1. **새 브랜치 생성** (룰 #10) — main 직접 커밋 금지
   - `chore/{설명}` (설정) · `feature/{설명}` (기능) · `fix/{설명}` (버그) · `refactor/{설명}` (리팩)
2. **`docs/work/{category}/{topic}-{yyyy-mm}/plan.md` 작성** — 명명 규칙 `workflow/docs-work-naming.md`
   - `§0 결정` — 회차 진입 시 확정 사항 (사용자 OK 받은 것 lock-in)
   - `§1~ 진행 단계` — 섹션 번호 · 확정 문구
3. **관련 문서·코드 read** — plan 작성 전 진실 확인
4. **진입 commit** — plan.md 만 담아 첫 commit (`chore(scope): {topic} plan`)

---

## 2. 회차 수행 (Work)

- **plan §1 → §2 → ...** 순차 진행
- **§ 사이 commit 하지 않음** — 회차 내 working tree 에 누적 후 마지막 단계에서 의미 단위 분할 commit
- **UI 시각 회귀** — 사용자 검토 여러 번 필요할 때는 회귀 통과 시점까지 commit 보류
- **에러 발생 시** — 실 에러 메시지·스택 트레이스 read · 추정 fix 금지

---

## 3. 회차 마무리 (Verify · Commit · Merge)

### 3-A. Verify
- `flutter analyze` 통과
- `flutter test` 통과 (변경된 영역)
- 시각 작업 시 사용자 회귀 OK
- **통과 전 commit 절대 금지**

### 3-B. Commit
- 사용자 명시 OK 받고 commit
- 의미 단위 분할 — 하나의 commit = 한 문장 요약 가능
- Conventional Commits — `feat: · fix: · refactor: · chore: · docs: · test:`
- Co-Authored-By 태그 (Claude 활용 시)

### 3-C. Merge
- main 머지 (fast-forward 또는 --no-ff 정책 지정)
- 브랜치 삭제 (`git branch -D {branch}`)

---

## 4. Multi-session Topic (epic + sub-work)

**언제** — 2+ 회차가 함께 배포돼야 정합인 topic (예: 브랜드 컬러 대개편 · 홈 IA 재설계).

**구조**
- `epic/{topic}` — main 파생
- sub-work — epic 파생 (`feature/{topic}-{sub}` 등)
- sub → epic 머지 (통합 검증 여지)
- epic → main 은 한 번에 (모든 sub 완료 후)

**주의** — 논리적 프로젝트가 여러 저장소 걸치면 각 저장소 별로 epic 판단 (물리적으로 브랜치는 저장소 단위).

---

## 5. 검증 없이 commit 금지 (룰)

- `analyze` + `test` 통과 없이 commit 금지
- 시각 작업 시 사용자 회귀 OK 없이 commit 금지
- 사용자 명시 승인 없이 commit/머지/브랜치 삭제 금지

---

## 6. context-notes 는 optional

- plan.md 안에 결정 배경 요약이 있으면 skip
- 필요 시 (사용자 명시 요청 · 큰 회차) `context-notes.md` 별도 작성

---

## 7. 회차 완료 후

- 다음 회차 진입 시 이 문서 참조
- 회차별 산출물 = `docs/work/{category}/{topic}-{yyyy-mm}/` 안 (plan · checklist · 산출물 · pending)
- 완료 회차는 rename·이관 안 함 (git blame 보존)
