# 01. 기술 스택

> 확정된 기술 스택과 실제 프로젝트 설정 (pubspec.yaml, .env) 을 담는다.
> 초기 원안에서 변경된 항목은 ⚠️ 로 표시.

---

## 1. 스택 한눈에 보기

| 영역 | 선택 | 비고 |
|---|---|---|
| Framework | {FRAMEWORK} | {NOTE} |
| State | {STATE} | {NOTE} |
| Routing | {ROUTING} | {NOTE} |
| Models | {MODELS} | {NOTE} |
| Backend | {BACKEND} | {NOTE} |
| Backend MCP | {BACKEND_MCP} | Claude Code 통합 여부 |
| Auth | {AUTH} | {NOTE} |
| Push | {PUSH} | {NOTE} |
| Local 알림 | {LOCAL_NOTIF} | {NOTE} |
| Analytics | {ANALYTICS} | {NOTE} |
| Crash | {CRASH} | {NOTE} |
| Deep Link | {DEEPLINK} | {NOTE} |
| Subscription | {SUB} | {WHEN} (예: V2 도입, MVP 제외) |
| Ads | {ADS} | {WHEN} |
| Domain | {DOMAIN} | Universal Link 용 |

> 버전은 개발 시작일 기준 최신 안정 버전 확인 후 고정.

---

## 2. 왜 이렇게 골랐나 (변경 사유)

### {영역_1} — {원안} → {선택}
{이유}

### {영역_2} — {원안} → {선택}
{이유}

---

## 3. pubspec.yaml (baseline)

> Flutter 기준. 실 baseline 은 `stack/flutter/config-baseline.md` 참조. 여기는 앱 도메인별 추가·제거만.

```yaml
name: {APP_SLUG}
description: {ONE_LINER}
publish_to: 'none'
version: 0.1.0+1

# 자세한 baseline — stack/flutter/config-baseline.md
# 도메인 추가 예시 —
# - qr_flutter (QR 생성 필요 시)
# - mobile_scanner (QR 스캔 필요 시)
```

---

## 4. .env 구조

`.env.dev` / `.env.prod` / `.env.example` — flavor 별 분리. 자세한 규칙 — `08_dev_ops.md` §1.

```
SUPABASE_URL={URL}
SUPABASE_ANON_KEY={KEY}
SENTRY_DSN={DSN}
AMPLITUDE_API_KEY={KEY}
FIREBASE_...={KEY}
```

---

## 5. MCP 활용

- **Backend MCP (Supabase 등)** — `mcp/supabase-dev-vs-prod.md` 참조. dev 만 MCP, prod 는 CLI
- **디자인 MCP** — `mcp/claude-design.md` 참조
- **Sentry MCP** — `mcp/sentry.md` 참조

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/01_tech_stack.md`. Flutter + Supabase + BLoC + FCM + Amplitude + Sentry 스택.
> 변경 이력 (Riverpod → BLoC · Magic Link → 익명 · PostHog → Amplitude · OneSignal → FCM) 은 `07_decisions_log.md` 참조.
