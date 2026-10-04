# Lecture 59 — Date and Time: Instant, Machine-Centric Time

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (5/13)

Previous: [Lecture 58 - ZonedDateTime](./Lecture%2058%20-%20Date%20and%20Time%20-%20ZonedDateTime.md)

Everything so far (`LocalDateTime`, `ZonedDateTime`, `OffsetDateTime`) is designed to be read and
understood by humans. `Instant` is different on purpose — it's built for **machines**, and this
lecture explains why that trade-off exists and when it's actually useful.

---

## 1. What `Instant` actually stores

```java
public class Instant {
    private final long seconds;
    private final int nanos;
    ...
}
```

> "Instant는 UTC를 기준으로 하는 시간의 한 지점을 나타냅니다... 쉽게 얘기해서 Instant 내부에는 초
> 데이터만 들어있어요."

- `Instant` represents a single point in time, always relative to **UTC**, at nanosecond precision.
- Internally, that's really just **one number**: `seconds` elapsed since a fixed reference point —
  **1970-01-01T00:00:00 UTC** — plus a `nanos` field for sub-second precision.
- Worked examples straight from the slide:
  - UTC `1970-01-01 00:00:00` → `seconds = 0`.
  - UTC `1970-01-01 00:00:10` → `seconds = 10`.
  - UTC `1970-01-01 00:01:00` → `seconds = 60` (1 minute = 60 seconds).
- There is **no** year/month/day/hour/minute field anywhere inside `Instant` — just "how many
  seconds (and nanoseconds) have passed since that one fixed moment."

### Epoch time / Unix timestamp

> "에포크라는 이 뜻은 뭐냐면 중요한 사건이 발생한 시점을 기준으로 삼는 어떤 기준점을 뜻하는 용어
> 입니다. 인스턴트는 바로 이 에포크 시간을 다루는 클래스입니다."

- This 1970-01-01T00:00:00 UTC reference point is called **epoch time**, also known as the **Unix
  timestamp**.
- It represents time as the total number of elapsed seconds since that moment — a representation
  that is **not tied to any particular time zone**, i.e. an absolute, unambiguous way to express a
  point in time.
- "Epoch" itself just means a fixed reference point used as the starting line for measuring
  something — `Instant` is Java's class for working directly with this epoch-based representation.

---

## 2. Strengths and weaknesses

### Strengths

- **Time-zone independence**: `Instant` is always UTC-based, so it's never affected by time zone.
  > "얘를 쓰면 항상 UTC 기준으로 다 고정이 돼요, 시간이. 그래서 어디서든지 로그를 동시에 딱 찍어도
  > 시간 데이터가 같이 찍히게 됩니다."
  - If a log line is written at the exact same real-world moment from Korea, the UK, and Germany,
    all three `Instant` values come out **identical** — there's no region-dependent representation
    to disagree about.
- **Fixed reference point**: because every `Instant` is measured from the same epoch, time
  calculations and comparisons are unambiguous and consistent.

### Weaknesses

> "사용자 친화적이지 않아요... 날짜와 시간을 계산하고 사용하는 데 필요한 기능이 부족해요. 왜?
> 이 초밖에 없으니까."

- **Not human-friendly**: fine for machine-level time handling, but not intuitive for a person to
  read or reason about directly.
- **Limited date/time calculation support**: since only a raw second count is stored, deriving
  "what day of the week is this" or similar higher-level facts requires manual division/arithmetic
  (÷60 for minutes, ÷24 further for hours, and so on) — there's no built-in notion of calendar
  fields to lean on.
- **No time zone information**: being purely UTC-based means converting to any specific region's
  local date/time requires extra work on top.

---

## 3. When to actually use `Instant`

- **Need a single, global time reference**: e.g. logging events in a way that's consistent no
  matter which region's server wrote them, synchronizing timestamps across servers in different
  countries, database transaction timestamps — anywhere multiple systems need to agree on "when,"
  `Instant`'s UTC-only nature removes ambiguity.
- **Pure elapsed-time calculations, no zone conversion needed**: since the internal value is just a
  second count, subtracting two `Instant`s directly gives elapsed time with no zone-conversion
  complexity involved.
- **Consistent data storage/exchange**: storing or exchanging date/time data using `Instant` means
  every system involved shares the exact same reference point, which helps keep data consistent
  across systems.
- These use cases mostly matter for **global** systems — for typical day-to-day date/time work,
  `LocalDateTime` or `ZonedDateTime` remain the right default.

> "일반적으로 날짜와 시간을 사용할 때는 LocalDateTime, ZonedDateTime 등을 사용하면 됩니다. 인스턴트는
> 날짜를 계산하기 어렵기 때문에 앞서 사용례와 같이 좀 특별한 경우에 한정해서 사용한다고 보시면
> 됩니다."

