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

## 4. 도구 활용

- **Backend (Supabase)** — dev 만 MCP · prod 는 CLI. 상세 + tips — `08_dev_ops.md` §4
- **Design** — Claude 디자인 프로토타입 → Flutter 포팅. 상세 — CLAUDE.md 룰 8 · `04_design_system.md` · `/docs-templates/launch/qa-test.template.md`
- **Observability (Sentry)** — Day 1 통합 · issue triage flow. 상세 — `08_dev_ops.md` §5 · `/stack/flutter/rule-enforcement/bloc-observer-sentry.md`

---

## 5. 외부 서비스 계정 체크리스트

개발 시작 전 준비할 계정·프로젝트 (앱 무관 baseline · 도메인 특화만 조정).

- [ ] Backend (Supabase 등) — dev / prod 프로젝트 각 1개
- [ ] Firebase — dev / prod 프로젝트 (FCM + App Distribution)
- [ ] Analytics (Amplitude 등) — dev / prod 앱 키
- [ ] Crash (Sentry) — 프로젝트 1개 (environment 로 dev/prod 구분)
- [ ] Apple Developer — 계정 ($99/년 · 푸시 인증서 + 배포)
- [ ] Google Play Console — 계정 ($25 1회)
- [ ] **도메인** — {DOMAIN} (예: startroom.live) · DNS (Cloudflare 권장) + well-known 호스팅. Universal Link — iOS `applinks:` · Android `assetlinks.json` 두 방향 · `app_links` 패키지 처리
- [ ] 랜딩 페이지 호스팅 — Vercel 무료 등

---

## 6. 무료 티어 한도 (참고)

| 서비스 | 무료 한도 | 초과 시 |
|---|---|---|
| Supabase | 500MB DB · 1GB 파일 · 2GB egress | Pro $25/월 |
| Firebase FCM | 무제한 | — |
| Firebase App Distribution | 무료 | — |
| Amplitude | 월 10M 이벤트 | 유료 플랜 |
| Sentry | 월 5k errors | 유료 플랜 |

> Beta (수백 명) 규모에서는 모두 무료 티어로 충분.

---

## 7. Baseline `pubspec.yaml`

상세 코드·버전 — `/stack/flutter/config-baseline.md`. 새 앱 착수 시 그대로 복붙.

---

## 8. .env 구조

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

## 9. 변경 이력

원안에서 바뀐 것 있으면 `07_decisions_log.md` 에 D-XX 로 기록.

---

## 10. 예시 (StartRoom)

StartRoom 은 §1 baseline + 도메인 특화 (qr_flutter · mobile_scanner · app_links · audioplayers · image_gallery_saver · share_plus · sliver_tools) + Pretendard 폰트.

**주요 변경 이력 —**
- Riverpod → BLoC (D-01) · 익숙한 패턴 · 학습 곡선
- Magic Link → 익명 인증 (D-02) · 로그인 화면 없이 시스템 인증
- PostHog → Amplitude (D-03) · 한국 자료 · Funnel 표준
- OneSignal → FCM (D-04) · 락인 없음
- Subscription / Ads → V2/V3 이연 (D-19)

---

## 참조 경로 표기 원칙 (문서 전체 공통)

- **product/ 안 서로 참조** — 파일명만 (예: `08_dev_ops.md`)
- **product/ 밖 참조** — repo 루트 기준 (예: `/stack/flutter/config-baseline.md`)
- CLAUDE.md 룰은 "CLAUDE.md 룰 {번호}" 로
