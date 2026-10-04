# Lecture 56 — Date and Time: Introducing Java's Date and Time Library

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (2/13)

Previous: [Lecture 55 - Why a Date and Time Library is Needed](./Lecture%2055%20-%20Date%20and%20Time%20-%20Why%20a%20Date%20and%20Time%20Library%20is%20Needed.md)

This lecture is still conceptual — no code yet. The goal is to get oriented across the full
`java.time` class lineup using one summary table, before diving into each class with real examples
starting next lecture.

> "자바 날짜와 시간 라이브러리는 이게 지금 자바 공식 문서가 제공하는 표거든요. 이거를 제가 조금
> 다듬은 건데 이 표 하나로 다 정리할 수가 있어요... 이 자바 공식 문서 중에 제가 볼 때 제일 잘 만든
> 표인 것 같아요."

---

## 1. The one-table overview

Source: [Oracle's `java.time` overview](https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html)
(the instructor's own cleaned-up version of this table):

| Class or Enum | Year | Month | Day | Hours | Minutes | Seconds* | Zone Offset | Zone ID | `toString` Output |
|---|---|---|---|---|---|---|---|---|---|
| `LocalDate` | V | V | V | | | | | | `2013-08-20` |
| `LocalTime` | | | | V | V | V | | | `08:16:26.943` |
| `LocalDateTime` | V | V | V | V | V | V | | | `2013-08-20T08:16:26.937` |
| `ZonedDateTime` | V | V | V | V | V | V | V | V | `2013-08-21T00:16:26.941+09:00[Asia/Tokyo]` |
| `OffsetDateTime` | V | V | V | V | V | V | V | | `2013-08-20T08:16:26.954-07:00` |
| `OffsetTime` | | | | V | V | V | V | | `08:16:26.957-07:00` |
| `Year` | V | | | | | | | | `2013` |
| `Month` | | V | | | | | | | `AUGUST` |
| `YearMonth` | V | V | | | | | | | `2013-08` |
| `MonthDay` | | V | V | | | | | | `--08-20` |
| `Instant` | | | | | | V | | | `2013-08-20T15:16:26.355Z` |
| `Period` | V | V | V | | | *** | | | `P10D` (10 days) |
| `Duration` | | ** | ** | ** | V | | | | `PT20H` (20 hours) |

\* seconds are captured at nanosecond precision (milliseconds or nanoseconds are both possible).
\*\* these classes don't store this unit as a field, but have methods that return a value in that unit.
\*\*\* adding a `Period` to a `ZonedDateTime` respects daylight saving time and other local time
differences.

- Every row is a different *combination* of which date/time components it knows about — once you
  can read this table, you can predict roughly what any of these classes can and can't do.

---

## 2. `LocalDate`, `LocalTime`, `LocalDateTime`

- **`LocalDate`**: represents **date only** — year, month, day. Example: `2013-11-21`.
- **`LocalTime`**: represents **time only** — hour, minute, second (seconds here can include
  milliseconds/nanoseconds). Example: `08:20:30.213`.
- **`LocalDateTime`**: the combination of the two above. Example: `2013-11-21T08:20:30.213`.
  - Looking inside the actual class confirms it literally composes a `LocalDate` and a `LocalTime`
    together.

### Why "Local"?

> "로컬이 뭐냐면 현지의 특정 지역에, 이런 뜻이거든요... 세계 시간대를 고려하지 않아서 타임존이
> 적용이 안 되기 때문에 그래요."

- "Local" (like "local food") means **specific to one place** — these three classes don't account
  for world time zones at all; no time zone is attached.
- Use them when you only care about one region's date/time and never need to convert across time
  zones.
- Example use cases from the slide:
  - Building an app that only serves domestic (Korean) users — no need to reason about other
    countries' time at all, so plain local date/time is enough.
  - Saying "my birthday is August 16th" — there's no notion of "which time zone" attached to a
    birthday; it's just a day.

---

## 3. `ZonedDateTime` and `OffsetDateTime`

Both represent date/time **with time-zone awareness** — but differently:

- **`ZonedDateTime`**: includes a full **time zone** (`ZoneId`), e.g.
  `2013-11-21T08:20:30.213+9:00[Asia/Seoul]`.
  - `+9:00` is the **offset** — the time difference from UTC. Korea is UTC+9.
  - `Asia/Seoul` is the **zone ID** (time zone). Knowing the zone ID is enough to derive the offset
    *and* to know whether DST applies.
  - A time zone object like `Asia/Seoul` internally carries **both** the offset (UTC+9:00) and DST
    information — Korea's doesn't apply DST, so that part is simply absent for Seoul specifically,
    but the mechanism is there for zones that do observe it (e.g. Berlin).
  - Because `ZonedDateTime` knows the actual zone, it can correctly account for DST — this makes it
    the right choice for **real, everyday date/times** (meeting times, flight times, anything
    actually tied to a place).
- **`OffsetDateTime`**: includes only a fixed **offset from UTC** — no zone ID at all. Example:
  `2013-11-21T08:20:30.213+9:00`.
  - Since there's no zone ID, it has no way to know about DST — it only represents "this many hours
    different from UTC," nothing more.
  - Use it when you specifically want a **fixed** offset and don't want (or need) DST-aware
    behavior.

> "ZonedDateTime은 일광 절약 시간제를 함께 처리를 해주고요. 반면에 타임 존을 알 수 없는
> OffsetDateTime은 일광 절약 시간제를 처리하지 못합니다."

| | Has Zone ID? | Has Offset? | DST-aware? | Typical use |
|---|---|---|---|---|
| `ZonedDateTime` | Yes | Yes (derived) | Yes | Real-world date/time (meetings, flights) tied to a place |
| `OffsetDateTime` | No | Yes (fixed) | No | Just need a UTC offset, don't need/want DST logic |

---

## 4. `Year`, `Month`, `YearMonth`, `MonthDay`, `DayOfWeek`

- `Year`, `Month`, `YearMonth`, `MonthDay` each represent a partial slice of a date (just the year,
  just the month, year+month, or month+day). Not used very often in practice.
- `DayOfWeek` represents Monday through Sunday.
  - The instructor pauses to check and confirms live that `DayOfWeek` is implemented as an **enum**
    — a natural fit, since the day of the week is a fixed, closed set of 7 constant values (exactly
    the kind of case Section 6's enum material was all about).

---

## 5. `Instant` — a special, machine-centric point in time

> "Instant는 UTC(협정 세계시)를 기준으로 하는, 시간의 한 지점을 나타낸다... 인스턴트 안에는 초
> 데이터만 들어있어요."

- `Instant` represents date/time at **nanosecond precision**, but internally it's really just a
  single number: the amount of time elapsed since a fixed reference point —
  **1970-01-01T00:00:00 UTC** (the Greenwich Observatory, at UTC).
- Internally, an `Instant` only stores **seconds since that reference point** (down to nanosecond
  precision) — it has no year/month/day/hour/minute fields at all.
- Because of that, `Instant` is **not well-suited for human-facing date/time calculations** (like
  "what day of the week is this" or "add one month") — it's purely a machine-centric point on a
  single continuous timeline.
- It's still useful for other purposes, which will be covered in a later lecture.
- Important: `Instant` is always **UTC-based** by definition.

---

## 6. `Period` and `Duration` — two different notions of "time"

> "우리가 시간이라고 하면 시간의 개념을 사실 크게 두 가지로 표현할 수가 있어요."

Everything covered so far (`LocalDate`, `ZonedDateTime`, `Instant`, etc.) represents a **specific
point in time** — a moment. But there's a second, distinct concept: a **span/interval of time**.

- **A point in time** (a moment): examples from the slide —
  - "This project must be done by 2013-08-16."
  - "The next meeting is at 11:30."
  - "My birthday is August 16th."
  - Each of these names one specific instant — not a length of time.
- **An interval of time** (an amount, a duration): examples —
  - "I still have 4 more years to study."
  - "This project has 3 months left."
  - "Boil the ramen for 3 minutes."
  - These describe **how much** time, not **when**.

The English term for this second concept is **"amount of time"** — a phrase the instructor calls
out as a genuinely intuitive way to think about it.

- **`Period`**: expresses an interval between two **dates**, in **years / months / days**.
- **`Duration`**: expresses an interval between two **times**, in **hours / minutes / seconds**
  (down to nanoseconds).
- The distinction to remember: **`Period` = calendar-based (Y/M/D)**, **`Duration` = clock-based
  (H/M/S)**.

---

## Summary

- Every `java.time` class can be understood by which fields it tracks — year/month/day,
  hour/minute/second, offset, and zone ID — captured in one official-docs-derived table.
- `LocalDate` / `LocalTime` / `LocalDateTime`: no time zone at all ("local" = specific to one place,
  no concept of converting across regions). Use for domestic-only apps or anything inherently
  region-less, like a birthday.
- `ZonedDateTime`: full time zone (`ZoneId`, e.g. `Asia/Seoul`) — knows the UTC offset *and* whether
  DST applies. Best for real-world date/times tied to an actual place (meetings, flights).
- `OffsetDateTime`: fixed UTC offset only, no zone ID, so **no DST awareness**. Use when you want a
  plain, fixed offset and don't need DST logic at all.
- `Year`, `Month`, `YearMonth`, `MonthDay`: rarely-used partial-date classes. `DayOfWeek` is an enum
  representing Monday–Sunday.
- `Instant`: a pure machine timestamp — seconds (nanosecond precision) elapsed since
  1970-01-01T00:00:00 UTC. No calendar fields internally, always UTC, not meant for human-facing
  date arithmetic — more on its actual use cases in a later lecture.
- Two distinct notions of "time" exist: a **point in time** (a moment, e.g. "the meeting is at
  11:30") vs. an **amount of time** / interval (e.g. "3 months left"). `Period` expresses the latter
  in years/months/days; `Duration` expresses it in hours/minutes/seconds.
- Next lecture starts hands-on coverage of the most commonly used type: `LocalDateTime` (and its
  siblings `LocalDate`/`LocalTime`), with real code examples.
