<!--
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ 참조 전용 예시 — 그대로 복사 금지.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

이 파일은 StartRoom 실 amplitude event 카탈로그다.
16개 화면 · 143개 event · cleanup 9원칙 · close→재진입 재fire 패턴 등 실 운영 반영.

**신규 앱 사용 방식 —**
1. 자기 앱 화면·event 로 처음부터 새로 작성.
2. Template scaffold 는 `/docs-templates/06_ops/amplitude-taxonomy/catalog.template.md` — 네이밍 규칙·§ 구조·cleanup 원칙만.
3. 이 파일은 "이 정도 상세로 event 카탈로그 만드는구나" 감 참고용.
   - Naming convention (`{screen}__{action}__{object}`) · __view umbrella 규칙 · close→재진입 재fire 패턴 · super property 활용 등.

**주의 —**
- 143개 event · 화면 이름 (home · room · matched · scrum 등) · property (state_type 등) 는 StartRoom 특화.
- 자기 앱은 자기 화면·event 로 재작성.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-->

# amplitude-taxonomy — event catalog

> Amplitude 트래킹 단일 진실. 모든 이벤트 / user property / super property 정의 한 곳.

## 컨벤션

- **Naming** — `{screen}__{action}__{object}` (double `__` 구분, 내부 single `_`).
- **Action 4종** — `view` / `click` / `close` / `input`.
- **Length** — 최대 40자. 길면 줄여서 (allow_notif, notif_toggle 등).
- **Chars** — 영문 소문자 + 숫자 + underscore.
- **Funnel** — Amplitude 대시보드에서 raw event 조합. 트래킹 코드는 raw event 전수.

## `__view` umbrella 규칙

`__view` = "사용자가 의미 있게 화면/sheet 진입 또는 큰 상태 전환을 한 시점". 단순 step/phase 진행은 sub-event 만.

| 패턴 | fire 시점 | 적용 화면 |
|---|---|---|
| **mount 1회** | screen/sheet/cubit init 1회 | onboarding / create / matched / finish / partner_sheet / share_sheet / notifications / settings / settings_blocks / settings_favorites / report / report_reason / create_done / room |
| **상태 전환마다** | umbrella 자체가 상태 추적용 | home (state_type) / join (hub_state) / stats (week_offset) |

### close → 재진입 시 재 fire 패턴

같은 session 안에서 사용자가 `__close` (뒤로가기 / X) 후 같은 화면 재진입 → 새 cubit instance 생성 → **mount `__view` + 그동안 진행된 phase/step sub-event 모두 다시 fire**. 특히 room 의 경우 DB ready 상태가 유지되어 phase 머신 (joining → waitingPartner → scrumChat ...) 빠르게 자동 진행하며 각 phase sub-event 재 fire.

예시 — room close 후 재진입 시 한 session 내:
```
1차: room__view → room__view__waiting_partner → ... → room__view__scrum_chat → room__close
2차: room__view → room__view__waiting_partner → room__view__scrum_chat → (이어서 진행)
```

이건 raw event 진실 (사용자가 실제로 떠났다 다시 진입). 분석 시 처리:
- **방 진입 횟수 (총)** — 그대로 카운트 (사용자 행동 신호).
- **Unique 방 진입** — Amplitude chart "Unique by `room_id`" 옵션 또는 funnel `Uniques` mode 로 dedup.
- **Funnel (waiting → scrum → countdown → live)** — `Uniques per User` mode 로 user 단위 dedup.
- **이탈 분석** — 첫 진입 후 close = 일시 이탈 신호. funnel 정의 시 의도된 신호.

원칙 — 트래킹 코드는 raw event 전수, dedup/dedup mode 는 대시보드 funnel 정의 단계 (memory `feedback_amplitude_no_funnel_priority` 정합).

## Cleanup 원칙

