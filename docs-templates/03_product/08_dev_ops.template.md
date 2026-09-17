# 08. 개발 운영 (Dev Ops)

> Git · 테스트 · 환경 분리 · Supabase MCP · 모니터링.
> Week 1 D1 (환경 셋업) 의 기준. 개발 시작 첫날 이 문서대로 셋업.

---

## 1. 환경 분리 (dev / prod)

flavor 로 두 환경을 완전 분리. 한 기기에 둘 다 설치 가능.

| 항목 | dev | prod |
|---|---|---|
| App ID | {APP_ID}.dev | {APP_ID} |
| 앱 이름 | {APP_NAME} Dev | {APP_NAME} |
| 아이콘 | 다른 색·마크 (구분) | 정식 |
| Supabase | dev 프로젝트 | prod 프로젝트 |
| Firebase | dev 프로젝트 | prod 프로젝트 |
| Sentry env | development | production |
| Amplitude | dev 프로젝트 (세팅 검증용) | prod 프로젝트 (실 데이터) |
| Xcode Config | Debug-dev / Release-dev | Debug-prod / Release-prod |

**원칙.**
- 베타 테스터는 prod 만 받음. dev 데이터 미노출
- dev 는 마음껏 데이터 리셋 가능 → 데이터 마이그레이션 불필요
- prod 는 실험·테스트 금지 (룰 · 절대 금지 #2)

### 진입점 구조
```
lib/
├── entry/
│   ├── entry.dev.dart    # dev 부트스트랩 · main() { bootstrap(env: AppEnv.dev); }
│   └── entry.prod.dart   # prod 부트스트랩
└── app/
    └── app.dart          # 공통 앱 (환경만 다르게 주입)
```

상세 코드 — `/stack/flutter/rule-enforcement/flavor-setup.md`

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

Xcode Build Phases Run Script — flavor 별 Firebase 설정 파일 복사. `Runner/config/dev/` · `Runner/config/prod/` 두 폴더에 각각 `GoogleService-Info.plist` 두고 아래 script 실행.

```bash
case "${CONFIGURATION}" in
  "Debug-dev"|"Release-dev"|"Profile-dev" )
    cp -r "${PROJECT_DIR}/Runner/config/dev/GoogleService-Info.plist" \
          "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/GoogleService-Info.plist" ;;
  "Debug-prod"|"Release-prod"|"Profile-prod"|"Debug"|"Release"|"Profile" )
    cp -r "${PROJECT_DIR}/Runner/config/prod/GoogleService-Info.plist" \
          "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/GoogleService-Info.plist" ;;
  *) echo "warning: no Firebase config for CONFIGURATION=${CONFIGURATION}, skipping" ;;
esac
```

> **안전망.** 기본 `Debug/Release/Profile` config (Xcode scheme 실수 시 사용) 도 prod Firebase 로 fallback. §7 사고 방지 원칙 실현.

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

### `.gitignore` 필수 항목

```
.env*                             # 환경 변수 (절대 커밋 금지)
!.env.example                     # 예시는 커밋
*.keystore                        # Android 서명 키
**/google-services.json           # dev/prod 분리 관리
**/GoogleService-Info.plist       # dev/prod 분리 관리
ios/Pods/
.dart_tool/
build/
*.g.dart                          # (선택) 생성 파일 — 팀 정책
*.freezed.dart                    # (선택)
```

> `.g.dart` / `.freezed.dart` 커밋 여부 = 선택. 1인 개발이면 커밋 안 하고 `build_runner` 로 재생성이 깔끔. CI 도입 시 커밋이 편할 수 있음.

상세 workflow — `/workflow/session-workflow.md` · `/workflow/git-flow.md`

---

## 3. 테스트 전략

**원칙 — 위험한 곳만.** 회귀 위험 큰 자리에 집중 · 커버리지 숫자 목표 X.

| 자리 | 도구 | 언제 |
|---|---|---|
| Repository / Data Layer | `mocktail` 단위 테스트 | Supabase 호출 · 도메인 로직 · 회귀 위험 큰 자리 |
| BLoC / Cubit | `bloc_test` | 복잡한 state machine · 단순 Cubit 은 skip |
| 위젯 · 통합 | 수동 QA | 자동화 대비 시간 효율 낮음. 베타 사용자가 통합 테스터 |

**commit 전 필수** — `flutter analyze` + `flutter test` (자동 실행). 시각 작업 시 사용자 회귀 OK.

---

## 4. Supabase MCP (dev 만)

**원칙.**
- **dev = MCP `apply_migration` 단일 채널** · prod = Supabase CLI `db push` (수동)
- 변경 전 `02_architecture.md` 스키마와 정합 확인 · 변경 후 sync

### 4-A. 마이그레이션 적용 채널 (dev)

**dev = MCP `apply_migration(name, query)` 단일 채널.** Dashboard SQL Editor 는 ad-hoc SELECT / EXPLAIN 전용 · DDL 은 SQL Editor 에서 치지 않음.

**이유.** Dashboard SQL Editor 로 DDL 직접 실행 시 `supabase_migrations` 메타 테이블에 record 안 남음. MCP `list_migrations` 가 실제 DB 상태와 어긋나 다음 회차에서 "적용 안 됐다고 착각해 재실행" 또는 "prod cutover 시 누락" 사고 발생.

**회차 closeout 절차.**
1. 작성한 migration 파일이 dev DB 에 실 적용됐는지 컬럼/enum SELECT 1회 확인
2. MCP `list_migrations` 응답과 파일명 sequence 일치 확인. 어긋나면 그 회차에서 정정
3. 어긋남 발견 시 = 이미 적용된 DDL 은 그대로 · 메타 정렬은 prod cutover 시점에 별도 회차

### 4-B. 마이그 관리 tips

- **표기명** — `{NNN}_{snake_case_설명}.sql` (숫자 3자리 zero-pad · 순차 증가). 겹침 방지 원칙
- **하나의 migration = 하나의 논리 변경** — 여러 테이블·목적 섞지 않음
- **회귀 정정** — 이전 migration 결과 뒤집을 때는 이전 파일 편집 X · 새 파일 추가 (이력 보존)
- **MCP `list_migrations` 누락 이슈** — apply 했는데 list 에 안 나타나는 케이스 있음. list 누락 ≠ DB effect 없음. 실 DB `pg_dump` 또는 Dashboard SQL Editor 로 재확인

### 4-C. 보안 · Edge Functions tips

- **SECURITY DEFINER 함수 `search_path` 고정** — Advisor WARN 뜨면 필수. `SET search_path = public, pg_temp` 명시
- **anon EXECUTE 회수** — helper 함수 (`is_{scope}_member` 등) 는 `authenticated` · `service_role` 만 EXECUTE. `anon` 은 REVOKE
- **Edge function CORS** — 앱에서 호출 시 `Access-Control-Allow-Origin` 헤더 필수
- **service_role key** — env 로만 주입 · 코드 하드코딩 금지
- **Edge function JWT 검증** — 사용자 요청은 authenticated JWT verify 후 진행
- **pg_net trigger → edge function** — DB trigger 에서 edge function 호출 시 pg_net 확장 활용

### 4-D. MCP 인증 모드 — PAT vs OAuth

hosted Supabase MCP 인증 모드 2가지.

- **PAT (`Authorization: Bearer sbp_...`)** — CI 환경 전용. hosted 서버가 read-only 강제 · `apply_migration` 거부. `?read_only=false` 붙여도 무시. read 류 (`list_migrations` · `execute_sql` SELECT) 만 가능
- **OAuth (dynamic client registration)** — `.mcp.json` 의 `headers` 통째로 제거 → Claude Code 재시작 시 브라우저 OAuth flow 1회 → `database:write` scope 승인 → write 가능

**`.mcp.json` 권장 형태 (dev).**
```json
{
  "mcpServers": {
    "supabase-dev": {
      "type": "http",
      "url": "https://mcp.supabase.com/mcp?project_ref=<DEV_PROJECT_REF>"
    }
  }
}
```

dev 작업 = OAuth default. PAT 는 CI/배치 전용. OAuth 토큰은 macOS Keychain 등 secure storage 저장 + refresh token 자동 갱신.

---

## 5. 모니터링

### 5-A. Sentry (Day 1 부터)

- **Day 1 통합** — 개발 첫날부터 · 개발 중 모든 에러 잡힘
- **환경 분리** — `SENTRY_ENVIRONMENT` = `development` / `production` · dev 이슈는 prod alert 대상 X
- **Release 태그** — `{version}+{build}` (예: `1.0.4+18`)
- **소스맵 · deobfuscation** — Flutter Release 는 자동 obfuscation · Sentry CLI 로 debug symbols 업로드 (배포 pipeline 통합)
- **초기 검증** — 통합 후 의도적 test error 발생 · 대시보드 도착 확인
- **BLoC 통합** — `bloc_observer` 에서 Bloc `onError` 시 Sentry 보고 (룰 #6). 상세 — `/stack/flutter/rule-enforcement/bloc-observer-sentry.md`

**Issue triage flow.**
1. Alert 수신 (Slack · email · in-app)
2. 스택 트레이스 · 이벤트 컨텍스트 read
3. dev 에서 재현 시도
4. `fix/{issue-slug}` 브랜치 · 회차 진입
5. 회귀 테스트 추가
6. release 시 Sentry release version 태깅

**Alert 우선순위.**
- **P0 (즉시)** — 크래시율 급증 · 특정 crash 다수
- **P1 (일간)** — 새 에러 유형 · 특정 flow 실패 급증
- **P2 (주간)** — 트렌드 분석 · 개선 여지

`mcp__sentry__*` 도구는 issue 조회·재현 편의 자리 · 활용은 필요 시.

### 5-B. Analytics (Amplitude)

**dev/prod 프로젝트 완전 분리.** 실 사용자 분석 데이터는 prod 만 · dev 는 세팅·이벤트 발화 검증용.

- **dev 프로젝트** — SDK 통합 · event 도착 확인 · taxonomy 검증. 실 대시보드 관측 X (사용자 데이터 없음)
- **prod 프로젝트** — 실 사용자 event · funnel · retention 대시보드 자리
- **super property `flavor` = dev/prod** 자동 첨부 (`/docs-templates/06_ops/amplitude-taxonomy/catalog.template.md` §Super Properties). prod 대시보드에서 `flavor=prod` 필터로 이중 안전
- Day 1 raw event 박음. funnel 정의는 배포 후 대시보드. 배포 후 관측 · 트리아지 — `/docs-templates/06_ops/`

---

## 6. 배포 절차

- **베타** — TestFlight (iOS prod) + Firebase App Distribution (Android dev/prod)
- **정식** — App Store · Play Store 심사 · Phased/Staged rollout
- 상세 — `/docs-templates/05_launch/prod-deploy-guide.template.md`

### 6-A. 베타 채널 원칙

UDID 등록 없이 이메일 초대만으로 가동 가능한 채널 조합.

- **iOS TestFlight 내부** — 심사 X · 즉시 초대 (최대 100명 · App Store Connect 사용자 등록). Ad-Hoc Distribution 은 UDID 등록 부담 · 회피
- **iOS TestFlight 외부** — 첫 빌드 심사 24-48h · 이후 이메일/링크 초대 자유 (최대 10,000명)
- **Android Firebase App Distribution** — FCM 프로젝트 그대로 활용 · `.aab` / `.apk` 업로드 · 테스터 수 제한 없음. Play Console 공개 테스트 (12명/14일) 는 정식 출시 직전 요건 · 베타 단계엔 Firebase 로 충분

### 6-B. FCM 푸시 인프라 runbook

repo 밖 사용자 수동 작업. dev / prod 각각 반복.

**A. Apple Developer (한 번만, dev/prod 공통).**
1. Certificates, Identifiers & Profiles > **Keys** > [+] > APNs 체크
2. **.p8 파일 다운로드** (한 번만 · 분실 시 revoke 후 재발급)
3. 기록 — **Key ID** + **Team ID** (Membership 좌상단)
4. **보관** — 1Password 등 secure storage + 백업 (iCloud Drive · 별도 클라우드). 디바이스 분실 = 키 분실 · 재발급 유일
5. Auth Key (.p8) 는 **단일 키로 Sandbox + Production 둘 다 인증**. dev/prod 분리 발급 X
6. Bundle ID (dev + prod) 둘 다 **Push Notifications** capability 활성화

**B. Firebase Console (dev / prod 각각).**
1. **APNs Key 등록** — Cloud Messaging > Apple app configuration > Upload (같은 p8 재사용 OK)
2. **service account JSON 다운로드** — Service accounts > Generate new private key

**C. Supabase Dashboard (dev / prod 각각).**
1. **Edge Function secret 박기** — `FCM_SERVICE_ACCOUNT_JSON` (JSON 전체 본문 raw string paste)
2. **placeholder 치환 migration 은 Dashboard SQL Editor 로 적용** (MCP 로 하면 plaintext key 가 migration history 박힘 회피)
3. **cron 등록** — 앱의 알림 트리거 (예: 이벤트 알림 · 리마인더 · 스케줄 등 · 앱마다 다름)

**D. 검증 (수동 QA).**
- dev 디바이스 2대 · 앱별 주요 알림 시나리오 (앱마다 다름) push 도착 확인
- foreground / background / killed 3분기 push 표시 + 탭 라우팅
- Edge Function 로그 확인 (UNREGISTERED / OAuth 교환 / 200 OK)
- 권한 거부 fallback (push_token null 시 발사 skip)

---

## 7. 사고 방지 · 재발 방지 (StartRoom 경험)

- **`--dart-define-from-file` 누락 크래시** — 빌드 스크립트에 명시 · CI 검증
- **iOS Run Script `Debug/Release/Profile` fallback** — Xcode scheme 실수 시에도 prod Firebase 로 (앱 이름 · Firebase 정합)
- **APNs Key Environment = Sandbox & Production** — 하나만 체크 시 TestFlight/App Store 빌드 푸시 실패 (BadEnvironmentKeyInToken). Apple Developer Portal 확인 필수

---

## 8. 핫픽스 우선순위

배포 후 발견 이슈 트리아지.

| 등급 | 예 | 대응 |
|---|---|---|
| **P0** | 크래시 · 핵심 flow 완전 차단 | 24h 내 |
| **P1** | 핵심 flow 부분 장애 | 2-3일 내 |
| **P2** | UX 어색함 · 카피 수정 | 다음 정기 빌드 |
| **P3** | 개선 제안 | 백로그 |

**피드백 채널.** 앱 안 "문의하기" → 이메일/폼. 베타는 1:1 카톡 병행.

---

## 9. 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/08_dev_ops.md`. flavor 완전 분리 (live.startroom.app / live.startroom.app.dev) · iOS Run Script Firebase 분기 · Android productFlavor 자동 · TestFlight prod / Firebase App Distribution dev 이원화 · 1.0.2+11 hotfix (env 주입 누락) 등 실사고 이력.
