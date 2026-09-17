# 05. 로드맵

> MVP 목표 기간 (예: 3주 · 21일) + 출시 후 1주 운영.
> **일별 일정은 가이드, 주별 마일스톤은 약속.** 일정보다 품질 우선.

개발 철학 — **기반 → 뼈대 → 화면 → 연결 → 기능 → 고도화**.

```
Week 1  기반 구축   "측량 · 기초 · 철근"     → 인프라 + 골격
Week 2  화면 완성   "외벽 · 인테리어"        → mock 데이터로 모든 UI
Week 3  기능 탑재   "전기 · 수도 연결"       → 실제 동작 + 베타 출시
```

---

## Week 1 — 기반 구축

> **마일스톤 (약속).** 주 끝에 "빈 화면들이 라우팅되고, 디자인 시스템이 살아있고, DB 가 동작하고, Realtime 이 양방향으로 검증된" 상태.

### 일별 가이드

**D1 — 환경 셋업**
- Flutter 프로젝트 생성 (org: {ORG})
- Git init + Conventional Commits + 첫 커밋
- flavor 분리 (dev/prod) + 진입점
- Backend 프로젝트 2개 (dev/prod)
- Firebase 프로젝트 2개
- Backend MCP 연결 (dev)
- Sentry / Analytics 통합 · 도착 확인
- 산출물 — dev/prod 둘 다 빌드 성공

**D2 — 디자인 시스템 (가장 중요한 날)**
- theme.dart — 컬러·타이포 토큰
- 핵심 컴포넌트 — `{PREFIX}Button` · `{PREFIX}Card` · `{PREFIX}Input` · `{PREFIX}IconButton` (먼저 4개 · `04_design_system.md` §5 필수 세트 참조)
- 폰트 적용
- 컴포넌트 카탈로그 화면 (/dev/components)
- 산출물 — 카탈로그에서 모든 위젯 확인 가능

**D3 — 폴더 구조 + 라우팅 골격**
- features/ 폴더 + 빈 Screen 생성
- go_router 셋업 + 모든 라우트 정의
- 산출물 — 빈 화면들끼리 자유롭게 이동 가능

**D4 — DB 스키마 + Repository**
- migrations 작성 + MCP 로 dev 실행
- 익명 인증 흐름 (필요 시)
- freezed 모델
- Repository 인터페이스 + 구현
- 산출물 — dev 화면 CRUD 확인

**D5 — Realtime 위험 검증 (조기 검증)**
- "Hello Realtime" 페이지 (dev 전용)
- 두 디바이스 broadcast 양방향 확인
- 실패 시 Replication 설정 디버깅
- 산출물 — Realtime 양방향 동작
- (이 검증을 D15 로 미루면 위험)

**D6~D7 — Buffer + 기반 마무리**
- 밀린 작업
- analytics wrapper
- AppError 타입
- BlocObserver
- 카탈로그 보강

### Week 1 종료 체크리스트
- [ ] dev/prod 빌드 둘 다 성공
- [ ] 컴포넌트 카탈로그 동작
- [ ] 모든 화면 라우팅 (빈 페이지)
- [ ] DB CRUD 동작
- [ ] Realtime 양방향 동작
- [ ] Sentry / Analytics 도착 확인

---

## Week 2 — 화면 완성

> **마일스톤.** "모든 화면이 디자인대로 완성되고 mock 데이터로 풀 플로우 시연 가능" 상태. 아직 실 백엔드 연결 최소.

### 일별 가이드
- D8~D14 — 화면별 UI 구현 (프로토타입 실측값 그대로 포팅)
- Bloc / Cubit 골격 + mock state
- 화면 간 flow 연결

### Week 2 종료 체크리스트
- [ ] 모든 화면 프로토타입 대비 시각 정합
- [ ] mock 데이터로 풀 플로우 (진입 → 액션 → 결과) 시연 가능

---

## Week 3 — 기능 탑재 + 베타

> **마일스톤.** "실제 백엔드·Realtime·푸시·딥링크 연결 · 베타 배포 완료".

### 일별 가이드
- D15 — Realtime 실 연결
- D16 — 매칭 (해당 시) / 도메인 핵심 기능
- D17 — FCM 푸시 (앱 안 알림 + OS 푸시)
- D18 — 딥링크 (Universal Link · Android App Link)
- D19 — 통합 테스트
- D20 — 베타 배포 (TestFlight · Firebase App Distribution)

### Week 3 종료 체크리스트
- [ ] 풀 플로우 real backend 동작
- [ ] 푸시 도착 · 딥링크 진입 확인
- [ ] 베타 빌드 배포 (TestFlight/Firebase)

---

## Week 4 — 베타 운영 + 정식 출시 준비

- 베타 테스터 피드백 수집 (`/docs-templates/ops/beta-observations/`)
- P0/P1 fix
- 스토어 등록 (`/docs-templates/launch/store-listing.template.md`)
- 심사 제출 · 롤아웃 (`/docs-templates/launch/prod-deploy-guide.template.md`)

---

## 리스크 & 대응

| 구간 | 리스크 | 대응 |
|---|---|---|
| D2 디자인 시스템 | 1일은 빡빡 (컴포넌트 + 폰트 + 카탈로그) | 핵심 4개 먼저 · 나머지는 화면 만들며 추가 |
| D5 Realtime (사용 시) | Replication 설정 누락 흔함 | D6-D7 Buffer 로 흡수 |
| Week 2 복잡 화면 | 복잡한 상태 머신 화면 (예: 세션 · 실시간 채팅 · 폼 단계 등) 은 하루 부족 | 골격/디테일 2일 분할 |
| Week 3 Realtime 통합 (사용 시) | 화면 만들고 한참 후 통합 = 회귀 위험 | D5 검증 패턴 그대로 이어붙이기 |
| Week 3 FCM | iOS 인증서 · APNs Key Environment 이슈로 1.5일 가능 | 충분한 시간 확보 · 급하면 일정 연장 |
| 심사 리젝 | 앱 성격에 따라 UGC · 연령 등급 · Data Safety · 권한 · 개인정보 등 오답 | Reviewer notes 강화 (`/docs-templates/launch/store-listing.template.md`) |

---

## 범위 밖 (V2+) — 절대 MVP 에 넣지 않음

- {V2 이연 기능 목록 · `06_revenue_model.md` §1 표와 동기화}
- 처음부터 V2 분류한 것은 축소가 아니라 원래 범위. 그대로 V2 에 둔다.

**대원칙 — 일정보다 품질 우선. 기능을 빼서 일정을 맞추지 않는다.**

- MVP 범위 기능은 모두 "핵심 가설 검증에 필요" 라 판단된 것. 시간이 없다고 빼면 검증 자체가 흔들림
- 일정이 밀리면 **기능을 줄이는 게 아니라 일정을 연장**한다. **MVP 목표 기간은 목표지 마감이 아니다**
- 주별 마일스톤의 "상태" 는 유지 · 도달 날짜는 유연
- 일별 일정 = 가이드 · 주별 마일스톤 = 약속

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/05_roadmap.md`. D1~D20 상세 일별 · Week 별 종료 체크리스트 · 실제 진행 이력 (D8~D14 화면 완성 · D15 Realtime · D16 매칭 · D17 FCM · D18 딥링크 · D20 베타).