1. **상태 분기 = property** — `home__view__empty/reserved/soon/missed` 같은 같은 화면의 상태 분기는 view event 1개 + `state_type` property 로 통합.
2. **애니메이션 = drop** — UI 애니메이션 (backdrop_fade / modal_enter / avatar_overshoot) drop.
3. **정적 컨텐츠 = drop** — screen view 시 자동 노출되는 정적 컨텐츠 (retro_self_body / sticker_grid / metric_pills 등) drop. interaction object 만.
4. **반복 slot = property** — `room__input__task_0/1/2` → `room__input__task` + `slot_index`.
5. **키보드 이벤트 = drop** — `code_cell_input`, `backspace`, 텍스트필드 매 char drop. 입력 완료 (debounce 500ms) 시 1회만.
6. **모달/시트 = 별도 screen** — partner_sheet, share_sheet, report, report_reason 등.
7. **Transit phase = drop** — 1초 이내 자동 transit phase (joining, both_present) drop. 사용자 의사결정 단계 X.
8. **Duplicate view event = drop** — state property 와 중복되는 view sub-event drop (예: hub_state=denied 와 `__view__permission_denied`).
9. **App lifecycle (background/foreground) = drop** — Amplitude SDK 자동 session 분석으로 cover.

---

## Table 1 — Screens (16개)

| screen | 한글 | 소스 | Bloc/Cubit |
|---|---|---|---|
| onboarding | 온보딩 | `features/onboarding/` | OnboardingCubit |
| home | 홈 | `features/home/` | HomeCubit |
| create | 시작방 만들기 (3 step) | `features/create/` | CreateCubit |
| create_done | 시작방 생성 완료 | `features/create/` | CreateCubit |
| join | 방코드 허브 (QR + 수동) | `features/join/` | JoinHubCubit, JoinCubit |
| matched | 매칭 완료 모달 | `features/matched/` | (state-only) |
| room | 세션 (joining → live → finishing) | `features/room/` | RoomBloc |
| finish | 회고 | `features/finish/` | FinishCubit |
| partner_sheet | 회고 상대방 전달 시트 | `features/finish/` | FinishCubit (공유) |
| stats | 통계 | `features/stats/` | StatsCubit |
| share_card | 공유 카드 시트 (Stats + Finish 공용) | `shared/widgets/share_card_sheet.dart` | — (source property 로 분기) |
| notifications | 알림 피드 시트 | `features/notifications/` | NotificationsCubit |
| settings | 설정 | `features/settings/` | SettingsCubit |
| settings_blocks | 차단 목록 | `features/settings/` | SettingsCubit (공유) |
| settings_favorites | 즐겨찾기 파트너 목록 | `features/settings/` | SettingsCubit (공유) |
| report | 신고 시트 (action picker) | `features/moderation/` | (호출형) |
| report_reason | 신고 사유 선택 시트 | `features/moderation/` | (호출형) |

dev / debug 화면 — taxonomy 대상 X (flavor=prod 만 트래킹).

---

## Table 2 — Events (143개)

### onboarding (9)

| event | properties | 위치 |
|---|---|---|
| onboarding__view | — | screen mount 1회 (cubit constructor) |
| onboarding__view__welcome | — | step 0 진입 (mount 또는 prev) |
| onboarding__view__noti_permission | — | step 1 진입 |
| onboarding__view__ready | — | step 2 진입 |
| onboarding__click__next | step: 0 | step 0 [다음] |
| onboarding__click__allow_notif | granted: bool | step 1 [알림 허용] |
| onboarding__click__skip_notif | — | step 1 [나중에] |
| onboarding__click__create | — | step 2 [시작방 만들기] |
| onboarding__click__join | — | step 2 [방 코드로 참가] |

### home (9)

> §3.5 IA 재설계 정합 — 단일 타임라인 피드 + ActionBar + CreateEntry.
> 옛 이벤트 삭제: `home__click__join` / `cancel_room` / `reschedule_room` / `repeat_room` / `other_reservations` (§3.5 이후 UI에서 사라짐).

