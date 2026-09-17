# Store Listing — App Store + Google Play

> 콘솔 입력 직전 검수 · 카피 조정. 한국어 텍스트는 띄어쓰기/맞춤법 검사기 통과.
> **먼저 확정 —** bundle identifier · 심사 카테고리 (§7).
>
> **콘솔 —** App Store Connect (`appstoreconnect.apple.com`) · Play Console (`play.google.com/console`).

---

## 1. 앱 이름 + 부제

| 자리 | 한도 | draft |
|---|---|---|
| App Name (iOS/Android) | 30자 | **{APP_NAME} — {태그라인}** |
| App Subtitle (iOS only) | 30자 | **{SUBTITLE}** |
| Short description (Play) | 80자 | **{SHORT_DESC}** |

**원칙** — 한글 병기 (검색 노출) · 문제→해결 콜투액션 톤 · 이전 프레임 폐기 시 이력 남기기.

**Localizations —** 앱 이름 · 부제 · 긴 설명 · 키워드는 언어별. App Store Connect "Add Language" · Play Console "Store listings > Add translation". 한국어 primary + 영어 추가 시 §2 · §3 각각 언어별 draft.

---

## 2. 긴 설명 (App Store description + Play full description)

> 4000자 한도. 한국어 1500자 안쪽 권장.

```
{HOOK — 사용자 문제 재현하는 첫 두 줄}

{APP_NAME}({영문}) 은 {WHO} 가 {WHAT} 하도록 돕는 앱입니다.
{핵심 mechanic 요약 한 문장}

[ 어떻게 동작하나요 ]
1. {단계 1}
2. {단계 2}
...

[ 왜 {특징 1}인가요 ]
{근거 · 원칙}

[ 왜 {특징 2}인가요 ]
{근거}

[ 무엇이 쌓이나요 ]
{축적 · retention 서사}

[ 누구를 위한 앱인가요 ]
· {페르소나 1}
· {페르소나 2}

[ {부가 가치 섹션} ]
· {기능 1}
· {기능 2}

[ {클로징 · 브랜드 메시지} ]
{한 줄}
```

---

## 3. 키워드 (App Store only)

> 100자 한도, 쉼표 구분. 띄어쓰기 0.

```
{한글명},{영문명},{경쟁 카테고리 키워드},{페르소나 키워드},{기능 키워드}
```

---

## 4. Brand assets

`flutter_launcher_icons` 패키지 도입 + 단일 1024 PNG 으로 iOS Asset Catalog + Android 런처 일괄 갱신.

| 파일 | 용도 |
|---|---|
| `assets/branding/app_icon.svg` | 1024 원본 source |
| `assets/branding/app_icon_1024.png` | iOS Asset Catalog · Play Store icon (`image_path`) |
| `assets/branding/app_icon_foreground.svg` | Android adaptive foreground source |
| `assets/branding/app_icon_foreground.png` | Android adaptive foreground (`adaptive_icon_foreground`) |
| `assets/branding/feature_graphic.svg` · `.png` | Play Store feature graphic (1024×500) |
| `flutter_launcher_icons.yaml` | 런처 아이콘 설정 단일 진실 |

### Iteration 흐름

1. SVG 수정 (색 · 형태)
2. PNG 재생성 — `rsvg-convert -w 1024 -h 1024 {source}.svg -o {target}.png`
3. 런처 아이콘 일괄 갱신 — `dart run flutter_launcher_icons`
4. 시뮬레이터 재설치 → 시각 회귀
5. Play Store / App Store 콘솔 PNG 업로드

**앱 아이콘 vs 앱 내부 accent 의도적 분리 가능** — 아이콘 = 스토어 진열대 impact · 앱 내부 = 장시간 피로 회피. 참조 — `/docs-templates/product/04_design_system.template.md` §1-5.

---

## 5. 스크린샷 가이드

### 5-A. 사이즈 · 개수 spec

| 플랫폼 | 사이즈 | 개수 |
|---|---|---|
| iOS 6.7" | 1290 × 2796 | 3~10장 |
| iOS 6.5" | 1284 × 2778 | 3~10장 |
| iOS 5.5" | 1242 × 2208 | 3~10장 |
| Android Phone | 1080 × 1920 이상 | 2~8장 |
| Feature Graphic (Play) | 1024 × 500 | 1장 |

