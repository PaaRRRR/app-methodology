# QA Test — 프로토타입 vs Flutter 실측 비교

> 배포 전 시각 QA · 시각 차이 발견 시 어떻게 소통하는지 매뉴얼.
> CLAUDE.md 룰 #2 (하드코딩 금지) · 룰 #8 (프로토타입 = 시각 진실) 실행.

---

## 1. 핵심 원칙

**스크린샷보다 코드 재확인이 먼저.** 80% 차이는 jsx 값을 잘못 옮긴 것. 텍스트 비교가 시각 비교보다 항상 정확·빠르다.

**순서.**
1. **코드 1차 확인** — 의심 토큰/위젯 코드 텍스트 비교
2. **시각 2차 확인** — 1차로 안 풀리면 스크린샷
3. **수치 3차 확인** — 그래도 모르면 Flutter Inspector · 브라우저 DevTools computed value 비교

---

## 2. 차이의 6가지 차원

차이가 보이면 어느 차원인지 분류. 차원 정해지면 의심 위치 좁혀짐.

| 차원 | 무엇을 보나 | 의심 위치 |
|---|---|---|
| **색** | 배경 · 텍스트 · border · 투명도 | `AppColors.{토큰}` |
| **크기** | height · width · font-size | `AppSpacing` / `AppTextStyles` |
| **간격** | padding · margin · gap | 위젯 `padding`, `gap`, `SizedBox` |
| **모양** | border-radius · border-width | `AppRadius.{토큰}`, `BorderSide.width` |
| **타이포** | weight · letter-spacing · line-height | `AppTextStyles.{스타일}` |
| **애니메이션** | 동작 · 타이밍 · easing | `AnimatedX`, `TweenAnimation`, Motion 토큰 |

---

## 3. 좋은 설명 vs 나쁜 설명

❌ 나쁜 예 (뭘 고쳐야 할지 모름)
- "버튼이 좀 달라"
- "느낌이 다른데"

✅ 좋은 예 (한 곳 짚음)
- "SrButton primary 배경 채도가 낮음. `AppColors.accent` 값 의심."
- "SrCard padding 이 좁음. 프로토타입 18, 실측 16 처럼 보임."

✅ 더 좋은 예 (차원 + 위치 + 근거)
- "SrButton lg 의 height 가 작음. `04_design_system.md` §5-1 에 56 명시, 시뮬레이터 50 처럼 보임. SrButton 위젯 height 상수 확인."

---

## 4. 프롬프트 공식

```
[화면/위젯] 의 [차원] 이 다름.
프로토타입은 [X], 내 구현은 [Y] 같음.
[의심 위치] 확인해줘.
```

---

## 5. QA 캡처 보관

**`_qa/` 폴더 관행.**
- 화면 캡처 · 색 측정 메모 · 영상 자유 보관
- `.gitignore` 로 추적 X · `README.md` 만 예외
- 정식 워크플로우 필요해지면 `_qa/visual_qa.md` 로 정리

---

## 6. QA 실행 시나리오 (배포 전 · 필수)

**두 대 디바이스 필요** (매칭·Realtime 앱은 필수).

- 시나리오 1 — 가입 → 핵심 flow 풀 완주
- 시나리오 2 — 상대방 관점 검증
- 시나리오 3 — 알림 (foreground · killed · 권한 거부)
- 시나리오 4 — 딥링크 진입
- 시나리오 5 — 신고 · 차단 · 계정 탈퇴 · 외부 URL · 버전 표시
- ... (앱별 flow 추가)

**발견 P0/P1** — `fix/{설명}` 브랜치 · 회차별 branch 누적 원칙.

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `_qa/visual_qa.md` (시각 소통 매뉴얼) + `_qa/README.md` (폴더 관행) + `docs/work/qa-prerelease-batch/manual-qa.md` (실 QA 시나리오 15개).
