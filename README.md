# app-methodology

> StartRoom 제작에서 검증된 knowledge (기획·디자인·개발·툴·워크플로) 를 다음 앱들에서 재사용할 template 로 문서화한 저장소.
> 사용자 실 작업 흐름을 그대로 재현 가능한 형태.

---

## 이 저장소 사용법 (다음 앱 시작 시)

1. **`milestone-order.md` 먼저 읽기** — 시간순 이정표 (Docs → Design → Code → 배포 → 운영)
2. **`CLAUDE.md.template` 복사** — 새 앱 저장소 root 에 `CLAUDE.md` 로. placeholder 치환 (`{APP_NAME}`, `{ONE_LINER}` 등)
3. **`docs-templates/product/00~08.template.md` 복사** — 새 앱 저장소 `docs/` 에. Docs-first 로 앱 코드 없이 문서부터 채움
4. **Phase 별 이동** — 각 phase 완료 시 `milestone-order.md` 의 다음 phase 로. 배포 후엔 `docs-templates/ops/` 강조
5. **매 회차 진입 시 `workflow/session-workflow.md` 참조** — plan → 진입 commit → 수행 → analyze+test → 사용자 OK → 의미 단위 commit → 머지

---

## 폴더 구조

```
├── milestone-order.md              # 시간순 이정표 (여기서 시작)
├── CLAUDE.md.template              # 프로젝트 CLAUDE.md 원형 (11개 절대 규칙)
├── docs-templates/
│   ├── product/                    # (Phase 0) Docs-first — 00~08
│   ├── launch/                     # (Phase 3) 배포 준비
│   ├── ops/                        # (Phase 4) 배포 후 — amplitude · beta-observations
│   └── legal/                      # 개인정보·약관
├── workflow/                       # 회차 workflow · git flow · 명명 규칙
├── mcp/                            # MCP 활용 (claude-design · supabase-dev-vs-prod · sentry)
└── stack/flutter/                  # Flutter 스택 특화
    ├── rule-enforcement/           # 절대 규칙 실현 코드 예시 (룰별)
    └── config-baseline.md          # pubspec · analysis_options · launcher_icons
```

---

## 원칙 (지도)

1. **무겁지 않게** — 폴더·문서 수 최소. 넣을지 뺄지 애매하면 뺀다
2. **헷갈릴 소지 없게** — 사용법·역할이 한눈에 명확한 것만 남긴다
3. **실 사용된 것만 문서화** — StartRoom 에서 실제로 활용된 자산만. "쓰면 좋을 것" 은 필요해질 때 추가 (YAGNI)
4. **StartRoom = reference 구현체** — 이 저장소가 다루지 않는 상세는 startroom_app 직접 참고

---

## 진행 상황

- **v0.1.0** — 회차 2 이관 중 (2026-09~)
- **v1.0.0** — 회차 4 back-port 완료 후 (신규 앱 dry-run 반영)
- 상세 로드맵 — `plan.md`

---

## 관련 저장소

- **StartRoom (reference 구현체)** — `/Users/haeincho/project/app/startroom_app`
- 다음 신규 앱 — 회차 3 dry-run 대상 (미정)
