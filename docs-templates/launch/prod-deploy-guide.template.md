# Prod 배포 가이드

> 정식 출시 시점 · SQL migration prod 적용 · 앱 배포 순서 · 심사 제출.
> 회차별 상세는 `docs/work/{회차}/prod-deploy-guide.md` (특정 회차 산출) 로 나눔 가능.

---

## 배포 방식

### Dev vs Prod 채널 차이

| | dev | prod |
|---|---|---|
| 적용 방식 | MCP `apply_migration` | Supabase CLI `db push` |

**파일명 규칙은 dev/prod 동일** — `{NNN}_{snake_case_설명}.sql` (숫자 3자리 zero-pad). MCP `apply_migration` 은 파일명 그대로 등록. Prod `db push` 는 파일명 숫자 prefix 를 version 으로 사용 · 미적용 번호부터 순차 적용.

### 예외 — 플레이스홀더 있는 migration

`<PROJECT_URL>` / `<SERVICE_ROLE_KEY>` 등 플레이스홀더가 있는 migration → **Dashboard 에서 먼저 수동 적용 · tracking row 수동 삽입 후 `db push` 실행.** 그대로 db push 하면 트리거 망가짐.

### 클라이언트 배포 순서 (중요)

**컬럼 DROP migration 은 client 참조 완전 제거 후 적용 필수.**

- 앱 신 버전이 사용하는 필드 = 필드 nullable 로 시작 · 앱 배포 완료 후 not null 강제
- 컬럼 DROP = 이전 client 프로덕션 배포 완료 후에만 (crash 방지)
- App Store · Play 재심사 완료 확인 후 DROP migration 적용

---

## Step 0 — 사전 확인

### 0-1. prod 현재 최신 버전 확인
```sql
SELECT version, name FROM supabase_migrations.schema_migrations ORDER BY version DESC LIMIT 5;
```
- 예상 버전이 최신인지 확인
- 낮으면 → 해당 번호 ~ 예상까지 먼저 적용 필요

### 0-2. Seed 데이터 존재 확인 (해당 시)
```sql
SELECT id FROM auth.users WHERE id = '{SEED_UUID}';
```
- 없으면 → 해당 seed migration 먼저 적용

### 0-3. Edge Function Secrets 확인
Supabase Dashboard > Edge Functions > Secrets — 필요 키·값 세팅 확인.

### 0-4. Cron / trigger 상태 확인
```sql
SELECT jobname FROM cron.job;
SELECT tgname, tgrelid::regclass FROM pg_trigger WHERE tgisinternal = false;
```
- 예상 대비 정합 확인 · 이상 시 원인 파악

### 0-5. 백업
- Supabase Dashboard · Manual snapshot
- 로컬 `pg_dump` 백업 (선택)

---

## Step 1 — SQL migration 적용

```bash
supabase link --project-ref {PROD_PROJECT_REF}
supabase db push --dry-run  # 미리보기
supabase db push            # 실 적용
```

**적용 확인.**
```sql
SELECT version, name FROM supabase_migrations.schema_migrations ORDER BY version DESC LIMIT 10;
```

---

## Step 2 — Edge Function 배포 (필요 시)

```bash
supabase functions deploy {function-name} --project-ref {PROD_PROJECT_REF}
```

Secrets 재확인 · 실 호출 테스트.

---

## Step 3 — 앱 빌드 · 심사 제출

### iOS (App Store)
```bash
flutter build ipa --flavor prod -t lib/entry/entry.prod.dart --release \
  --dart-define-from-file=.env.prod \
  --export-options-plist=ios/ExportOptions.plist
```
- Transporter 로 App Store Connect 업로드
- 심사 제출 · 심사 노트 작성

### Android (Google Play)
```bash
flutter build appbundle --flavor prod -t lib/entry/entry.prod.dart --release \
  --dart-define-from-file=.env.prod
```
- Play Console `aab` 업로드
- 심사 제출 · Data Safety form 확인

---

## Step 4 — 롤아웃

**iOS.**
- Phased Release ON (자동 7일)
- 또는 Manual Release

**Android.**
- Staged rollout (10% → 50% → 100% · 관찰 시점마다 확대)

---

## Step 5 — Post-deploy 관측

- **Sentry** — 크래시율 · 새 이슈 유형
- **Amplitude** — DAU · 핵심 funnel 지표
- **Beta observations** — `docs-templates/ops/beta-observations/protocol.template.md` 프로토콜 적용

문제 발생 시 hotfix (`fix/{이슈}` 브랜치 · buildNumber 증분 · 재심사).

---

## 예시 (StartRoom)

> 실 채운 예 — StartRoom `docs/work/home-ia-redesign/prod-deploy-guide.md`. 048~070 migration 배치 · 057 예외 처리 (플레이스홀더 · Dashboard 수동) · 068/069 DROP migration client 배포 순서 · Guide 프로필 seed 확인 등 상세.
