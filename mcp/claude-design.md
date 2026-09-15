# claude-design — Claude 디자인 산출물 → Flutter 포팅

> Claude 디자인 (별도 도구) 에서 프로토타입 생성 · Claude Code 에서 MCP 로 산출물 read · Flutter 로 포팅.
> 룰 #8 실현 — 시각·동작·애니메이션은 프로토타입이 진실.

---

## 1. 활용 순서 (Phase 1 · Design)

1. **`docs/03_screens/` 디자인 브리프 준비** — 화면 목적 · 사용자 액션 · 상태 · 엣지케이스 · 카피 톤 (디자인 시 참고 재료)
2. **Claude 디자인에서 프로토타입 생성** — 별도 도구 (Claude 디자인 앱/웹) 에서 각 화면 프로토타입 (React jsx) 생성
3. **Claude Code 에서 MCP 로 산출물 read** — `mcp__claude-design__*` 도구 활용해 프로토타입 read · Flutter 포팅 재료로
4. **`design/prototype/{화면}.jsx` 저장** — 프로토타입 = 시각·동작 단일 진실 · 앱 저장소에 커밋
5. **`04_design_system.md` 확정** — 프로토타입 실측 팔레트·토큰 값 반영

---

## 2. 프로토타입 저장 관행

```
design/
├── prototype/
│   ├── home.jsx
│   ├── room.jsx
│   ├── stats.jsx
│   └── ...
└── (원본 HTML 번들 등 참고 자원)
```

- 각 화면 하나의 jsx 파일
- React 로 실 동작 · 상태 전환 · 애니메이션까지 구현
- 색 · 간격 · 폰트 값 = 프로토타입 코드에서 실측

---

## 3. Claude Code 에서 MCP 활용

세션 시작 시 사용 가능한 mcp__claude-design 도구 확인 후 —

- `mcp__claude-design__authenticate` · `complete_authentication` — 최초 1회
- Claude 디자인 산출물 read · Flutter 포팅 재료로

**포팅 원칙 (룰 #8).**
- 시각 · 동작 · 애니메이션 = 프로토타입 코드 실측값 그대로. 추측 X, 새 디자인 X
- 색 · 간격 · 폰트 · duration · easing curve 모두 프로토타입 값
- 프로토타입 vs `docs/03_screens/` 충돌 시 —
  - **시각 = 프로토타입**
  - **동작 흐름 · Bloc 연결 = md 문서**

---

## 4. 프로토타입 iteration

- Claude 디자인에서 여러 시안 · 사용자 검토 · 수정
- 최종 확정 프로토타입만 `design/prototype/` 에 커밋
- 폐기 시안은 삭제 (git history 로만)

---

## 5. QA 프로세스 (Phase 3 · Launch)

배포 전 QA 시 프로토타입 vs Flutter 실측 비교 필수. 상세 — `docs-templates/launch/qa-test.template.md` · StartRoom `_qa/visual_qa.md`.

---

## 6. Tips

- **React 로 만드는 이유** — Flutter 로 미리 만들면 새 디자인 감각 놓침. React 로 실측 파악 후 Flutter 포팅
- **애니메이션 curve · duration 명시** — Flutter Motion 토큰 (`AppDurations` · `AppCurves`, `stack/flutter/rule-enforcement/design-tokens.md`) 과 매핑
- **다크/라이트 이중 모드** — 프로토타입에서도 mode 스위치 · 실 대비 · WCAG AA 확인
- **다크 고정 예외 (공유 카드 · QR 등)** — 프로토타입에서도 다크 고정으로 그림
- **`04_design_system.md` sync** — 프로토타입 확정 후 팔레트·토큰 hex 값을 문서에 옮겨 코드 (`theme.dart`) 와 정합

---

## 7. StartRoom 사례

- `design/prototype/*.jsx` — 전 화면 프로토타입
- v1.0.3 브랜드 컬러 대개편 — Claude 디자인에서 라임 accent · 다크/라이트 이중 mode 실측 후 Flutter 포팅
- 폐기 시안 (코랄 `#FF8A65` · 크리미 옐로 등) iteration 도중 제거
