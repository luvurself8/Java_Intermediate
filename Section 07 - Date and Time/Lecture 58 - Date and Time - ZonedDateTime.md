# Lecture 58 — Date and Time: Time Zones, ZonedDateTime

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (4/13)

Previous: [Lecture 57 - LocalDateTime](./Lecture%2057%20-%20Date%20and%20Time%20-%20LocalDateTime.md)

Now that `LocalDate`/`LocalTime`/`LocalDateTime` are solid, this lecture adds the missing piece for
global/multi-region work: time zones, via `ZoneId`, `ZonedDateTime`, and `OffsetDateTime`.

> "ZonedDateTime이나 OffsetDateTime은 글로벌 서비스를 하지 않으면 사실 잘 사용하지 않아요. 따라서
> 이것을 지금부터 너무 깊이 있게 파기보다는 아, 대략 이런 게 있구나 정도의 개념 정도만 알아두시면
> 돼요."

---

## 1. `ZoneId` — how Java represents a time zone

> "아시아/서울과 같은 타임 존 안에는 일광 절약 시간제에 대한 정보와 UTC+9와 같은 UTC로부터의 시간
> 차이인 오프셋 정보를 모두 포함하고 있습니다."

- A `ZoneId` (e.g. `Asia/Seoul`) bundles **two** pieces of information together:
  - The **offset** from UTC (e.g. `+09:00`).
  - Whether/when **DST** applies in that region.

```java
package time;

import java.time.ZoneId;

public class ZoneIdMain {

    public static void main(String[] args) {
        for (String availableZoneId : ZoneId.getAvailableZoneIds()) {
            ZoneId zoneId = ZoneId.of(availableZoneId);
            System.out.println(zoneId + " | " + zoneId.getRules());
        }

        ZoneId zoneId = ZoneId.systemDefault();
        System.out.println("ZoneId.systemDefault = " + zoneId);

        ZoneId seoulZoneId = ZoneId.of("Asia/Seoul");
        System.out.println("seoulZoneId = " + seoulZoneId);
    }
}
```

**Output (abridged):**

```
Europe/London | ZoneRules[currentStandardOffset=Z]
UTC | ZoneRules[currentStandardOffset=Z]
GMT | ZoneRules[currentStandardOffset=Z]
Asia/Seoul | ZoneRules[currentStandardOffset=+09:00]
Asia/Dubai | ZoneRules[currentStandardOffset=+04:00]
US/Arizona | ZoneRules[currentStandardOffset=-07:00]
Asia/Istanbul | ZoneRules[currentStandardOffset=+03:00]
Asia/Shanghai | ZoneRules[currentStandardOffset=+08:00]
...
Europe/Paris | ZoneRules[currentStandardOffset=+01:00]
ZoneId.systemDefault = Asia/Seoul
seoulZoneId = Asia/Seoul
```

- **`ZoneId.getAvailableZoneIds()`**: returns every zone ID Java knows about (the instructor notes
  this returns a `Set<String>` — collections haven't been covered yet in the course, so for now it's
  treated like "basically an array you can loop over" with a for-each).
- **`ZoneId.of(String)`**: builds a `ZoneId` instance from one of those ID strings (e.g.
  `"Asia/Seoul"`) — a typo here is not caught at compile time, only at runtime.
- **`ZoneId.systemDefault()`**: the zone ID the current machine/OS is configured to use — varies per
  machine (the instructor's own machine returns `Asia/Seoul`, since that's usually tied to the OS's
  date/calendar region setting).
- `zoneId.getRules()` exposes the offset, and (digging further, not shown in detail here) the DST
  information too.

---

## 2. `ZonedDateTime` — `LocalDateTime` + `ZoneId`

```java
public class ZonedDateTime {
    private final LocalDateTime dateTime;
    private final ZoneOffset offset;
    private final ZoneId zone;
}
```

> "얘는 뭐냐면, 얘는 LocalDateTime에 시간대 정보인 ZoneId가 합쳐진 거예요."

- Internally, `ZonedDateTime` holds a `LocalDateTime` **plus** a `ZoneId` — plus a `ZoneOffset` field
  too, purely as a convenience so offset lookups don't need to be recomputed from the zone every
  time (the zone alone is already enough to derive the offset).