| event | properties | 위치 |
|---|---|---|
| home__view | state_type: loading\|empty\|loaded\|error, unread_count: int | HomeCubit state emit 시 (`_trackHomeView`) |
| home__click__create | source: empty\|feed | FeedEmpty [첫 세션 시작하기] (empty) / CreateEntry [시작하기] compact·expanded (feed) |
| home__click__enter_room | room_id: str, partner_type: pending\|guide\|user | ActionBar [입장하기] (live 또는 선택된 임박 세션 진입) |
| home__click__stats | — | 헤더 [통계] |
| home__click__notifications | unread_count: int | 헤더 [알림] 벨 |
| home__click__settings | — | 헤더 [설정] |
| home__click__profile | — | 헤더 [프로필] |
| home__click__retry | error_type: str | Error 상태 [재시도] |
| home__select__imminent | room_id: str | 입장 가능한 임박 카드가 2개 이상일 때 카드 몸통 탭 (ActionBar 대상 이동) |

### create (23) — 회차 D §11 v2 정합

옛 이벤트 삭제 (§11): `create__click__when_now` / `when_soon` / `when_schedule` (chip → wheel), `create__input__schedule_day` / `schedule_time` (chip → wheel), `create__click__partner_favorite` (즐겨찾기 폐지).

| event | properties | 위치 |
|---|---|---|
| create__view | — | screen mount 1회 (cubit constructor) |
| create__view__when | — | step 0 진입 |
| create__view__type | — | step 1 진입 |
| create__view__partner | — | step 2 진입 |
| create__input__wheel_slot | day_index: int, slot_index: int | wheel day/slot 변경 (setDaySlot) |
| create__click__quick_jump | jump_label: 지금\|30분 뒤\|1시간 뒤\|저녁 7시\|내일 아침 | 빠른 이동 칩 |
| create__click__type_dev | — | step 1 [개발] |
| create__click__type_write | — | step 1 [글쓰기] |
| create__click__type_work | — | step 1 [일] |
| create__click__type_study | — | step 1 [공부] |
| create__click__type_exercise | — | step 1 [운동] |
| create__click__type_custom | — | step 1 [직접 입력] |
| create__input__custom_name | text_length: int | custom 이름 입력 완료 (debounce 500ms) |
| create__click__partner_match | — | step 2 [파트너 매칭] |
| create__click__partner_invite | — | step 2 [친구 초대] |
| create__click__partner_guide | — | step 2 [Guide와 시작] |
| create__click__scope | scope: any\|pool | match 하위 스코프 세그먼트 [누구든지] / [아는 사람 중에서] |
| create__input__pool | pool_count: int | PeopleSheet 확인 후 pool 최종 반영 |
| create__click__prev | from_step: 1\|2 | [이전] |
| create__click__next | from_step: 0\|1 | [다음] |
| create__click__submit | partner_type, room_type | step 2 [시작방 만들기] |
| create__close | exit_step: 0\|1\|2 | [×] / back |

### people_sheet (4) — 회차 D §11 pool 다중 선택

| event | properties | 위치 |
|---|---|---|
| people_sheet__view | item_count: int | 시트 오픈 (showPeopleSheet 진입) |
| people_sheet__click__toggle | user_id: str, selected: bool | 파트너 체크 토글 |
| people_sheet__click__confirm | pool_count: int | 하단 [N명 선택 완료] |
| people_sheet__close | — | 시트 dismiss (whenComplete) |

### create_done (4)

| event | properties | 위치 |
|---|---|---|
| create_done__view | room_id: str, partner_type | screen mount |
| create_done__click__copy_link | room_id: str | invite 의 [링크 복사] (_LinkRow) |
| create_done__click__copy_code | room_id: str | invite 의 [코드 복사] (_CodeRow) |
| create_done__click__done | room_id: str | [홈으로] → 홈 |