---

## 4. Code walkthrough

```java
package time;

import java.time.Instant;
import java.time.ZonedDateTime;

public class InstantMain {

    public static void main(String[] args) {
        //생성
        Instant now = Instant.now();//UTC 기준
        System.out.println("now = " + now);

        ZonedDateTime zdt = ZonedDateTime.now();
        Instant from = Instant.from(zdt);
        System.out.println("from = " + from);

        Instant epochStart = Instant.ofEpochSecond(0);
        System.out.println("epochStart = " + epochStart);

        //계산
        Instant later = epochStart.plusSeconds(3600);
        System.out.println("later = " + later);

        //조회
        long laterEpochSecond = later.getEpochSecond();
        System.out.println("laterEpochSecond = " + laterEpochSecond);
    }
}
```

**Output:**

```
now = 2024-02-13T06:46:07.101393Z
from = 2024-02-13T06:46:07.117732Z
epochStart = 1970-01-01T00:00:00Z
later = 1970-01-01T01:00:00Z
laterEpochSecond = 3600
```

### 4.1 Creation

- **`Instant.now()`**: current time, always on a UTC basis.
  - Demonstrated live: the printed value is **not** the instructor's current Korean wall-clock
    time — it's 9 hours earlier, because Korea is UTC+9 and `Instant.now()` always reports in UTC.
  - This is exactly the point: whether printed in Korea, Germany, or the UK, the same real-world
    moment always produces the same `Instant` value.
- **`Instant.from(TemporalAccessor)`**: builds an `Instant` from another date/time type — here, from
  a `ZonedDateTime`. Since `ZonedDateTime` already carries zone info, Java can correctly convert it
  into its UTC equivalent.
  - Important restriction: **`LocalDateTime` cannot be used with `from(...)`** — because
    `LocalDateTime` has no time zone information at all, there's nothing for `Instant` to convert
    *from*. A zone-aware type like `ZonedDateTime` is required.
- **`Instant.ofEpochSecond(long)`**: builds an `Instant` directly from a raw epoch-second count.
  - `ofEpochSecond(0)` → exactly `1970-01-01T00:00:00Z`, the epoch itself.
  - (From the transcript, not shown in the final file) `ofEpochSecond(100)` would print
    `1970-01-01T00:01:40Z` — 100 seconds = 1 minute 40 seconds past the epoch.

### 4.2 Calculation

- **`plusSeconds(3600)`**: adds a raw number of seconds — here, 3600 seconds (1 hour) added to the
  epoch start gives exactly `1970-01-01T01:00:00Z`.
- Like every other date/time type covered so far, `Instant` is **immutable** — the result must be
  captured via the return value (`later = epochStart.plusSeconds(3600)`), same pattern as always.
- Only a small set of simple calculation methods exist (`plusSeconds`, `plusMillis`, `plusNanos`) —
  nothing resembling `plusMonths`/`plusYears`, since there's no calendar-field concept to operate on.

### 4.3 Retrieval

- **`getEpochSecond()`**: returns the number of seconds elapsed since the epoch for this `Instant`.
  - For `later` (which had 3600 seconds added), this correctly returns `3600`.

---

## Summary

- `Instant` represents a single point in time, always relative to **UTC**, stored internally as just
  a `seconds` (+ `nanos`) count elapsed since the epoch, **1970-01-01T00:00:00 UTC** — this same
  underlying concept is also called epoch time / Unix timestamp.
- Strengths: time-zone independence (same real-world moment → identical `Instant` value everywhere)
  and a single fixed reference point that makes calculations/comparisons unambiguous.
- Weaknesses: not human-readable, no built-in calendar-aware calculation support (just raw seconds),
  and no time zone info of its own.
- Good use cases: a global, zone-independent time reference for logging/transactions/server-time
  sync, pure elapsed-time math between two instants, and consistent timestamp storage/exchange
  across systems — mostly relevant for genuinely global systems.
- `Instant.now()` is always UTC — printed values will look "off" by the local UTC offset (e.g. 9
  hours earlier than Korean local time) by design.
- `Instant.from(...)` needs a zone-aware source (e.g. `ZonedDateTime`) — **cannot** be built directly
  from a `LocalDateTime`, which has no zone information to convert from.
- `ofEpochSecond(long)` builds an `Instant` directly from a raw epoch-second count;
  `getEpochSecond()` reads that count back out; `plusSeconds(...)` (and similar) perform simple,
  immutable arithmetic on it.
- Default guidance restated: use `LocalDateTime` as the everyday default, `ZonedDateTime` when
  internationalization/time zones matter, and `Instant` only for the specific UTC/epoch-based needs
  described above.
- Next lecture: expressing a **span** of time — `Duration` and `Period`.