- Since it carries the zone itself, `ZonedDateTime` automatically knows about DST for that region —
  this is what makes it suitable for **real-world** date/times: meeting times, calendar invites,
  anything two people in different places need to agree on.

```java
package time;

import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;

public class ZonedDateTimeMain {

    public static void main(String[] args) {
        ZonedDateTime nowZdt = ZonedDateTime.now();
        System.out.println("nowZdt = " + nowZdt);

        LocalDateTime ldt = LocalDateTime.of(2030, 1, 1, 13, 30, 50);
        ZonedDateTime zdt1 = ZonedDateTime.of(ldt, ZoneId.of("Asia/Seoul"));
        System.out.println("zdt1 = " + zdt1);

        ZonedDateTime zdt2 = ZonedDateTime.of(2030, 1, 1, 13, 30, 50, 0, ZoneId.of("Asia/Seoul"));
        System.out.println("zdt2 = " + zdt2);

        ZonedDateTime utcZdt = zdt2.withZoneSameInstant(ZoneId.of("UTC"));
        System.out.println("utcZdt = " + utcZdt);
    }
}
```

**Output:**

```
nowZdt = 2024-02-09T12:02:13.457712+09:00[Asia/Seoul]
zdt1 = 2030-01-01T13:30:50+09:00[Asia/Seoul]
zdt2 = 2030-01-01T13:30:50+09:00[Asia/Seoul]
utcZdt = 2030-01-01T04:30:50Z[UTC]
```

### 2.1 Creating a `ZonedDateTime`

- **`now()`**: current date/time, using `ZoneId.systemDefault()` since no zone was given explicitly
  — this is why `nowZdt`'s output ends in `[Asia/Seoul]` even though the zone was never written out.
- **`of(LocalDateTime, ZoneId)`**: Lego-block style again — build a plain `LocalDateTime` first, then
  attach a `ZoneId` to it. Exactly the same composition idea used for `LocalDateTime` itself
  (`LocalDate` + `LocalTime`).
- **`of(year, month, day, hour, minute, second, nano, ZoneId)`**: a longer overload that takes every
  individual field directly, including nanoseconds — noted as fairly verbose, but useful to know it
  exists.

> "레고 블록 조립하듯이 할 수 있죠."

### 2.2 Changing zones: `withZoneSameInstant(ZoneId)`

> "이 메서드를 사용하면 지금 다른 나라는 몇 시인지 확인일 수 있습니다. 예를 들어서 서울이 지금
> 9시라면, UTC 타임존으로 변경하면 0시를 확인할 수 있습니다."

- Motivating question: "it's 9 AM in Korea — what time is it in the UK right now?"
- `withZoneSameInstant(ZoneId.of("UTC"))` answers this directly: it converts the **same instant** in
  time into a different zone's local representation.
- In the example: `zdt2` is `13:30:50+09:00[Asia/Seoul]`. Converting to UTC: `13 − 9 = 4`, so the
  result is `04:30:50Z[UTC]` — same underlying moment, displayed in a different zone.
- This is the go-to method for "what time is it elsewhere right now, for this same moment" questions.

---

## 3. `OffsetDateTime` — `LocalDateTime` + a fixed `ZoneOffset` (no zone)

```java
public class OffsetDateTime {
    private final LocalDateTime dateTime;
    private final ZoneOffset offset;
}
```

> "OffsetDateTime은 방금 버전(ZonedDateTime)에서 기능이 하나 빠진 거예요... zone id가 없어요. zone
> id가 없으면 뭘 할 수 없다? 서머 타임, 그런 DST 같은 일광 절약 시간대가 안 되는 겁니다."

- Compared to `ZonedDateTime`, `OffsetDateTime` is missing the `ZoneId` field entirely — it only
  keeps a fixed `ZoneOffset` (e.g. `+01:00`).
- Without a zone ID, there's no way to derive DST behavior — the offset is just a static number, so
  it never shifts automatically the way a real zone's offset can.

