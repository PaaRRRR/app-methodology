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
| Stats | {LIMITED} | 전체 + 인사이트 |
| {FEATURE_2} | {BASIC} | {ENHANCED} |

### 페이월 트리거 (어디서 구독 권함)
- {한도 도달 시}
- {가치 느낀 순간 직후}
- {잠긴 카드}

> SDK — RevenueCat (iOS StoreKit / Android Play Billing 래핑). V2 통합.

---

## 3. V3 — 광고 (데이터 보고 결정)

### 형태
- **배너 광고** — 홈 하단 · 완료 화면 하단. **핵심 flow 안에는 절대 없음.**
- **Rewarded** — 페이월에서 "광고 보고 1회 추가"

### 원칙
- 사용자 방해 최소
- 핵심 flow 오염 금지
- 구독자에게는 광고 노출 없음

---

## 4. V1 준비 (지금 심어둘 것)

- DB — `profiles.subscription_status` 컬럼 (nullable, MVP 는 항상 null)
- 코드 — `isPlus(user)` helper (MVP 는 항상 false)
- Feature flag — Plus 기능 코드 분기점 표시 (`// TODO(V2): Plus only`)

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/06_revenue_model.md`. 월 ₩4,900 / 연 ₩39,000 · 주 3회 무료 vs 무제한 Plus · 페이월 3자리 (한도 도달 · 3번째 완주 직후 · Stats 잠긴 카드).
