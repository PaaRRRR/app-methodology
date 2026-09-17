<!--
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ 참조 전용 예시 — 그대로 복사 금지.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

이 파일은 StartRoom `home-ia-redesign` 회차 (2026-09-05) 의 실 배포 가이드다.
그 회차 특수 상황 (048~070 migration 순서 · 057 예외 플레이스홀더 · 068/069 DROP client 배포 순서 · Guide 프로필 seed 확인 등) 이 정교하게 반영됨.

**신규 앱 사용 방식 —**
1. 자기 회차 배포 시 `docs/work/{deployment-topic}/prod-deploy-guide.md` 로 이 정도 정교함으로 자기 상황 반영해서 작성.
2. Template scaffold 는 `/docs-templates/05_launch/prod-deploy-guide.template.md` — 앱 무관 원칙·순서만.
3. 이 파일은 "이 정도 정교함으로 회차 문서 만드는구나" 감 참고용.

**왜 이 파일이 여기 있는가 —**
- Template scaffold 만으로는 실 배포 정교함 감 부족.
- 실 회차 산출물 통째로 읽어야 "특수 상황 · 순서 · 검증 방식" 감 온다.
- Fork 시 참고 가능하도록 저장소에 포함.

**주의 —**
- Migration 번호 (048~070) · seed UUID · 회차 이름 (home-ia-redesign) 은 StartRoom 특화.
- 자기 앱은 자기 상황 (자기 migration 번호 · 자기 회차) 로 재작성.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-->

# home-ia-redesign → prod 배포 가이드

**대상 브랜치**: `main` (fix/pre-release-qa 머지 완료 후)
**신규 마이그레이션**: 048 ~ 070 (23개, 단 057은 별도 처리)
**수정된 EF**: `send-push`, `match-tick` (v19, 코멘트 60초 갱신)

> **회차 확장 안내** — 원 048~069 배치에 fix/pre-release-qa 회차 발견 사항 추가:
> - **070_participant_join_notify** (신규) — INSERT 트리거 · invite 참가 시 방장에게 visible push. QR/코드 참가 후 방장 홈 즉시 동기화 (Realtime) + 배너 알림.
> - **064 최종 카피 병합** — QA 발견 카피 폴리시 (`072_room_canceled_copy_polish` consolidated). Title fallback 분기 · body "괜찮아요, 다시 준비해봐요".
> - **070 client backward compat OK** — 이전 client (v1.0.1+10) 도 새 트리거 안전 (notifications insert 만, 스키마 변경 없음).

---

## 배포 방식

### Dev vs Prod 추적 방식 차이

| | dev | prod |
|---|---|---|
| 적용 방식 | MCP `apply_migration` | `supabase db push` |
| schema_migrations 버전 | `20260823080217` (타임스탬프) | `048`, `049`, ... (파일명) |

Prod는 처음부터 `supabase db push` 로 셋업됐다. 파일명의 숫자 prefix가 version이 되므로,
`048_rooms_duration_secs.sql` → version `048` → prod에 없음 → 정확히 048부터 적용된다.

**`supabase db push`를 메인 배포 수단으로 사용한다.**

### 단, 057은 예외

`057_silent_notification_guard.sql`에는 `<PROJECT_URL>` / `<SERVICE_ROLE_KEY>` 플레이스홀더가 있다.
`db push`가 그대로 실행하면 push 알림 트리거가 망가진다.
→ **057은 Dashboard에서 먼저 적용하고, tracking row를 수동 삽입한 뒤 `db push`를 실행한다.**

### 클라이언트 배포 순서 (중요)

**068 · 069 는 client 참조 완전 제거 후 적용 필수:**

- **068 `drop_favorites_columns`** — `preferred_partner_ids` · `prefer_favorites` 컬럼 DROP. **client v1.0.0+9 이상 (§18 마감) 프로덕션 배포 완료 후에만 적용.** 이전 client 는 이 컬럼 fetch 시도 → crash. → App Store · Google Play 재심사 완료 확인 후 적용.
- **069 `drop_seen_tooltips`** — `seen_tooltips` 컬럼 DROP. Client 참조 0 (오래 전부터 미사용, §19-1 verify grep 확인). **client 버전 무관, 안전.**