```java
package time;

import java.time.LocalDateTime;
import java.time.OffsetDateTime;
import java.time.ZoneOffset;

public class OffsetDateTimeMain {

    public static void main(String[] args) {
        OffsetDateTime nowOdt = OffsetDateTime.now();
        System.out.println("nowOdt = " + nowOdt);

        LocalDateTime ldt = LocalDateTime.of(2030, 1, 1, 13, 30, 50);
        System.out.println("ldt = " + ldt);
        OffsetDateTime odt = OffsetDateTime.of(ldt, ZoneOffset.of("+01:00"));
        System.out.println("odt = " + odt);
    }
}
```

**Output:**

```
nowOdt = 2024-02-13T15:03:36.422230+09:00
ldt = 2030-01-01T13:30:50
odt = 2030-01-01T13:30:50+01:00
```

- `ZoneOffset.of("+01:00")` builds just the offset value — no region name, no DST rules, nothing
  beyond "this many hours/minutes different from UTC."
- `OffsetDateTime.of(LocalDateTime, ZoneOffset)` attaches that fixed offset to a plain
  `LocalDateTime` — same composition pattern as everywhere else in this API.

---

## 4. `ZonedDateTime` vs `OffsetDateTime` — which one to use

| | Has `ZoneId`? | DST-aware? | Best for |
|---|---|---|---|
| `ZonedDateTime` | Yes | Yes, automatically | Real, region-tied date/times people actually use — scheduling meetings, calendar entries, flight times |
| `OffsetDateTime` | No | No | Cases where you specifically want a **fixed** UTC difference with no DST complexity — e.g. timestamping logs or stored data consistently, without values silently shifting by an hour when DST starts/ends |

> "ZonedDateTime은 구체적인 지역 시간대를 다룰 때 사용하고 일광 절약 시간이 자동으로 처리가
> 됩니다... OffsetDateTime은 UTC와의 시간 차이만을 나타낼 때 쓰는 거예요... 시간대 변환 없이 로그를
> 기록하고 데이터를 저장하고 처리할 때 적합하다."

- Concrete reasoning for `OffsetDateTime`'s use case: if timestamps were stored using a zone-aware
  type, a DST transition partway through the year could make the recorded offset silently jump by an
  hour — undesirable for logs/data you want to compare consistently. A fixed offset avoids that.
- If time-zone-correct behavior (including DST) genuinely matters for the use case, use
  `ZonedDateTime` instead.

> "참고로 여러분, ZonedDateTime이나 OffsetDateTime은 글로벌 서비스를 하지 않으면 사실 잘 사용하지
> 않아요."

- Realistically, most non-global applications will rarely reach for either of these — the
  instructor's advice is to understand the concept now, and dig deeper only if/when an actual global
  project calls for it.

---

## Summary

- `ZoneId` represents a named time zone (e.g. `Asia/Seoul`) and internally bundles **both** the UTC
  offset and DST rules for that region.
  - `ZoneId.getAvailableZoneIds()` lists all known zone IDs; `ZoneId.of(String)` builds one from a
    name; `ZoneId.systemDefault()` returns the machine's own configured zone.
- `ZonedDateTime` = `LocalDateTime` + `ZoneId` (plus a cached `ZoneOffset` for convenience). Because
  it carries the actual zone, it's **DST-aware** — the right choice for real-world, place-tied
  date/times (meetings, calendar events).
  - Build with `of(LocalDateTime, ZoneId)` or the long `of(y, m, d, h, min, s, nano, ZoneId)` form.
  - `withZoneSameInstant(ZoneId)` converts the same instant into another zone's local representation
    — the tool for "what time is it right now, somewhere else?" questions.
- `OffsetDateTime` = `LocalDateTime` + a fixed `ZoneOffset` only — **no** zone ID, so **no** DST
  awareness; the offset never shifts on its own.
  - Build with `OffsetDateTime.of(LocalDateTime, ZoneOffset)`; `ZoneOffset.of("+01:00")` builds the
    offset value itself.
- Choose `ZonedDateTime` when DST-correct, region-tied behavior matters (scheduling); choose
  `OffsetDateTime` when a stable, non-shifting UTC difference is wanted (e.g. consistent log/data
  timestamps).
- Both are mostly relevant only for genuinely global applications — fine to understand at a
  conceptual level for now and revisit in depth if a real global project requires it.
- Next lecture: `Instant`, "machine-centric time."