### manage_sheet (7) — 회차 D §10 홈 카드 예약 관리

| event | properties | 위치 |
|---|---|---|
| manage_sheet__view | room_id: str, matched: bool | showHomeFeedManageSheet 진입 |
| manage_sheet__click__edit_time | room_id: str | menu [시간 수정] |
| manage_sheet__click__cancel | room_id: str | menu [예약 취소] |
| manage_sheet__click__cancel_confirm | room_id: str, soon: bool | cancel view [예약 취소하기] (soon = secs≤600 & matched) |
| manage_sheet__click__save_time | room_id: str | time view [이 시간으로 바꾸기] |
| manage_sheet__click__invite | room_id: str | menu [아는 사람 부르기] |
| manage_sheet__close | room_id: str | 시트 dismiss (whenComplete) |

### invite_sheet (4) — 회차 D §12 SrInviteSheet 공용

| event | properties | 위치 |
|---|---|---|
| invite_sheet__view | invite_source: manage_sheet\|... | showSrInviteSheet 진입 |
| invite_sheet__click__copy | invite_source: str | 링크 복사 chip |
| invite_sheet__click__share | invite_source: str | [공유하기] |
| invite_sheet__close | invite_source: str | 시트 dismiss (whenComplete) |

### profile / profile_edit (7) — 회차 D §15 프로필 재구성

| event | properties | 위치 |
|---|---|---|
| profile__view | — | 화면 mount 1회 |
| profile__click__edit_my_info | — | 내 정보 row 탭 (avatar/nickname/bio) |
| profile__click__add_invite | — | 함께하는 사람들 헤더 [+ 초대] |
| profile__click__partner | partner_id: str | 파트너 row 탭 (drilldown 진입) |
| profile_edit__view | — | 편집 화면 mount |
| profile_edit__click__save | has_bio: bool, color_index: int | [저장] |
| profile_edit__close | — | [×] 헤더 닫기 |

### join (13)

| event | properties | 위치 |
|---|---|---|
| join__view | hub_state: priming\|scanning\|denied\|failed | JoinHubCubit emit |
| join__view__qr_detected | code: str | QR 자동 인식 팝 노출 |
| join__click__scan | — | [카메라 허용하고 스캔] |
| join__click__settings_camera | — | [설정에서 허용] |
| join__click__manual_input | — | [코드 직접 입력] |
| join__click__close | — | [×] 닫기 |
| join__view__code_sheet | initial_code: str? | 코드 입력 시트 오픈 |
| join__input__code | code_length: int, source: typed\|pasted | 6자리 입력 완료 (debounce) |
| join__view__preview | room_id: str, kind: now\|scheduled, is_self: bool | 미리보기 시트 |
| join__click__join_now | room_id: str | [지금 입장] |
| join__click__reserve | room_id: str | [예약 참가] |
| join__click__home_preview | room_id: str | [홈으로] (is_self=true) |
| join__click__retry_code | — | 에러 [코드 다시 입력] |

### matched (4)

| event | properties | 위치 |
|---|---|---|
| matched__view | room_id: str, is_guide: bool, partner_name?: str | screen mount |
| matched__click__accept | room_id: str, is_guide: bool | [좋아요!] / [Guide와 시작] |
| matched__click__backdrop | room_id: str | 배경 dismiss |
| matched__close | room_id: str | [×] / back |

### room (17, transit phase 2개 + click__scrum_send 1개 + view__partner_left 1개 drop)