---

## Step 0 — 사전 확인

### 0-1. prod 현재 최신 버전 확인

```sql
SELECT version, name FROM supabase_migrations.schema_migrations ORDER BY version DESC LIMIT 5;
```

- `047`이 최신이면 정상 (048~ 미적용 상태)
- `047`보다 낮으면 → 해당 번호~047까지 먼저 적용 필요 (이번 배포 범위 외)

### 0-2. Guide 프로필 존재 확인

```sql
SELECT id FROM auth.users WHERE id = '00000000-0000-0000-0000-000000000001';
```

- 결과 있음 → OK
- **결과 없음 → 배포 전 `043_seed_guide_profile.sql` 먼저 적용 필수**

> 과거 prod 이슈: guide 프로필 없음 → 매칭 폴백 실패. 반드시 확인.

### 0-3. GUIDE_USER_ID EF 시크릿 확인

Supabase Dashboard > Edge Functions > Secrets에서
`GUIDE_USER_ID = 00000000-0000-0000-0000-000000000001` 인지 확인. 다르면 수정.

### 0-4. retro_reminder cron 존재 여부 확인

```sql
SELECT jobname FROM cron.job WHERE jobname LIKE '%retro%';
```

- 결과 있음 → 삭제 (056_notif_cleanup이 함수는 DROP하지만 cron row는 수동 삭제 필요)

```sql
SELECT cron.unschedule('여기에-jobname-입력');
```

### 0-5. 클라이언트 프로덕션 버전 확인 (068 배포 전 필수)

App Store Connect + Google Play Console 에서 현재 프로덕션 버전 확인:

- **v1.0.0+9 이상 (§18 마감) 배포 완료 + 재심사 통과 확인** → 068 적용 가능
- **v1.0.0+9 미만** → 068 적용 skip (Step 2 에서 `--include-file-until 067` 옵션 or 파일 임시 제외). 재심사 통과 후 별도 배포 사이클에 068 · 069 순차 적용.

### 0-6. 기존 cron job 목록 확인 (신규 등록 대상 파악)

```sql
SELECT jobname, schedule FROM cron.job ORDER BY jobname;
```

아래 항목 중 없는 것만 Step 4에서 등록:

| jobname | schedule | 담당 migration |
|---|---|---|
| `notif-tick-60s` | `* * * * *` | 025 (이미 존재해야 함) |
| `match-tick` | `* * * * *` | (Step 4.5 rename 대상) |
| `notif-streak-daily` | `0 11 * * *` | 053 (신규) |
| `notif-done-recap-daily` | `0 23 * * *` | 054 (신규) |
| `cancel-stale-rooms` | `*/15 * * * *` | 055 (신규) |
| `notif-weekly-summary-sun-21kst` | `0 12 * * 0` | 033 (이미 존재해야 함) |

**`match-tick-30s` 또는 `match-tick-60s` 로 존재하는 경우** → Step 4.5 에서 `match-tick` (suffix 없이) 로 rename. jobname 에 스케줄 박으면 스케줄 변경 시마다 rename 필요 · 향후 정합성 위해 suffix 제거.

---

## Step 1 — 057 Dashboard 적용 (db push 전 필수)

`supabase/migrations/057_silent_notification_guard.sql` 파일을 열어 두 곳을 치환.

```
<PROJECT_URL>      →  https://[prod-project-id].supabase.co
<SERVICE_ROLE_KEY> →  Dashboard > Project Settings > API > service_role (secret key)
```

치환 후 Dashboard > SQL Editor에서 실행.

완료되면 tracking row 수동 삽입:

```sql
INSERT INTO supabase_migrations.schema_migrations (version, name, statements)
VALUES ('057', 'silent_notification_guard', NULL);
```

> 이 row가 있어야 `db push`가 057을 건너뛴다.