### 5-B. 각 장 자리 (5-7장 baseline)

첫 3장이 가장 중요 · 사용자가 스토어에서 훑을 때 이 3장으로 결정.

1. **문제 재현** — 첫 화면 · 사용자 문제 감정 훅
2. **핵심 mechanic** — 앱 진입 후 첫 action 자리
3. **결과 · 완주** — action 후 결과 화면 (와우모먼트)
4. **부가 가치 1** — 통계 · 기록 · 소셜 등
5. **부가 가치 2** — 브랜드 identity (공유 카드 · 배지 등)
6~7. **부가 flow** — 옵션

### 5-C. 촬영 방법 (Screenshot Launcher 화면)

수동 캡처는 매 심사마다 재촬영 부담. **`/dev/screenshots` 라우트 (dev flavor only)** 로 표준화.

- `lib/features/dev/screenshot_launcher_screen.dart` — 각 화면 진입 버튼
- 실행 — `flutter run --flavor dev -t lib/entry/entry.dev.dart --dart-define=DEV_INIT_ROUTE=/dev/screenshots`
- 각 버튼 → 시각 정합된 mock 상태 화면 → 시뮬레이터 스크린샷 (Cmd+S · iOS) · Android Studio emulator 캡처

**촬영 시점 —** main 머지 · 시각 회귀 통과 후. 심사 재제출 사이클마다 재촬영 (UI 변경 시).

### 5-D. Overlay 텍스트 · 파일명

- **Overlay 텍스트** — Figma · Canva 로 5분. 카피 톤 = 짧고 감정적 · 한 화면 한 메시지
- **파일명** — `screenshot_{NN}_{자리}.png` (예: `screenshot_01_home.png` · `screenshot_02_action.png`)
- **Localizations** — 영어 추가 시 별도 캡처 (`en/screenshot_01_home.png` · `ko/screenshot_01_home.png`)

---

## 6. 미리보기 영상 (선택)

- iOS App Preview — 15~30초 · 3장까지 · 시뮬레이터 화면 녹화 (Cmd+R)
- Play — 30초~2분 · YouTube 링크

---

## 7. 심사 카테고리

- Primary — {카테고리 1}
- Secondary — {카테고리 2}

---

## 8. 연령 등급

- Apple — 4+ / 9+ / 12+ / 17+ 중 선택 (UGC 있으면 12+ 권장)
- Google — IARC questionnaire 로 자동 (Everyone · Teen · Mature 등)

### Apple questionnaire (App Store Connect > Age Rating)

앱 성격에 따라 답변. Baseline 이 "None" · 실 앱에 맞게 조정.

| 항목 | 답변 (baseline) |
|---|---|
| Profanity or Crude Humor | None |
| Sexual Content or Nudity | None |
| Realistic Violence · Cartoon Violence | None |
| Horror/Fear · Gambling · Alcohol/Tobacco/Drug · Medical Info | None |
| **User-Generated Content** | UGC 있으면 Yes (Infrequent/Mild) · 없으면 No |

⚠️ **UGC 있는 앱만** — Reviewer notes 에 moderation SLA (24h 등) 강조 필수. Frequent 판정 시 자동 17+ 로 상향 가능.

### Google Play > Content rating > IARC

- **Social features · UGC · 익명 매칭 있으면** → 자동 **Teen** (13+). Everyone 10+ 로 낮추기 사실상 불가
- **툴 · 유틸 · 개인 사용 앱** → Everyone 가능

---

## 9. Reviewer notes (App Review Information)

App Store Connect "App Review Information" + Play Console "App content".

```
[ Test Account ]
{테스트 계정 안내 · 익명 인증이면 "자동 익명 생성"}

[ 동작 흐름 ]
1. {진입 → 온보딩 → 계정 생성 방식}
2. {핵심 flow 요약}
3. {상호작용 있으면 자리·방식 · 없으면 skip}
4. {세션/액션 흐름}
5. {완료·회고}

[ 권한 ]
· {권한 1} — {용도} · {거부 시 앱 사용 가능 여부}
· {권한 2} — {용도} · {거부 fallback}

[ UGC 정책 · 모더레이션 (UGC 있는 앱만 · 연령 등급 근거) ]
· UGC surface — {제한된 자리 명시 · 세션 단위 vs 지속 SNS}
· 상호작용 시간 — {bounded interaction 명시}
· 신고 — {메뉴 진입 방식 · report/block flow}
· 차단 — {차단 시 매칭 제외 · 필터 명세}
· 신고 접수 시 SLA — {24h 등 시간 명시}

[ 계정 삭제 ]
{설정 위치 · cascade 삭제 범위}

[ 연락처 ]
{support 이메일}
```