| event | properties | 위치 |
|---|---|---|
| room__view | room_id: str, initial_phase: str | screen mount 1회 (RoomBloc `_onEntered`) |
| room__view__waiting_partner | room_id, partner_type | phase=waitingPartner 진입 |
| room__view__scrum_chat | room_id | phase=scrumChat 진입 |
| room__view__scrum_countdown | room_id, countdown_sec: int | phase=scrumCountdown 진입 |
| room__view__live | room_id, planned_duration_min: int | phase=live 진입 |
| room__input__task | text_length: int | 할 일 입력 완료 (debounce) |
| room__click__ready_waiting | room_id, task_count: int | [준비 완료] (waiting) |
| room__input__scrum_message | text_length: int | 스크럼 메시지 전송 ([전송] 누르는 시점 = input 1회) |
| room__click__scrum_quick_chip | chip_text: 좋아요, 가봅시다\|오늘도 화이팅\|준비됐어요\|굿굿 | 빠른 메시지 칩 (`_quickChips` 4종) |
| room__click__scrum_ready | — | 스크럼 [준비 완료] |
| room__click__task_toggle | task_id: str, is_done: bool | live 할 일 토글 |
| room__click__finish | room_id, elapsed_min: int | [마치기] |
| room__click__menu | room_id | 헤더 메뉴 |
| room__click__leave | room_id, phase: str | [나가기] 확인 |
| room__close | room_id, exit_phase: str | back / pop |
| room__click__history | item_count: int | 이벤트 피드 박스 탭 → 히스토리 시트 오픈 |
| room__click__reaction | room_id: str, sticker_type: str | live 세션 중 스티커 리액션 전송 |

### finish (10)

| event | properties | 위치 |
|---|---|---|
| finish__view | room_id: str, partner_present: bool | screen mount |
| finish__view__letter | room_id: str, is_guide: bool | 편지 시트 표시 (loadAll 완료 시 letterSheetOpen=true) |
| finish__input__note | text_length: int | 회고 노트 입력 완료 (retro_self_body) |
| finish__click__partner_sheet_open | room_id | [상대방에게 전달] (메인 화면 CTA) |
| finish__click__reschedule | room_id | [내일도 같은 시간에] submitFinish |
| finish__close | room_id | X 버튼 / back (별도 [홈으로] CTA 없음 — submitFinish 후 자동 go('/')) |
| finish__click__add_favorite | user_id: str | partner row 별 아이콘 탭 → 즐겨찾기 추가 |
| finish__click__reveal_letter | room_id: str | 편지 시트 탭 → 파트너 회고 내용 reveal |
| finish__close__letter | room_id: str, revealed: bool | 편지 시트 backdrop/swipe/버튼 닫기 (미열람 여부 포함) |
| finish__click__share | — | AppBar 공유 아이콘 / ghost 버튼 탭 → ShareCardSheet 오픈 |

### partner_sheet (6, click__sticker 포함)

| event | properties | 위치 |
|---|---|---|
| partner_sheet__view | room_id, partner_role: user\|guide | 시트 오픈 |
| partner_sheet__click__sticker | sticker_type: str | 스티커 선택 (sticker_grid) |
| partner_sheet__input__thanks | text_length: int | 고마움 메시지 입력 완료 (debounce) |
| partner_sheet__click__send | room_id, has_sticker: bool, has_thanks: bool | [전달하기] |
| partner_sheet__click__later | — | [나중에] |
| partner_sheet__close | — | swipe / backdrop |

### stats (5)

| event | properties | 위치 |
|---|---|---|
| stats__view | week_offset: int | screen mount + week 전환 |
| stats__click__prev_week | week_offset: int | [이전 주] |
| stats__click__next_week | week_offset: int | [다음 주] |
| stats__click__share | week_offset: int | [공유] → share_sheet |
| stats__close | — | back |

### share_card (3)

| event | properties | 위치 |
|---|---|---|
| share_card__view | source: stats\|finish | 시트 오픈 (`ShareCardSheet.initState`) |
| share_card__save | source: stats\|finish | [이미지 저장] 탭 |
| share_card__share | source: stats\|finish | [공유하기] 탭 |

### notifications (4)

