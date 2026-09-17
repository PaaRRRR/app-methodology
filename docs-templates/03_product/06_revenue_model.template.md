# 06. 수익모델

> 핵심 가치는 무료. 더 좋은 경험은 유료.
> **MVP (V1) 는 수익화 OFF.** 단, 나중에 켤 때 큰 공사 없도록 DB 자리와 코드 분기점만 미리 심어둔다.

---

## 1. 단계별 로드맵

| 단계 | 시점 | 내용 |
|---|---|---|
| **V1 (MVP)** | 지금 | 구독·광고 없음. DB 자리 + 코드 분기점만 |
| **V2** | 출시 1-2개월 후 | 구독만. 광고 없음 |
| **V3** | 출시 2-3개월 후 (데이터 보고 결정) | 광고 추가 |

V2 에 구독만 먼저 켜는 이유 — 구독 전환율을 광고 영향 없이 깨끗하게 측정. 한 달 데이터 보고 광고 여부·시점 판단.

---

## 2. V2 — 구독 ({APP_NAME} Plus)

> 출시 후 "더 쓰고 싶은데 막힌다" 신호 쌓이면 켠다.

### 가격
- 월 ₩{MONTHLY} / 연 ₩{YEARLY} (약 {DISCOUNT}% 할인)
- {TRIAL_DAYS} 일 무료 트라이얼
- 벤치마크 — {COMPETITOR_PRICE}

### Free vs Plus
| 항목 | Free | Plus |
|---|---|---|
| {DOMAIN_QUOTA} | {LIMIT} | 무제한 |
| {FEATURE_1} (기록/통계 등 · 있으면) | {LIMITED} | 전체 + 인사이트 |
| {FEATURE_2} | {BASIC} | {ENHANCED} |

### 페이월 트리거 (어디서 구독 권함)
- {한도 도달 시}
- {가치 느낀 순간 직후}
- {잠긴 기능/UI 자리}

> SDK — RevenueCat (iOS StoreKit / Android Play Billing 래핑). V2 통합.

---

## 3. V3 — 수익 채널 확장 (데이터 보고 결정)

### 3-A. 채널 옵션

| 채널 | 자리 | 수익성 (한국) | 심리 저항 |
|---|---|---|---|
| **제휴 (쿠팡 파트너스 등)** | 핵심 flow 외 자연 접점 (예 · 결과·요약 화면 · 콘텐츠 사이 · 앱마다 다름) | ★★★ 최상 | 낮음 |
| **Rewarded 광고** | 구독 대안 자리 (버튼 탭 → 광고 시청 → 보상 1회) | ★★ | 낮음 (사용자 자발적 선택) |
| **배너 광고 (AdMob 등)** | 화면 여백 자리 (예 · 홈/피드 하단 · 앱마다 다름) | ★ (한국 낮음 · ARPU $0.5-2/user/월) | 중간 |

### 3-B. 한국 시장 유의

- **쿠팡 파트너스가 압도적 우위.** 인앱 결제 30% 수수료 회피 + 사용자에게 실 가치 (관련 도구 추천) + conversion 높음 (한국 e-commerce dominant player)

### 3-C. 원칙 (앱 무관)

- **핵심 flow 절대 금지** — 세션/집중/입력 화면 안 광고 = 앱 핵심 가치 훼손. 어떤 앱이든 동일
- **무동의 전면 광고 (interstitial) 금지** — 앱 진입 시 · 화면 전환 시 자동 뜨는 전면 광고. 사용자 방해 · 이탈 유발. **Rewarded 는 사용자 자발적 선택이므로 OK**
- **구독자 자동 광고 제거** — Free vs Plus 차별 유지
- **카테고리 필터링** — 도박 · 저질 게임 등 브랜드 훼손 카테고리 차단

### 3-D. 보류 (앱마다 판단)

- **B2B / 팀 플랜** — 개인 앱은 검토 보류
- **데이터 판매** — 검토 X (프라이버시)

---

## 4. V1 준비 (지금 심어둘 것)

> 지금 구현하는 건 아니다. 나중에 켤 때 1-2일이면 되도록 **"자리"만** 만든다.

### DB 자리
`profiles` 에 다음 필드를 둔다 (값은 V2 까지 기본값).

- `subscription_status` — 기본 `'free'`. (`'free' | 'plus_monthly' | 'plus_annual'`)
- `{도메인 한도 컬럼}` — 기본 0. Free 한도 체크용 (V2 활용)

### 코드 분기점 (`Entitlements` 게이트)

```dart
// core/entitlements.dart
// 구독/한도 관련 게이트. V1 은 항상 허용, V2 에서 실제 로직 활성화.
class Entitlements {
  final Profile profile;
  const Entitlements(this.profile);

  bool get isPaid => profile.subscriptionStatus != 'free';

  // V1: 항상 true. V2 에서 아래 한 줄만 활성화.
  bool get canCreate{Action} {
    if (isPaid) return true;
    return true; // V2: return profile.{quota_col} < {limit};
  }

  // V1: 항상 false (광고 없음). V3 에서 활성화.
  bool get shouldShowAds => false; // V3: return !isPaid;
}
```

### "미리 만들지 않음" 원칙 (CLAUDE.md 절대 금지 #5)

- **페이월 화면 · 광고 컴포넌트 미리 만들지 않음.** DB 필드 + `Entitlements` 게이트만.
- 컴포넌트 분리 (결과 화면 하단 · 특정 자리 등) 는 강제 아님 — 과한 추상화 회피.
- V2/V3 진입 시점에 실 UI 생성.

---

## 5. 수익성 참고 (보수적 baseline · 앱별 조정)

SaaS 구독 앱 감 잡기용 baseline. 실 전환율은 페이월 UX · 가격 탄력성 · 도메인에 따라 크게 상이.

- **구독 전환율 3~5%** 가정 (일반 SaaS 앱 표준)
- **MAU 1,000** → 구독자 30-50명 → 월 ₩{MONTHLY} × 30-50 ≈ 예상 매출
- **MAU 10,000** → 구독자 300-500명 → 월 매출 10배

**손익분기점 판단 —** 인프라 비용 (Backend · Firebase · Sentry · Amplitude 무료 티어 초과분 · 도메인 · Apple $99/년 · Google $25) 이 예상 매출 넘지 않는 MAU 구간 = 손익분기.

**초기 무료 티어로 커버 가능 규모** — Beta 수백 명 · 정식 출시 초기 수천 MAU 까지. 상세 — `01_tech_stack.md` §6.

---

## 6. 이벤트 트래킹 (V2/V3 도입 시)

수익화 이벤트는 V2/V3 도입 시점에 정의 · Amplitude taxonomy 로 흡수 (`/docs-templates/06_ops/amplitude-taxonomy/catalog.template.md` §Cleanup 원칙).

**Baseline 이벤트 —**

```
paywall_viewed / paywall_subscribe_clicked
subscription_started / subscription_cancelled
rewarded_ad_offered / rewarded_ad_completed
affiliate_link_clicked (제휴 채널 있으면)
```

**Property —** `plan` (monthly/annual) · `trigger_source` (한도 도달 / 가치 순간 / 잠긴 UI 등) · `days_since_install`.

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/06_revenue_model.md`. 월 ₩4,900 / 연 ₩39,000 · 주 3회 무료 vs 무제한 Plus · 페이월 3자리 (한도 도달 · 3번째 완주 직후 · Stats 잠긴 카드).