---

## 10. 개인정보 답변 (Apple App Privacy + Google Data Safety)

**두 콘솔 모두 필수 · 별개 자리.** 실 수집 항목은 동일 · 폼이 다름.

- **Apple** — App Store Connect > App Privacy > Data Types 선택 + 각 항목 사용 목적 (Analytics · App Functionality · Product Personalization 등)
- **Google** — Play Console > App content > Data safety

### 수집 항목 (Apple · Google 공통 baseline · 앱마다 조정)

| 항목 | 수집 여부 | 목적 | 필수/선택 | 사용자 식별 가능? |
|---|---|---|---|---|
| User IDs | Yes | 앱 기능 (계정) | 필수 | {No (익명 UUID) · Yes 등} |
| Push token (FCM) | Yes (알림 있으면) | 앱 기능 (알림) | 필수 | No |
| UGC — {콘텐츠 유형} | Yes (UGC 있으면) | 앱 기능 (사용자 기록) | {선택/필수} | {식별 가능 여부} |
| App activity — 제품 상호 작용 | Yes | 분석 (Amplitude) | 필수 | No (익명 이벤트) |
| App info — crash logs | Yes | 앱 기능 (Sentry) | 필수 | No (익명) |

### 수집 안 함 (명시 필요)

Personal info · Financial info · Health & fitness · Photos and videos · Audio files · Files and docs · Calendar · Contacts · Web browsing history · Device or other IDs (IDFA / Android ID) — 사용 안 함 자리.

### 광고 · 인앱 결제 답변 (Play Data Safety 필수)

- **광고 (Ads)** — {V1 = No · V3 도입 시 Yes (배너/Rewarded 유형 명시)}
- **인앱 결제 (In-app purchases)** — {V1 = No · V2 도입 시 Yes (구독)}
- **광고 tracking (IDFA/AD_ID)** — {ATT 미사용 시 No · 사용 시 Yes + 목적 명시}

### 공유

**0건.** 사용자 데이터 제3자 공유하지 않음 (기본 원칙).

### 보안 관행

- [x] Data is encrypted in transit — HTTPS/TLS
- [x] Users can request data deletion — 설정 > 계정 탈퇴 (앱 안)
- [ ] Independent security review — {V1 시행 여부}
- [ ] Committed to Google Play Families Policy — {유아·아동 대상 앱만 · 대상 아니면 N/A}

---

## 11. What's New (버전별 자리)

App Store Connect "What's New in This Version" + Play Console "출시 노트" · 500자 이내.

```
· {이번 릴리스 사용자 관점 변경 1}
· {변경 2}
· {변경 3}
```

**원칙 —** 내부 정합 (마이그레이션 · 리팩토링 등 사용자 관점 impact 낮음) 은 별도 언급 X. 사용자에게 보이는 것만.

**이전 이력 (참조용).**
- **{VERSION} ({YYYY-MM-DD} 배포)** — {요약}

---

## 12. 출시 진행 순서 (Phase 1~5)

첫 출시 시 순차 진행. 이후 재제출 시엔 변경 자리만.

### Phase 1 — 준비 (문서·자산)
- [ ] **베타 검증 완료 확인** — TestFlight (iOS) · Firebase App Distribution (Android) 로 실기기 QA 통과. 상세 — `/docs-templates/product/08_dev_ops.template.md` §6-A
- [ ] **legal URL 준비** — `docs/legal/{terms,privacy}` 콘텐츠를 landing site (`{DOMAIN}/terms` · `/privacy`) 에 배포. App Store `App Privacy` + Play `Privacy Policy URL` 둘 다 필수 (인앱 표시만으론 심사 통과 X)
- [ ] **Support 이메일** 확정
- [ ] **앱 아이콘** — 1024 원본 + `flutter_launcher_icons` 생성 (§4)
- [ ] **스크린샷** — Screenshot Launcher 로 촬영 · Overlay 텍스트 (§5)

