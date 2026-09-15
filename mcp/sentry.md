# sentry — issue triage flow

> Sentry MCP 활용 · issue 발생 시 대응 순서.

---

## 1. 초기 setup (Day 1)

- Sentry 프로젝트 2개 (dev · prod)
- Flutter 통합 — `sentry_flutter` 패키지 · `SentryFlutter.init` 에서 dsn 주입 (env)
- `bloc_observer.dart` 에서 Bloc onError 시 Sentry 보고 (룰 #6)
- 초기 test error 로 도착 확인
- 상세 코드 — `stack/flutter/rule-enforcement/bloc-observer-sentry.md`

---

## 2. Issue 발생 시 flow

1. **Sentry alert 수신** — Slack · email · in-app
2. **MCP 로 issue 조회** — `mcp__sentry__search_issues` 또는 `analyze_issue_with_seer`
3. **스택 트레이스 · 이벤트 컨텍스트 read** — 룰 #6 (에러 삼키지 않음) 위반 아닌지 확인
4. **재현 시도** — dev 에서 같은 조건 만들어 재현
5. **fix 회차 진입** — `fix/{issue-slug}` 브랜치 · `docs/work/fix/{issue-slug}-{yyyy-mm}/plan.md`
6. **테스트 추가** — 회귀 방지
7. **release 시 Sentry release version 태깅** — 어느 버전에서 fix 됐는지 tracking

---

## 3. Alert 설정 (Ops phase)

- **P0 (즉시)** — 크래시율 급증 · 특정 crash 다수 발생
- **P1 (일간)** — 새 에러 유형 등장 · 특정 flow 실패 급증
- **P2 (주간)** — 트렌드 분석 · 개선 여지 발견

---

## 4. Environment 분리

- `SENTRY_ENVIRONMENT` = `development` / `production` (env 로 주입)
- dev 이슈는 prod alert 대상 아님
- release 태그 = `{version}+{build}` (예: `1.0.4+18`)

---

## 5. Source map · deobfuscation

- Flutter Release 빌드는 자동으로 stack trace obfuscation
- Sentry CLI 로 debug symbols 업로드 (배포 pipeline 에 통합)
- 상세 — Sentry Flutter docs

---

## 6. StartRoom 사례

- Day 1 부터 통합 · bloc_observer 에서 자동 보고
- `1.0.2+11` hotfix — flavor env 주입 실패 → LateInitializationError 크래시 · Sentry 로 즉시 인지 · hotfix