| event | properties | 위치 |
|---|---|---|
| notifications__view | unread_count: int, item_count: int | 시트 오픈 |
| notifications__click__item | notif_type: match\|done_recap\|weekly_summary\|streak\|... | 알림 항목 탭 |
| notifications__click__mark_all_read | unread_count: int | [모두 읽음] |
| notifications__close | — | swipe / backdrop |

### settings (15)

| event | properties | 위치 |
|---|---|---|
| settings__view | — | screen mount |
| settings__click__request_noti | granted: bool | [알림 허용 요청] |
| settings__click__request_camera | granted: bool | [카메라 허용 요청] |
| settings__click__notif_toggle | category: match\|countdown\|done_recap\|streak\|..., enabled: bool | 알림 카테고리 토글 |
| settings__click__chime_toggle | enabled: bool | 소리 토글 |
| settings__click__auto_match_toggle | enabled: bool | 자동 매칭 토글 |
| settings__click__favorites | — | [즐겨찾기 파트너] |
| settings__click__blocks | — | [차단 목록] |
| settings__click__app_settings | — | [기기 설정 열기] |
| settings__click__terms | — | 약관 |
| settings__click__privacy | — | 개인정보 |
| settings__click__feedback | — | 피드백 (mailto) |
| settings__click__delete_account | — | [계정 삭제] |
| settings__click__confirm_delete | — | 계정 삭제 확인 |
| settings__close | — | back |

### settings_blocks (3)

| event | properties | 위치 |
|---|---|---|
| settings_blocks__view | blocked_count: int | screen mount |
| settings_blocks__click__unblock | user_id: str | 차단 해제 |
| settings_blocks__close | — | back |

### settings_favorites (3)

| event | properties | 위치 |
|---|---|---|
| settings_favorites__view | item_count: int | screen mount |
| settings_favorites__click__remove | user_id: str | 즐겨찾기 제거 확인 |
| settings_favorites__close | — | back |

### report (5)

| event | properties | 위치 |
|---|---|---|
| report__view | target_user_id: str, source_screen: room\|finish | 시트 오픈 |
| report__click__report | — | [신고하기] → report_reason |
| report__click__block | target_user_id: str | [차단하기] |
| report__click__confirm_block | target_user_id: str | 차단 확인 다이얼로그 [차단] |
| report__close | — | swipe |

### report_reason (3)

| event | properties | 위치 |
|---|---|---|
| report_reason__view | target_user_id: str | 시트 오픈 |
| report_reason__click__reason | reason: rude\|spam\|illegal\|other, target_user_id: str | 사유 선택 → 제출 |
| report_reason__close | — | swipe / [취소] |

### app lifecycle (4, push__received drop — onMessage 구독 없음)

| event | properties | 위치 |
|---|---|---|
| app_opened | flavor: str, cold_start: bool | bootstrap |
| push__opened | notif_type: str, notif_id?: str | onMessageOpenedApp (`_handleOpen`) |
| deeplink__opened | path: str, kind: qr\|other | DeeplinkService.onLink |
| push__register_failed | error_type: apns_timeout\|token_null\|get_token_error\|upsert_failed | RealPushService `_reportRegisterFailure` — qa-real-device §1 push token 등록 실패 진단 |

---

## Table 3 — User Properties (17개, 영구 상태)

Amplitude `identify()` 로 push. 변화 시점에 갱신.

Profile 모델 진실 (`lib/data/models/profile.dart`) — streakDays / totalRooms / lastCompletedDate 만. first_session_at / partner_count_total / reported_count 는 컬럼 없음 → V2+ 추가 시 같이 박음.

SettingsPrefs 실제 키 9 종 (`lib/core/settings/settings_prefs.dart` SettingKey enum) 과 1:1 매핑.

