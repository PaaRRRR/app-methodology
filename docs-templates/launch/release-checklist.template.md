# Release Checklist

> 출시 진입 전 단일 진실. 회차별 pending 들을 통합.
> 항목 완료 시 `[x]`. 새 항목 발견 시 같은 카테고리 안에 추가.

---

## 1. 출시 차단 (필수)

### 1-A. Manual QA (사용자 디바이스 · 두 대 필요)

**풀 플로우 통합 QA**
- [ ] 가입 → 핵심 flow 완주 (시나리오 1)
- [ ] 상대방 관점 검증 (시나리오 2)
- [ ] {도메인 특화 flow}
- [ ] Home 복귀 · 상태 정합 확인
- [ ] Stats · 히트맵 · 최근 기록

**푸시 잔여**
- [ ] iOS foreground push 정책 통일
- [ ] iOS killed cold start push 도착
- [ ] 푸시 탭 → `onMessageOpenedApp` + `getInitialMessage` → 라우팅
- [ ] cron 시나리오 (before_1h · before_10m 등)
- [ ] 알림 권한 거부 fallback

**딥링크**
- [ ] 시나리오 N/N 통과 (Universal Link · Android App Link)

**심사 대응**
- [ ] UGC 신고 · 차단 sheet
- [ ] 차단 → 매칭 제외 서버 검증
- [ ] 계정 탈퇴 cascade + 새 익명 세션
- [ ] 외부 URL / 메일 진입 (약관 · 개인정보 · 피드백)
- [ ] 버전 동적 표시 (`package_info_plus`)
- [ ] 회귀 위험 zero 확인 (long-press · 관련 위젯)

### 1-B. QA 중 발견 P0/P1 fix
- 회차별 `fix/{설명}` 브랜치 · 한 branch 에 누적 ([[feedback-single-branch-per-issue]])
- 각 fix 별 회귀 테스트

---

## 2. 출시 전 권장 (옵션 · 시간 되면)

- [ ] {비용 재평가 · connection 최적화}
- [ ] {알림 loop fix · echo 방지}
- [ ] {통계 정합 검증}
- [ ] 소요·위험도 각 기록

---

## 3. 사용자 결정 필요 (출시 직전)

- [ ] 외부 링크 URL 확정 (피드백 · 약관 · 개인정보 최종 URL)
- [ ] 계정 삭제 정책 (cascade / 회고 보존)
- [ ] FAQ 실 콘텐츠 (5~10항)
- [ ] 스토어 리뷰 응답 template
- [ ] CS SLA (문의 응답 시간 목표)

---

## 4. Legal · Store 심사 대응

- [ ] Privacy Policy 최종 (변호사 검토 완료)
- [ ] Terms of Service 최종 (변호사 검토 완료)
- [ ] Play Data Safety form 작성 (수집 항목 · 처리 방식 답변)
- [ ] Apple App Privacy Details 답변
- [ ] App Store 심사 노트 작성 (UGC 처리 · 계정 탈퇴 방법 · 테스트 계정)
- [ ] 연령 등급 확정

---

## 5. Store Listing

- [ ] 앱 이름 · 부제 최종 (`docs-templates/launch/store-listing.template.md`)
- [ ] 스크린샷 (iOS 3 사이즈 · Android · Feature Graphic)
- [ ] 긴 설명 · 짧은 설명 · 키워드
- [ ] 미리보기 영상 (선택)

---

## 6. Build · Release Note

- [ ] 버전 bump (`pubspec.yaml`) · buildNumber
- [ ] What's New 작성 (한국어 · 영어)
- [ ] iOS ipa 빌드 · TestFlight 업로드 · Store 심사 제출
- [ ] Android aab 빌드 · Play Console 업로드 · Store 심사 제출

---

## 7. Prod 배포 (SQL · 앱)

- [ ] `docs-templates/launch/prod-deploy-guide.template.md` 절차 실행
- [ ] SQL migration prod 적용 순서 확인 (앱 배포 순서 vs SQL 순서)
- [ ] Guide 프로필 · seed 데이터 (해당 시)
- [ ] Edge Function 배포 (필요 시)

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/release-checklist.md` (D-20 진입 전 audit 결과 · 시나리오 15개 · P0/P1 fix 이력 · V1.1 이연 항목 등 상세).
