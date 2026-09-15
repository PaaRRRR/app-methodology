# 01. 기술 스택

> 확정된 기술 스택 + 실제 프로젝트 설정.
> 대부분 앱마다 동일 (Flutter + Supabase baseline). 도메인 특화만 추가·수정.

---

## 1. 스택 한눈에 보기 (baseline · 대부분 그대로 사용)

| 영역 | 선택 | 비고 |
|---|---|---|
| **Framework** | Flutter (3.35+ 안정) | Dart 단일 코드베이스, iOS+Android |
| **State** | BLoC + Cubit | 복잡=Bloc · 단순=Cubit · 익숙한 패턴 · 학습 곡선 낮음 |
| **Routing** | go_router | Deep link · redirect 표준 |
| **Models** | freezed + json_serializable | 불변 모델 · copyWith · sealed |
| **Backend** | Supabase | Postgres + Realtime + Auth + Edge Functions |
| **Backend MCP** | Supabase MCP | Claude Code 에서 dev 직접 DB 작업 · prod 는 CLI |
| **Auth** | Supabase 익명 인증 | `signInAnonymously()` · 사용자에게 로그인 화면 안 보임 · V2 승급 옵션 |
| **Push** | FCM 직접 구현 | 하나로 iOS+Android · OneSignal 등 락인 없음 |
| **Local 알림** | flutter_local_notifications | 시각 리마인더 |
| **Analytics** | Amplitude | 한국 자료 많음 · 무료 한도 큼 (10M/월) · Funnel 표준 |
| **Crash** | Sentry | Day 1 부터 · 상시 보고 |
| **Deep Link** | app_links | Universal Link · Android App Link 표준 |
| **디자인** | Claude 디자인 (별도 도구) | 프로토타입 React jsx 산출 · Claude Code MCP 로 read |

---

## 2. 도메인별 추가 (필요 시 · placeholder)

| 영역 | 예시 패키지 | 언제 |
|---|---|---|
| QR 생성 | qr_flutter | QR 초대·공유 필요 |
| QR 스캔 | mobile_scanner | QR 진입 필요 |
| 미디어 공유 | share_plus · image_gallery_saver | 공유 카드 · 이미지 저장 |
| 오디오 | audioplayers | 사운드 재생 |
| 권한 | permission_handler · app_settings | 특수 권한 필요 |
| 특수 UI | sliver_tools 등 | iOS UITableView 스타일 sticky 헤더 등 |

**도메인 없으면 넣지 않음** (룰 · 절대 금지 #5 · 미리 만들지 않음).

---

## 3. V2+ 도입 예정 (MVP 제외)

| 영역 | 선택 | 시점 |
|---|---|---|
| Subscription | RevenueCat | V2 |
| Ads | AdMob | V3 (데이터 보고 결정) |

MVP 는 수익화 OFF. DB 필드 + 코드 분기점만 미리 심어둠. 상세 — `06_revenue_model.md`.

---

## 4. Baseline `pubspec.yaml`

상세 코드·버전 — `stack/flutter/config-baseline.md`. 새 앱 착수 시 그대로 복붙.

---

## 5. .env 구조

`.env.dev` / `.env.prod` / `.env.example` — flavor 별 분리. 상세 규칙 — `08_dev_ops.md` §1.

```
SUPABASE_URL={URL}
SUPABASE_ANON_KEY={KEY}
SENTRY_DSN={DSN}
AMPLITUDE_API_KEY={KEY}
FIREBASE_...={KEY}
```

**`.env*` 는 git 미포함** (룰 · 절대 금지 #1). `.env.example` 만 커밋 (필드 이름만).

---

## 6. Domain (Universal Link 용)

- 앱 도메인 — {DOMAIN} (예: startroom.live)
- iOS `applinks:` · Android `assetlinks.json` 두 방향 · `app_links` 패키지가 처리

---

## 7. MCP 활용

- **Backend MCP (Supabase)** — dev 만 MCP · prod 는 CLI. 상세 — `mcp/supabase-dev-vs-prod.md`
- **Claude 디자인** — 프로토타입 산출물 read. 상세 — `mcp/claude-design.md`
- **Sentry MCP** — issue triage flow. 상세 — `mcp/sentry.md`

---

## 8. 변경 이력 (원안 → 현재)

원안에서 바뀐 것 있으면 `07_decisions_log.md` 에 D-XX 로 기록.

**StartRoom 사례.**
- Riverpod → BLoC (D-01) · 익숙한 패턴 · 학습 곡선
- Magic Link → 익명 인증 (D-02) · 로그인 화면 없이 시스템 인증
- PostHog → Amplitude (D-03) · 한국 자료 · Funnel 표준
- OneSignal → FCM (D-04) · 락인 없음
- Subscription / Ads → V2/V3 이연 (D-19)

---

## 9. 예시 (StartRoom)

> StartRoom 은 위 baseline + 도메인 특화 (qr_flutter · mobile_scanner · app_links · audioplayers · image_gallery_saver · share_plus · sliver_tools) + Pretendard 폰트.