---

## Step 2 — supabase db push

**068 배포 조건 재확인** — Step 0-5 결과가 "v1.0.0+9 이상 배포 완료" 인 경우에만 048~069 전 범위. 그렇지 않으면 048~067 만.

**사전 — CLI 로그인 + link** (한 번만):
```bash
supabase login
supabase link --project-ref [prod-project-ref]
```

link 완료 후 `--project-ref` 생략 가능.

**`--include-all` 필수** — 057 tracking row 삽입 시점부터 remote 최신 버전이 057 로 인식됨. CLI 는 "remote 최신보다 낮은 번호 (048~056) 는 안전상 스킵" 로직 · `--include-all` 로 우회.

```bash
supabase db push --include-all
```

이 명령이 자동으로 처리하는 것:
- `048` ~ `070` 순서대로 적용 (057 skip)
- 이미 tracking row 있는 057 은 자동 건너뜀

dry-run으로 먼저 확인 권장:

```bash
supabase db push --dry-run --include-all
```

예상 dry-run 출력:
```
Would push these migrations:
  048_rooms_duration_secs.sql
  049_sticker_emoji_backfill.sql
  050_profiles_total_focus_sec.sql
  051_fn_notify_partner_retro_title_3branch.sql
  052_gen_nickname_v2.sql
  053_streak_cron.sql
  054_done_recap_cron.sql
  055_cancel_stale_rooms_cron.sql
  056_notif_cleanup.sql
  058_silent_triggers.sql              ← 057 skip (tracking row)
  059_cancel_empty_room_trigger.sql
  060_rooms_match_after_at.sql
  061_fix_on_participant_finished.sql
  062_fn_cancel_room.sql
  063_rooms_match_pool_user_ids.sql
  064_room_canceled_payload_expand.sql
  065_profiles_bio.sql
  066_profiles_avatar_color.sql
  067_max_streak_and_focus_sec_fix.sql
  068_drop_favorites_columns.sql
  069_drop_seen_tooltips.sql
  070_participant_join_notify.sql
```
총 22건 (23 − 057).

실 push (dry-run 뺀):
```bash
supabase db push --include-all
```
`Applying migration ...` 순차 로그 + `Finished supabase db push.` 로 마감.

> `db push` 실행 전 반드시 `main` (fix/pre-release-qa 머지 완료) 기준으로 실행한다.

**070 특이사항 없음** — 새 함수 `fn_participant_join_notify` + 새 트리거 `tg_participant_join_notify` (INSERT on room_participants). 기존 데이터 마이그레이션 없음 · placeholder 없음 · client 참조 없음. `db push` 로 자동 적용.

**064 카피 병합** — 이전 072 파일이 카피 폴리시로 존재했으나 fix/pre-release-qa 회차에서 064 파일에 병합 (prod 미적용 상태라 consolidate 가능). Prod 는 064 apply 시 최종 카피 (title 분기 + body "괜찮아요, 다시 준비해봐요") 로 바로 적용됨.

---

## Step 3 — 049 적용 후 sticker 데이터 확인 (권장)

049는 `retrospectives.sticker` 컬럼을 enum → text로 전환하면서 기존 데이터를 UPDATE한다.

```sql
SELECT sticker, count(*) FROM retrospectives GROUP BY sticker LIMIT 20;
```

값이 emoji(🔥, ✨ 등)로 되어 있으면 정상. enum 코드명(`punch`, `fire`)이 남아 있으면 이상.

---

## Step 4 — cron 등록 (Step 0-6에서 없는 것만)

```sql
-- streak 알림 (매일 20:00 KST = 11:00 UTC)
SELECT cron.schedule(
  'notif-streak-daily',
  '0 11 * * *',
  $$ SELECT notify_streak_daily(); $$
);

-- done_recap 알림 (매일 08:00 KST = 전날 23:00 UTC)
SELECT cron.schedule(
  'notif-done-recap-daily',
  '0 23 * * *',
  $$ SELECT notify_done_recap_daily(); $$
);

-- 만료 방 자동 취소 (매 15분)
SELECT cron.schedule(
  'cancel-stale-rooms',
  '*/15 * * * *',
  $$ SELECT cancel_stale_rooms(); $$
);
```

