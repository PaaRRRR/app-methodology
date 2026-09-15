# supabase-dev-vs-prod — Supabase 활용 원칙 + tips

> dev · prod 환경 분리 원칙 + StartRoom 경험에서 정리한 실 tips.

---

## 1. 원칙

| 환경 | 스키마 변경 채널 | 이유 |
|---|---|---|
| **dev** | Supabase MCP `apply_migration` | Claude Code 안에서 직접 실행 · 실험 자유 · 데이터 리셋 가능 |
| **prod** | Supabase CLI `db push` | 신중 · 회귀 위험 · 백업·롤백 확인 · 사용자 데이터 |

**절대** — prod 에 MCP `apply_migration` 사용 금지.

---

## 2. dev migration flow (MCP)

1. `docs/02_architecture.md` 스키마 정합 확인 · 필요하면 사전 update
2. MCP `apply_migration` 호출 — migration name + SQL
3. MCP `list_migrations` 로 등록 확인 (**tip 3 · 누락 이슈 주의**)
4. `docs/02_architecture.md` sync (필드 · enum · 함수 · 트리거)
5. 코드 반영 (freezed model · Repository)

---

## 3. prod migration flow (CLI)

1. dev 에서 충분히 검증
2. `supabase db diff` 로 SQL 최종 확인
3. 데이터 백업 (Supabase Dashboard · manual snapshot)
4. `supabase db push` — prod 프로젝트 대상
5. 앱 신 빌드에 반영된 필드 사용 시 앱 배포 순서 조정 (SQL 먼저 · 앱 나중, 또는 필드 nullable 로 시작)
6. 심사 통과 후 롤아웃 시점에 사용자에게 필요 시 안내

상세 절차 — `docs-templates/launch/prod-deploy-guide.template.md`

---

## 4. Tips (StartRoom 경험 · 배포 phase 진입 시 재검토·확장)

### 4-A. Migration 관리
- **표기명 규칙** — `{NNN}_{snake_case_설명}.sql` (예: `041_leave_room_soft_delete.sql`). 번호 3자리 zero-pad · 순차 증가 · 겹침 방지 원칙
- **MCP `list_migrations` 누락 이슈** — MCP 로 apply 했는데 list 에 안 나타나는 케이스 있음. list 누락 ≠ DB effect 없음. 실 DB `pg_dump` 또는 Dashboard SQL Editor 로 스키마 재확인
- **하나의 migration 은 하나의 논리 변경** — 여러 테이블 · 여러 목적 섞지 않음
- **회귀 정정 migration** — 이전 migration 결과 뒤집을 때는 이전 파일 편집하지 말고 새 파일 추가 (이력 보존)

### 4-B. 보안 (Advisor)
- **SECURITY DEFINER 함수 search_path 고정** — Advisor WARN 뜨면 필수. `SET search_path = public, pg_temp` 명시
- **anon EXECUTE 회수** — is_room_member 같은 helper 함수는 authenticated · service_role 만 EXECUTE. anon 은 REVOKE
- **RLS 헬퍼** — 협업 · 방 기반 앱은 `is_{scope}_member(scope_id)` SECURITY DEFINER 헬퍼 만들어 RLS policy 에서 재사용

### 4-C. 데이터 보존
- **soft delete 우선** — CASCADE 대신 `left_at IS NULL` · `deleted_at IS NULL` 필터링. 감사·복구 가능
- **GDPR delete-account** — JWT 검증 → service_role CASCADE 삭제 edge function. 사용자 요청 시에만 hard delete
- **auth.users CASCADE** — 사용자 삭제 시 관련 데이터 자동 삭제되도록 FK 정책 설계 (CASCADE / SET NULL 명확 구분)

### 4-D. Realtime
- **publication 등록** — Realtime 사용 테이블만 `alter publication supabase_realtime add table {table}` 명시
- **silent trigger 패턴** — 알림 vs Realtime 신호 분리. `_silent` 플래그 · silent trigger 로 UI cubit 만 신호받고 알림 안 보냄

### 4-E. Edge functions
- **CORS** — 앱에서 호출 시 `Access-Control-Allow-Origin` 헤더 필수
- **service_role key** — env 로만 주입 · 코드 하드코딩 금지
- **JWT 검증** — 사용자 요청은 authenticated JWT verify 후 진행
- **pg_net trigger → edge function** — DB trigger 에서 edge function 호출 시 pg_net 확장 활용

### 4-F. FCM · notification_settings
- **notification_settings JSONB** — 사용자 알림 토글 관리. 키 · 기본값은 앱별 정의
- **UNREGISTERED 정리** — FCM 응답 UNREGISTERED 오면 token 삭제

### 4-G. 회차 진입 시 브레인스토밍
이 tips 는 배포 phase 진입 시점에 사용자와 재확인. StartRoom 이후 새 앱 경험 쌓이면 tips 추가·수정.

---

## 5. StartRoom migration 참고 자원

> reference 구현체. 다음 앱이 유사 도메인 (협업 · 방 기반) 이면 직접 참조.

- `supabase/migrations/001_extensions.sql` — pgcrypto · uuid-ossp
- `supabase/migrations/004_rls_policies.sql` + `is_room_member()` — 방 기반 RLS 패턴
- `supabase/migrations/006, 007, 042` — SECURITY DEFINER 보안 강화
- `supabase/migrations/041_leave_room_soft_delete.sql` — soft delete 패턴
- `supabase/functions/delete-account/index.ts` — GDPR RPC

---

## 6. 관련 결정

- [[reference-supabase-migration-channel]] — MCP vs Dashboard 채널 선택 시 사용자 확인
