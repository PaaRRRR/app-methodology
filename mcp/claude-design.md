# claude-design MCP — 프로토타입 → Flutter 포팅

> Claude 디자인 MCP 를 활용해 프로토타입 (React jsx) 을 생성하고, 이를 Flutter 로 포팅하는 flow.
> 룰 #8 실현 — 시각·동작·애니메이션은 프로토타입이 진실.

---

## 1. 활용 순서 (Phase 1 · Design)

1. **`docs/03_screens/{화면}.md` 준비** — 구조·상태·엣지케이스 정의
2. **Claude 디자인 MCP 호출** — 프로토타입 생성 요청
3. **iteration** — 시안 여러 개 → 사용자 선택 · 수정
4. **`design/prototype/{화면}.jsx` 저장** — 프로토타입 = 시각·동작 단일 진실
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
- 색·간격·폰트 값 = 프로토타입 코드에서 실측

---

## 3. Flutter 포팅 원칙 (룰 #8)

- **시각 · 동작 · 애니메이션은 프로토타입 코드의 실측값을 읽어 그대로 포팅** — 추측 X, 새 디자인 X
- 색 · 간격 · 폰트 · duration · easing curve 모두 프로토타입 값
- 프로토타입 vs `docs/03_screens/{화면}.md` 충돌 시 —
  - **시각 = 프로토타입**
  - **동작 흐름 · Bloc 연결 = md 문서**

---

## 4. 프로토타입 iteration

- 사용자 검토 후 여러 회차 iteration 가능
- 최종 확정 프로토타입만 `design/prototype/` 에 커밋
- 폐기 시안은 삭제 (git history 로만 남김)

---

## 5. QA 프로세스 (Phase 3 · Launch)

배포 전 QA 시 프로토타입 vs Flutter 실측 비교 필수. 상세 — `docs-templates/launch/qa-test.template.md` · StartRoom `_qa/visual_qa.md`.

---

## 6. Tips

- **프로토타입은 React 로** — Flutter 로 미리 만들면 새 디자인 감각 놓침. React 로 실측 파악 후 Flutter 포팅
- **애니메이션 curve · duration 명시** — Flutter Motion 토큰 (`stack/flutter/rule-enforcement/design-tokens.md`) 과 매핑
- **다크/라이트 이중 모드** — 프로토타입에서도 mode 스위치 구현 (실 대비 · WCAG AA 확인)
- **다크 고정 예외 (공유 카드 · QR 등)** — 프로토타입에서도 다크 고정으로 그림

---

## 7. StartRoom 사례

- `design/prototype/*.jsx` — 전 화면 프로토타입
- v1.0.3 브랜드 컬러 대개편 — 프로토타입에서 라임 accent · 다크/라이트 이중 mode 실측 후 Flutter 포팅
- 폐기 시안 (코랄 `#FF8A65` · 크리미 옐로 등) 은 iteration 도중 제거
