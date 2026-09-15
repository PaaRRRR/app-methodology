# 08. 개발 운영 (Dev Ops)

> Git · 테스트 · 환경 분리 · Backend MCP · 모니터링.
> Week 1 D1 (환경 셋업) 의 기준. 개발 시작 첫날 이 문서대로 셋업.
> 대부분 앱마다 동일. 앱 이름 · Bundle ID · 도메인만 placeholder.

---

## 1. 환경 분리 (dev / prod)

flavor 로 두 환경을 완전 분리. 한 기기에 둘 다 설치 가능.

| 항목 | dev | prod |
|---|---|---|
| App ID | {APP_ID}.dev | {APP_ID} |
| 앱 이름 | {APP_NAME} Dev | {APP_NAME} |
| 아이콘 | 빨강 계열 (구분) | 정식 |
| Backend | dev 프로젝트 | prod 프로젝트 |
| Firebase | dev 프로젝트 | prod 프로젝트 |
| Sentry env | development | production |
| Xcode Config | Debug-dev / Release-dev | Debug-prod / Release-prod |

### 진입점 구조
```
lib/
├── entry/
│   ├── entry.dev.dart    # dev 부트스트랩 · main() { bootstrap(env: AppEnv.dev); }
│   └── entry.prod.dart   # prod 부트스트랩
└── app/
    └── app.dart          # 공통 앱 (환경만 다르게 주입)
```

상세 코드 — `stack/flutter/rule-enforcement/flavor-setup.md`

### 빌드 명령
```bash
# ── 실행 (개발) ──────────────────────────────
flutter run --flavor dev -t lib/entry/entry.dev.dart --dart-define-from-file=.env.dev

# ── iOS ──────────────────────────────────────
# prod (스토어) — CLI 한 줄로 IPA 완료 (Xcode Organizer 불필요)
flutter build ipa --flavor prod -t lib/entry/entry.prod.dart --release --dart-define-from-file=.env.prod --export-options-plist=ios/ExportOptions.plist
# dev (Firebase Distribution)
flutter build ipa --flavor dev -t lib/entry/entry.dev.dart --release --dart-define-from-file=.env.dev --export-method=ad-hoc

# ── Android ──────────────────────────────────
# prod (스토어)
flutter build appbundle --flavor prod -t lib/entry/entry.prod.dart --release --dart-define-from-file=.env.prod
# dev (Firebase Distribution)
flutter build apk --flavor dev -t lib/entry/entry.dev.dart --release --dart-define-from-file=.env.dev
```

> **필수.** `--dart-define-from-file=.env.{flavor}` 없이 빌드하면 env 값이 빈 문자열이 되어 `Supabase.initialize()` 등 skip · 실기기 크래시. StartRoom 1.0.2+11 hotfix 사례.

### iOS · GoogleService-Info.plist 분기
Xcode Build Phases Run Script — flavor 별 Firebase 설정 파일 복사. 상세 — StartRoom `docs/08_dev_ops.md` §1 참고.

원칙 —
- 베타 테스터는 prod 만 받음. dev 데이터 미노출
- dev 는 마음껏 데이터 리셋 가능 → 데이터 마이그레이션 불필요
- prod 는 실험·테스트 금지 (룰 · 절대 금지 #2)

### Android · productFlavor
`android/app/src/dev/` · `android/app/src/prod/` 각각 `google-services.json` — productFlavor 자동 인식.

```gradle
android {
  flavorDimensions "env"
  productFlavors {
    dev {
      dimension "env"
      applicationIdSuffix ".dev"
      resValue "string", "app_name", "{APP_NAME} Dev"
    }
    prod {
      dimension "env"
      resValue "string", "app_name", "{APP_NAME}"
    }
  }
}
```

---

## 2. Git (룰 #10)

**브랜치 네이밍.**
- `chore/{설명}` — 설정 · 스캐폴딩
- `feature/{설명}` — 기능 · 화면
- `fix/{설명}` — 버그
- `refactor/{설명}` — 리팩토링
- `epic/{topic}` — 2+ 회차 함께 배포 정합 필요 시 · main 파생
- sub-work — epic 파생 (`feature/{topic}-{sub}` 등)

**Commit — Conventional Commits.**
```
feat: 새 기능       fix: 버그 수정
refactor: 리팩토링   chore: 설정/잡일
docs: 문서          test: 테스트
```

**원칙.**
- main 은 항상 빌드 가능 상태
- 작동 단위로 commit · 안 되는 코드 commit 금지
- 세션 끝나면 main 머지 + 브랜치 삭제
- `.env*` commit 금지 · `--no-verify` (hook skip) 금지

상세 workflow — `workflow/session-workflow.md` · `workflow/git-flow.md`

---

## 3. 테스트

- **BLoC 단위 테스트** — `bloc_test` 활용
- **Repository 테스트** — mock 없이 실제 Supabase dev 붙임 (mock/real 정합 · widget test 안 supabase getter assertion 실패 케이스 주의)
- **Widget 테스트** — 핵심 flow 위주
- **commit 전 필수** — `flutter analyze` + `flutter test`
- **시각 작업 commit 전** — 사용자 회귀 OK

---

## 4. Backend MCP (dev 만)

- 스키마 변경 · migration = MCP `apply_migration`
- 변경 전 `02_architecture.md` 스키마와 정합 확인
- 스키마 바꾸면 `02_architecture.md` 도 sync
- prod = Supabase CLI `db push` (수동)
- 상세 + tips (표기명 규칙 · list_migrations 누락 이슈 · SECURITY DEFINER · RLS 패턴 · soft delete 등) — `mcp/supabase-dev-vs-prod.md`

---

## 5. 모니터링

- **Sentry** — Day 1 부터 · 크래시 · 에러 즉시 인지 · env 별 분리 (`development` · `production`) · release 태그 (`{version}+{build}`)
- **Analytics** — Day 1 raw event 박음. funnel 정의는 배포 후 대시보드
- 배포 후 관측 · 트리아지 — `docs-templates/ops/`

---

## 6. 배포 절차

- **베타** — TestFlight (iOS prod) + Firebase App Distribution (Android dev/prod)
- **정식** — App Store · Play Store 심사 · Phased/Staged rollout
- 상세 — `docs-templates/launch/prod-deploy-guide.template.md` · `release-checklist.template.md`

---

## 7. 사고 방지 · 재발 방지 (StartRoom 경험)

- **`--dart-define-from-file` 누락 크래시** — 빌드 스크립트에 명시 · CI 검증
- **iOS Run Script `Debug/Release/Profile` fallback** — Xcode scheme 실수 시에도 prod Firebase 로 (앱 이름 · Firebase 정합)
- **APNs Key Environment = Sandbox & Production** — 하나만 체크 시 TestFlight/App Store 빌드 푸시 실패 (BadEnvironmentKeyInToken). Apple Developer Portal 확인 필수

---

## 8. 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/08_dev_ops.md`. flavor 완전 분리 (live.startroom.app / live.startroom.app.dev) · iOS Run Script Firebase 분기 · Android productFlavor 자동 · TestFlight prod / Firebase App Distribution dev 이원화 · 1.0.2+11 hotfix (env 주입 누락) 등 실사고 이력.
