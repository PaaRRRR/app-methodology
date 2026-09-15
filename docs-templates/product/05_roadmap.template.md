# 05. 로드맵

> 3주 (21일) MVP 일정 + 출시 후 1주 운영.
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
- 핵심 컴포넌트 — `{PREFIX}Button`, `Card`, `Avatar`, `Input` (먼저 4개)
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

- 베타 테스터 피드백 수집 (`docs-templates/ops/beta-observations/`)
- P0/P1 fix
- 스토어 등록 (`docs-templates/launch/store-listing.template.md`)
- 심사 제출 · 롤아웃 (`docs-templates/launch/prod-deploy-guide.template.md`)

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/05_roadmap.md`. D1~D20 상세 일별 · Week 별 종료 체크리스트 · 실제 진행 이력 (D8~D14 화면 완성 · D15 Realtime · D16 매칭 · D17 FCM · D18 딥링크 · D20 베타).
