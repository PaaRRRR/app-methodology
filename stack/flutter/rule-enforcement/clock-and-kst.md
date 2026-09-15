# clock-and-kst — 룰 #11 강제 asset

> 룰 #11 실현. 시간 표현 단일 진실. raw `DateTime.now()` 금지 · 항상 `appNow()`. UTC 저장 · KST 표시.
> 다음 앱에서 그대로 복붙 가능 (한국 시간대 앱이면 즉시).

---

## 왜 이 룰이 있는가

- **DST/타임존 경계 오판** — `.toUtc()` 누락 시 DB `timestamptz` 와 client `DateTime` 비교가 로컬 시간대 기준으로 오판. streak 계산 · 예약 시각 등 자정 경계에서 하루 어긋남
- **자체 offset 정의 남발** — 팀원마다 `Duration(hours: 9)` 하드코딩 → 값 불일치 위험
- **테스트 어려움** — `DateTime.now()` 직접 호출은 mock 불가

**해결.** 단일 진입점 (`appNow`) · 단일 offset 상수 (`kKstOffset`) · 표시 헬퍼 (`toKst`, `formatKst*`).

---

## 파일 구조

```
lib/core/datetime/
├── clock.dart              # appNow()
├── kst.dart                # kKstOffset, toKst, formatKst*
└── format_duration.dart    # 초 → 사람 읽는 문자열 (선택)
```

---

## 1. `clock.dart`

```dart
// 앱 전역에서 UTC 시각을 단일 진실로 반환하는 헬퍼.
//
// 원칙 (CLAUDE.md 규칙 #11) — DB·모델·비교 시점은 UTC, 표시 직전에만 KST 변환.
// raw `DateTime.now()` 직접 호출 금지 — `.toUtc()` 누락 시 DST/타임존 경계 오판정.

/// 현재 시각을 UTC 로 반환.
DateTime appNow() => DateTime.now().toUtc();
```

---

## 2. `kst.dart`

```dart
// UTC DateTime 을 한국 표시용으로 변환하는 포맷 헬퍼.
//
// 원칙 (CLAUDE.md 규칙 #11) — DB·모델은 UTC, 표시 직전에만 KST 변환.

import 'package:intl/intl.dart';

/// 한국 표준시 offset (UTC+09:00). 서머타임 없음. KST→UTC 환산 시 직접 사용.
const Duration kKstOffset = Duration(hours: 9);

/// UTC DateTime 을 KST(+9) DateTime 으로 변환. `.weekday` / `.year` 등 KST 필드 접근 시 사용.
DateTime toKst(DateTime utc) => utc.toUtc().add(kKstOffset);

/// `HH:mm` (예 `14:30`).
String formatKstTime(DateTime utc) =>
    DateFormat('HH:mm').format(toKst(utc));

/// `yyyy-MM-dd HH:mm` (예 `2026-06-04 14:30`).
String formatKstDateTime(DateTime utc) =>
    DateFormat('yyyy-MM-dd HH:mm').format(toKst(utc));

/// `M/d(E)` (예 `7/22(화)`). ko_KR 로케일 필요 — bootstrap 에서 초기화.
String formatKstDate(DateTime utc) {
  final kst = toKst(utc);
  return DateFormat('M/d(E)', 'ko_KR').format(kst);
}

/// `M/d 요 HH:mm` (예 `7/22 화 21:00`). ko_KR 로케일 필요.
String formatKstReservationDate(DateTime utc) {
  final kst = toKst(utc);
  final date = DateFormat('M/d', 'ko_KR').format(kst);
  final day = DateFormat.EEEE('ko_KR').format(kst).substring(0, 1);
  final time = DateFormat('HH:mm').format(kst);
  return '$date $day $time';
}
```

---

## 3. bootstrap 에서 locale 초기화

```dart
// lib/core/bootstrap.dart 안
import 'package:intl/date_symbol_data_local.dart';
await initializeDateFormatting('ko_KR', null);
```

---

## 4. 검증 (룰 강제)

세션 마무리 시 grep 으로 raw `DateTime.now()` 남아있는지 확인.

```bash
grep -rnF "DateTime.now()" lib/ | grep -v clock.dart
```

**결과가 0 이어야 함.** 나오면 즉시 `appNow()` 로 교체.

---

## 5. 다른 시간대 앱

- 한국 외 시간대 앱 = `kKstOffset` 대신 `kAppOffset` 로 명명 · 값만 조정
- 또는 `intl` `getLocation` 활용해 IANA 시간대 지원
- 원칙 (UTC 저장 · 표시 직전 변환) 은 그대로 유지

---

## 6. 관련 룰

- 룰 #4 — 모든 시간 UTC 저장, 표시 KST (개념)
- 룰 #11 — 구현 강제 (`appNow()` · `kKstOffset`)