---

## Step 4.5 — match-tick cron jobname rename (있는 경우만)

Step 0-6 에서 `match-tick-30s` 또는 `match-tick-60s` 로 존재하면 `match-tick` (suffix 없이) 로 rename. jobname 에 스케줄 (`-30s`/`-60s`) 을 박으면 스케줄 변경 시마다 rename 요구 · suffix 제거로 identity/schedule 분리.

**스케줄 값**:
- Dev 기준 `* * * * *` (매분).
- Prod 가 `30 seconds` 로 돌고 있으면 부하 절반 감소 + dev 통일 목적으로 매분 정합. 매칭 latency 최악 +30s 는 MVP 특성상 미미.

```sql
-- 기존 jobname 해제 (`-30s` 또는 `-60s` 중 존재하는 것)
SELECT cron.unschedule('match-tick-30s');   -- 존재하지 않으면 오류 · 실 이름으로

-- 신규 등록 (suffix 없이)
SELECT cron.schedule(
  'match-tick',
  '* * * * *',
  $$ SELECT net.http_post(
       url := '<PROJECT_URL>/functions/v1/match-tick',
       headers := jsonb_build_object(
         'Authorization', 'Bearer <SERVICE_ROLE_KEY>',
         'Content-Type', 'application/json'
       )
     ) $$
);

-- 확인
SELECT jobname, schedule FROM cron.job WHERE jobname LIKE '%match%';
```

`<PROJECT_URL>` · `<SERVICE_ROLE_KEY>` 치환 필수 (service_role key 노출 자리이므로 Dashboard SQL Editor 사용자 직접 실행).

> **Naming 원칙** — cron jobname 은 identity 만 (스케줄 표기 X). `cron.job.schedule` 컬럼이 스케줄 단일 진실. 향후 신규 cron 도 suffix 없이 (`notif-tick`, `cancel-stale-rooms` 등) — 기존 `notif-tick-60s` · `notif-weekly-summary-sun-21kst` 는 필요 시 후속 회차에 정합.

---

## Step 5 — Edge Function 배포

**send-push** — 기능 변경 없지만 done_recap 토글 코멘트 정정으로 배포 권장.

```bash
supabase functions deploy send-push --project-ref [prod-project-ref]
```

**match-tick (v19)** — 즐겨찾기 삭제 (§18) 반영 + 60초 코멘트 갱신 (§19-3). 반드시 배포.

```bash
supabase functions deploy match-tick --project-ref [prod-project-ref]
```

다른 EF (`assign-guide`, `convert-to-guide`, `delete-account`)는 변경 없음.

---

## Step 6 — 사후 검증