### Phase 2 — 콘솔 메타 입력
- [ ] **App Store Connect 앱 생성** — Bundle ID · **App SKU** (내부 관리 ID · 재사용 X) · **Primary Language** · **가격** (Free/Paid) · **국가·지역** (Availability)
- [ ] **Play Console 앱 생성** — Package name · Primary Language · Free/Paid · 국가·지역
- [ ] **긴 설명 · 짧은 설명 · 키워드** 입력 (§2 · §3)
- [ ] **카테고리 · 연령 등급 questionnaire** 답변 (§7 · §8)
- [ ] **개인정보 답변** — Apple App Privacy + Play Data Safety (§10)
- [ ] **Reviewer Notes** 작성 (§9)
- [ ] **스크린샷 · Feature Graphic** 업로드
- [ ] **Play Console 폐쇄 테스트 12명 · 14일 요건** — 2023+ 정식 출시 전 필수 (`/docs-templates/product/08_dev_ops.template.md` §6-A)

### Phase 3 — Signing · 빌드
- [ ] **iOS Signing** — App Store **Distribution Provisioning Profile** + **Distribution Certificate** (Xcode automatic signing 활용 or 수동)
- [ ] **Android Signing** — **Play App Signing 활성화** (Google 서명 · 개발자는 upload key 만) + **keystore 관리** (분실 시 재발급 불가 · 별도 백업 필수)
- [ ] **What's New** 작성 (§11)
- [ ] iOS ipa 빌드 · Transporter 로 App Store Connect 업로드 · **정식 심사 별도 제출** (TestFlight 업로드만으로 정식 출시 X · TestFlight = 심사 없이 내부 100명 즉시 초대)
- [ ] Android aab 빌드 · Play Console 프로덕션 트랙 업로드

### Phase 4 — 심사 제출
- [ ] iOS 심사 제출 · **Manual** (심사 통과 후 개발자 수동 출시 · 출시일 조정) or **Automatic release** (심사 통과 즉시 출시) 선택
- [ ] Android 심사 제출 · Staged rollout 설정 (10% → 50% → 100%)

### Phase 5 — 롤아웃
- [ ] 심사 통과 확인 (iOS 24-48h · Android 4-24h 일반)
- [ ] iOS + Android 동시 수동 출시 (or 각자 롤아웃)
- [ ] Post-deploy 관측 (Sentry · Amplitude). 관측 protocol — `/docs-templates/ops/beta-observations/protocol.template.md` · 출시 후 첫 1주 flow — `/docs-templates/product/05_roadmap.template.md` Week 4

---

## 13. 심사 리젝 대응

**첫 출시 = 리젝 흔함.** 침착하게 대응.

### 자주 나오는 리젝 사유

- **UGC · 연령 등급 부족** — Reviewer notes 에 moderation SLA · 신고/차단 flow 강조 (§9). 대부분 재제출로 통과
- **개인정보 처리방침 URL 불명** — landing site 실 접근 확인 · 인앱 표시만으론 부족 (§12 Phase 1)
- **Data Safety 답변 불일치** — 실 수집 항목과 form 정합 (§10)
- **테스트 계정 접근 불가** — Reviewer notes 에 "익명 자동 생성" or 실 계정 명시 (§9)

### 재제출 흐름

1. **리젝 사유 정확히 읽기** — Apple/Google 심사 메시지에 사유 명시
2. **관련 § 자리 수정** — 카피 · 메타 · Reviewer notes 강화
3. **콘솔에서 응답** — Resolution Center (Apple) · Play Console 답변 (Google)
4. **재빌드 필요 여부 판단** — 메타 수정만 = 재빌드 X · 코드 수정 = buildNumber 증분 후 재업로드
5. **재제출** — 24-48h 대기

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/store-listing.md`. "혼자는 어렵죠. 파트너와 함께..." 훅 · "10분 시작방" mechanic · "왜 익명인가요" 섹션 · Productivity + Lifestyle 카테고리 · UGC (익명 파트너 채팅) 12+ · shareCardHues 6색 (§Brand assets 예).