| property | type | 출처 | 갱신 시점 |
|---|---|---|---|
| onboarding_completed | bool | OnboardingPrefs.isCompleted | 온보딩 완료 시 |
| notif_permission | str (granted\|denied\|prompt) | NotificationPermissionStatus | bootstrap + 권한 변경 시 |
| camera_permission | str | CameraPermission.status | bootstrap + 권한 변경 시 |
| notif_chime_enabled | bool | ChimePrefs | 토글 시 |
| notif_setting_match | bool | SettingsPrefs(notifMatch) | 토글 시 |
| notif_setting_before_1h | bool | SettingsPrefs(notifBefore1h) | 토글 시 |
| notif_setting_before_10m | bool | SettingsPrefs(notifBefore10m) | 토글 시 |
| notif_setting_partner_arrived | bool | SettingsPrefs(notifPartnerArrived) | 토글 시 |
| notif_setting_streak | bool | SettingsPrefs(notifStreak) | 토글 시 |
| notif_setting_retro_reminder | bool | SettingsPrefs(notifRetroReminder) | 토글 시 |
| notif_setting_weekly_summary | bool | SettingsPrefs(notifWeeklySummary) | 토글 시 |
| room_auto_match | bool | SettingsPrefs(roomAutoMatch) | 토글 시 |
| room_activity_signal | bool | SettingsPrefs(roomActivitySignal) | 토글 시 |
| total_rooms | int | profile.totalRooms | finish loadAll 시 |
| current_streak | int | profile.streakDays | finish loadAll 시 |
| last_completed_date | str | profile.lastCompletedDate | finish loadAll 시 |
| blocked_user_count | int | BlocksRepository.list count | blocks screen 진입 시 |

---

## Table 4 — Super Properties (3개, 명시 박음)

| property | 출처 | 비고 |
|---|---|---|
| flavor | AppEnv.flavor | dev / prod |
| app_version | package_info_plus | PackageInfo.version |
| user_id | AuthRepository.currentUser.id | Amplitude setUserId() (내장) |

SDK 자동 — `session_id`, `platform`, `os_version`, `device_model`, `language`, `country` (Amplitude Flutter SDK 가 자동 수집).

---

## 이벤트 수 요약

| 화면 | 이벤트 수 |
|---|---|
| onboarding | 9 |
| home | 12 |
| create | 24 |
| create_done | 4 |
| join | 13 |
| matched | 4 |
| room | 17 |
| finish | 10 |
| partner_sheet | 6 |
| stats | 5 |
| share_card | 3 |
| notifications | 4 |
| settings | 15 |
| settings_blocks | 3 |
| settings_favorites | 3 |
| report | 5 |
| report_reason | 3 |
| lifecycle | 4 |
| **총** | **144** |

> share_card 3개 추가 / finish 1개 추가(finish__click__share) / share_sheet 4개 제거 → net 0 (139). home 4개 추가(cancel_room / reschedule_room / repeat_room / other_reservations) → 143. qa-real-device §1 lifecycle 1개 추가(push__register_failed) → 144.

User property — 17. Super property — 3 명시 (+ SDK 자동 다수).

---

## 정합성 체크

- 모든 event_name ≤ 40자. 최장 — `onboarding__view__noti_permission` = 33자, `settings__click__auto_match_toggle` = 34자, `partner_sheet__click__sticker` = 29자.
- view / click / close / input 4종 외 action 없음.
- 모달/시트 = 별도 screen (partner_sheet / share_sheet / report / report_reason).
- 상태 분기 = property (state_type / phase / hub_state).
- 반복 slot = property (slot_index).
- 정적 컨텐츠 view event = drop.
- 키보드 단위 event = drop. 입력 완료 (debounce 500ms) 시 1회만.
- Transit phase (joining, both_present) = drop.
- Duplicate view event (permission_denied) = drop.
- App lifecycle (background/foreground) = drop.
- `partner_sheet__click__sticker` — finish 메인 화면이 아니라 partner_sheet 안 (`finish/widgets/partner_sheet.dart:352`).
