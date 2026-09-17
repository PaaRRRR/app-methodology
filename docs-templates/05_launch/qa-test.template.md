# QA Test — 배포 전 시각·기능 체크

> Phase 2 프로토타입 (jsx) vs Flutter 실측 시각 비교 + 기능 flow 체크리스트.
> CLAUDE.md 룰 #2 (하드코딩 금지) · 룰 #8 (프로토타입 = 시각 진실) 실행.

---

## 1. 원칙 — 코드 먼저, 시각 나중

시각 차이 발견 시 순서 —

1. **코드 1차** — 의심 토큰/위젯 코드 텍스트 비교 (jsx vs Dart · 80% 차이는 값 잘못 옮긴 것)
2. **시각 2차** — 안 풀리면 스크린샷 비교
3. **수치 3차** — 그래도 모르면 Flutter Inspector · 브라우저 DevTools computed value

스크린샷 비교 전에 코드 비교. 텍스트 비교가 시각 비교보다 정확·빠름.

---

## 2. 시각 차이 6가지 차원

차이 발견 시 어느 차원인지 분류. 차원 정해지면 의심 위치 좁혀짐.

| 차원 | 예 | 의심 위치 |
|---|---|---|
| 색 | 배경 · 텍스트 · border · 투명도 | `AppColors.{토큰}` |
| 크기 | height · width · font-size | `AppSpacing` / `AppTextStyles` |
| 간격 | padding · margin · gap | 위젯 `padding` · `SizedBox` |
| 모양 | border-radius · border-width | `AppRadius.{토큰}` · `BorderSide.width` |
| 타이포 | weight · letter-spacing · line-height | `AppTextStyles.{스타일}` |
| 애니메이션 | 동작 · 타이밍 · easing | `AnimatedX` · motion 토큰 |

---

## 3. 배포 전 QA 체크리스트

**앱 무관 항목 = 기본 · 앱 특화 = 조건부.** 앱 성격 안 맞는 항목 skip.

### 3-A. 시각 QA (Phase 2 프로토타입 있으면)

- [ ] **각 화면 실측 확인** — §2 6차원 대비 프로토타입 정합
- [ ] **애니메이션 · 상태 전환** — 프로토타입 정합
- [ ] **다크·라이트 모드** — 각각 확인 (해당 시)
- [ ] **Dynamic Type · 접근성 크기** — 큰 글자·작은 글자 대응
- [ ] **VoiceOver / TalkBack** — 주요 화면 스크린리더 통과 (접근성 앱)

### 3-B. 기능 QA (앱 무관 · 공통 · 필수)

- [ ] **온보딩 flow** — 첫 진입 → 권한 요청 → 완료 · 각 화면 이탈 재진입
- [ ] **알림 3상태** — foreground · background · killed · 권한 거부 시 처리
- [ ] **딥링크** — Universal Link · 앱 미설치 시 fallback · 잘못된 URL 처리
- [ ] **오프라인 · 재접속** — 네트워크 끊김 시 UI · 재접속 시 복구
- [ ] **신고 · 차단 · 계정 탈퇴** — 각 flow 완주
- [ ] **버전 · 외부 URL** — 설정 화면 버전 표시 · 개인정보처리방침·이용약관 링크
- [ ] **에러 · 크래시** — Sentry 실측 · 오류 로그 확인
- [ ] **Amplitude event** — 핵심 event 발화 확인 (dev 환경)

### 3-C. 기능 QA (앱 특화 · 해당 시만)

- [ ] **멀티 사용자 · Realtime 앱** — 두 대 디바이스 필수 · 상대방 관점 검증 · 동시 진행 상태 sync · 상대방 이탈 감지
- [ ] **결제 앱** — 구독 결제 · 환불 · 프로모 코드 · 가족 공유 · 결제 실패 처리
- [ ] **음성/영상** — 마이크·카메라 권한 · 백그라운드 이어짐 · 스피커·이어폰 전환 · 인터럽트 (전화 등)
- [ ] **위치 기반** — GPS 권한 · 백그라운드 위치 · 정확도 저하 대응
- [ ] **{앱 특화 flow}** — {…}

### 3-D. 발견 처리

- **P0/P1 이슈** — `fix/{설명}` 브랜치 · 회차별 branch 누적 원칙 · 재QA 필수
- **P2/P3** — 배포 후 hotfix 여부 판단 (심하지 않으면 다음 회차로)

---

## 예시 (StartRoom)

> StartRoom `_qa/visual_qa.md` (시각 소통 매뉴얼) + `_qa/README.md` (폴더 관행) + `docs/work/qa-prerelease-batch/manual-qa.md` (실 QA 시나리오 15개).
>
> StartRoom 은 매칭·Realtime·음성 앱이라 §3-C 앱 특화 시나리오 다수 (두 대 디바이스 · 상대방 관점 · 마이크 권한 · 백그라운드 유지). 신규 앱은 자기 앱 특화만 필터해서 채택.