```sql
-- 마이그레이션 23개 확인 (057 포함)
SELECT version, name
FROM supabase_migrations.schema_migrations
WHERE version::int >= 48
ORDER BY version::int;
-- 048~070 (068 skip 시 048~067 + 069 skip · 070 은 client 무관)

-- 핵심 함수 존재 확인
SELECT proname FROM pg_proc
WHERE proname IN (
  'notify_streak_daily',
  'notify_done_recap_daily',
  'cancel_stale_rooms',
  'on_participant_finished',
  'fn_notifications_after_insert',
  'gen_nickname',
  'fn_cancel_room',
  'cancel_empty_room',
  'notif_weekly_summary_tick'
);
-- 9건 모두 리턴해야 함. `cancel_empty_room` 는 059 정의 (fn_ prefix 없음).

-- notify trigger + cancel trigger 확인
SELECT tgname FROM pg_trigger
WHERE tgname IN (
  'tg_participant_entered_notify',
  'tg_room_canceled_notify',
  'trg_cancel_empty_room',
  'tg_participant_join_notify'
);
-- 4건 모두 리턴해야 함 (tg_participant_join_notify 는 070 신규).
-- 이름 주의 — 058 은 `_notify` suffix (`_silent` 아님).
-- 059 는 prefix `trg_` (다른 트리거들의 `tg_` 아님).

-- 070 fn_participant_join_notify 함수 최종 상태 (visible push 확인)
SELECT pg_get_functiondef(oid) FROM pg_proc WHERE proname = 'fn_participant_join_notify';
-- payload 안 title/body 있어야 함 · '_silent' 없어야 함

-- 064 fn_room_canceled_notify 함수 최종 상태 (카피 폴리시 확인)
SELECT pg_get_functiondef(oid) FROM pg_proc WHERE proname = 'fn_room_canceled_notify';
-- notif_title 분기 있어야 함 (nickname 없으면 '예약이 취소되었어요')
-- body '괜찮아요, 다시 준비해봐요' 확인

-- cron 등록 확인
SELECT jobname, schedule FROM cron.job ORDER BY jobname;

-- Guide 프로필 최종 확인
SELECT p.nickname, u.id
FROM profiles p
JOIN auth.users u ON u.id = p.id
WHERE u.id = '00000000-0000-0000-0000-000000000001';

-- 신규 컬럼 존재 확인 (065·066·067)
SELECT column_name FROM information_schema.columns
WHERE table_schema='public' AND table_name='profiles'
  AND column_name IN ('bio', 'avatar_color_index', 'max_streak_days', 'total_focus_sec');
-- 4건 모두 리턴해야 함

-- 삭제 컬럼 확인 (068·069, 068 skip 시 seen_tooltips 만)
SELECT column_name FROM information_schema.columns
WHERE table_schema='public' AND table_name='profiles'
  AND column_name IN ('preferred_partner_ids', 'seen_tooltips');
-- 0 rows 여야 함 (068 skip 시 preferred_partner_ids 는 남음)

SELECT column_name FROM information_schema.columns
WHERE table_schema='public' AND table_name='match_queue'
  AND column_name IN ('prefer_favorites');
-- 0 rows 여야 함 (068 skip 시 남음)

-- 신규 rooms 컬럼 확인 (060·063)
SELECT column_name FROM information_schema.columns
WHERE table_schema='public' AND table_name='rooms'
  AND column_name IN ('match_after_at', 'match_pool_user_ids', 'duration_secs');
-- 3건 모두 리턴해야 함
```

---

## 배포 순서 요약

```
Step 0: 사전 확인
  0-1 마지막 version = 047 확인
  0-2 Guide 프로필 존재 확인
  0-3 GUIDE_USER_ID EF 시크릿 확인
  0-4 retro cron 삭제 (있으면)
  0-5 클라 프로덕션 버전 확인 (068 배포 조건 판단)
  0-6 기존 cron 목록 확인
Step 1: 057 Dashboard 적용 → schema_migrations tracking row 삽입
Step 2: supabase db push (dry-run 먼저, 이상 없으면 실행)
        - 068 배포 조건 미충족 시 067 까지만
        - 070 (partner_join visible push) · 064 (cancel 카피 폴리시 병합) 자동 포함
Step 3: 049 sticker 데이터 확인
Step 4: cron 3개 등록 (streak/done_recap/stale_rooms)
Step 4.5: match-tick jobname rename (있는 경우만)
Step 5: send-push + match-tick EF 배포
Step 6: 사후 검증 쿼리 전체 통과 확인
```

---

## 068 skip 시 후속 배포 (§18 client 재심사 통과 후)

클라 v1.0.0+9 이상 App Store · Google Play 재심사 통과 확인 후:

```bash
supabase db push --project-ref [prod-project-ref]
# → 068_drop_favorites_columns.sql + 069_drop_seen_tooltips.sql (skip 상태에서 이어붙임)
```

사후 검증 — Step 6 삭제 컬럼 확인 쿼리 재실행 → `preferred_partner_ids` · `prefer_favorites` · `seen_tooltips` 모두 0 rows.
