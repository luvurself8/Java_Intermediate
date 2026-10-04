# Lecture 61 — Date and Time: Core Interfaces

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (7/13)

Previous: [Lecture 60 - Duration and Period](./Lecture%2060%20-%20Date%20and%20Time%20-%20Duration%20and%20Period.md)

Having used `LocalDateTime`, `ZonedDateTime`, `Instant`, `Period`, and `Duration` hands-on, this
lecture steps back to look at the **interfaces** that tie them all together — and introduces two new
supporting concepts, "unit" and "field," needed for the next lecture's query/manipulation methods.

---

## 1. The big split: point in time vs. interval, as interfaces

```
            Specific point in time                    Interval of time
         <interface>                               <interface>
         TemporalAccessor                          TemporalAmount
               ▲                                         ▲
         <interface>                            ┌────────┴────────┐
         Temporal                             Period           Duration
               ▲
    ┌──────────┼──────────┐
LocalDateTime ZonedDateTime Instant
```

- **A specific point in time** → implements **`Temporal`** (which itself extends
  **`TemporalAccessor`**). Implementations: `LocalDateTime`, `LocalDate`, `LocalTime`,
  `ZonedDateTime`, `OffsetDateTime`, `Instant`.
- **An interval of time** → implements **`TemporalAmount`**. Implementations: `Period`, `Duration`.

> "이 시간의 양이라는 표현이 너무 좋아요. 뭔가 시간이 이렇게 딱, 뭔가 양이 차 있는 것 같잖아요."

- Quick mental shortcut: anything named "Temporal" alone (or `TemporalAccessor`) is about a **point**
  in time; anything with "**Amount**" attached is about a **quantity/span** of time.

### 1.1 `TemporalAccessor` vs `Temporal` — read-only vs. read/write

> "TemporalAccessor는 읽기 전용 접근을, Temporal은 읽기와 쓰기 모두를 지원합니다."

- **`TemporalAccessor`**: the base interface for **reading** date/time information — the minimum
  functionality needed to read a specific point in time's data.
- **`Temporal`**: a sub-interface of `TemporalAccessor` that adds **manipulation** (adding,
  subtracting) — letting date/time values actually be changed/adjusted, not just read.
- Since `Temporal` extends `TemporalAccessor`, every `Temporal` implementation automatically carries
  all of `TemporalAccessor`'s read capability too — just with manipulation added on top.

### 1.2 `TemporalAmount`

> "시간의 간격(시간의 양, 기간)을 나타내며, 날짜와 시간 객체에 적용하여 그 객체를 조정할 수 있습니다.
> 예를 들어, 특정 날짜에 일정 기간을 더하거나 빼는 데 사용됩니다."

- This is exactly the interface behind what was seen in Lecture 60: `currentDate.plus(period)` and
  `lt.plus(duration)` both work because `Period` and `Duration` are `TemporalAmount`s that `plus(...)`
  accepts.

---

## 2. A new concept: units and fields

Two more interface/implementation pairs matter for working with date/time data at a finer grain:

| Interface | Implementation | Meaning |
|---|---|---|
| `TemporalUnit` | `ChronoUnit` (enum) | A **unit** of time measurement — e.g. hours, days, months |
| `TemporalField` | `ChronoField` (enum) | A specific **field/slot** within a date/time — e.g. "month of year," "day of month" |

> "유닛은 뭔가 단위야, 시간 단위를 말하는구나. 그러면 템포럴 필드는 시간에 대한 필드, 그게 구현체로
> 크로노 필드."

Both `ChronoUnit` and `ChronoField` are enums (an enum can implement an interface, same as any other
class, as emphasized throughout Section 6) — and in practice, they're essentially the **only**
implementations of their respective interfaces that get used.

---

## 3. `TemporalUnit` / `ChronoUnit` — units of measurement

```java
package time;

import java.time.LocalTime;
import java.time.temporal.ChronoUnit;

public class ChronoUnitMain {

    public static void main(String[] args) {
        ChronoUnit[] values = ChronoUnit.values();
        for (ChronoUnit value : values) {
            System.out.println("value = " + value);
        }
        System.out.println("HOURS = " + ChronoUnit.HOURS);
        System.out.println("HOURS.duration = " + ChronoUnit.HOURS.getDuration().getSeconds());
        System.out.println("DAYS = " + ChronoUnit.DAYS);
        System.out.println("DAYS.duration = " + ChronoUnit.DAYS.getDuration().getSeconds());

        //차이 구하기
        LocalTime lt1 = LocalTime.of(1, 10, 0);
        LocalTime lt2 = LocalTime.of(1, 20, 0);

        long secondsBetween = ChronoUnit.SECONDS.between(lt1, lt2);
        System.out.println("secondsBetween = " + secondsBetween);

        long minutesBetween = ChronoUnit.MINUTES.between(lt1, lt2);
        System.out.println("minutesBetween = " + minutesBetween);
    }
}
```

**Output:**

```
value = Nanos
value = Micros
value = Millis
value = Seconds
value = Minutes
value = Hours
value = HalfDays
value = Days
value = Weeks
value = Months
value = Years
value = Decades
value = Centuries
value = Millennia
value = Eras
value = Forever
HOURS = Hours
HOURS.duration = 3600
DAYS = Days
DAYS.duration = 86400
secondsBetween = 600
minutesBetween = 10
```

### 3.1 What `ChronoUnit` is

- `ChronoUnit` (`java.time.temporal.ChronoUnit`) is an enum covering every unit of time Java's
  date/time API recognizes — time-based units (`NANOS`, `MICROS`, `MILLIS`, `SECONDS`, `MINUTES`,
  `HOURS`) and date-based units (`DAYS`, `WEEKS`, `MONTHS`, `YEARS`, `DECADES`, `CENTURIES`,
  `MILLENNIA`), plus a couple of special ones (`ERAS`, `FOREVER`).
- Its `toString()` is overridden, so printing a constant directly (`ChronoUnit.HOURS`) shows a
  friendly name (`Hours`) rather than the default enum identity output.

### 3.2 Key `ChronoUnit` members and methods

| Method | What it does |
|---|---|
| `getDuration()` | Returns a `Duration` representing how long one unit of this `ChronoUnit` actually is |
| `between(Temporal, Temporal)` | Measures the gap between two `Temporal` values, expressed in this unit |
| `isDateBased()` | Whether this unit is date-based (day/week/month/year, etc.) |
| `isTimeBased()` | Whether this unit is time-based (hour/minute/second, etc.) |
| `isSupportedBy(Temporal)` | Whether a given `Temporal` value supports this unit at all |

- `ChronoUnit.HOURS.getDuration().getSeconds()` → `3600` — one hour **is** 3600 seconds, confirmed
  directly from the API rather than hardcoded.
- `ChronoUnit.DAYS.getDuration().getSeconds()` → `86400` — one day is 86,400 seconds.
- `ChronoUnit.SECONDS.between(lt1, lt2)` between `01:10:00` and `01:20:00` → `600` (10 minutes = 600
  seconds).
- `ChronoUnit.MINUTES.between(lt1, lt2)` on the same pair → `10` (same gap, expressed directly in
  minutes instead).

> "ChronoUnit을 사용하면 두 날짜 또는 시간 사이의 차이를 해당 단위로 쉽게 계산할 수가 있어요."

---

## 4. `TemporalField` / `ChronoField` — specific fields within a date/time

```java
package time;

import java.time.temporal.ChronoField;

public class ChronoFieldMain {

    public static void main(String[] args) {
        ChronoField[] values = ChronoField.values();
        for (ChronoField value : values) {
            System.out.println(value + ", range = " + value.range());
        }

        System.out.println("MONTH_OF_YEAR.range() = " + ChronoField.MONTH_OF_YEAR.range());
        System.out.println("DAY_OF_MONTH.range() = " + ChronoField.DAY_OF_MONTH.range());
    }
}
```

**Output (partial):**

```
MonthOfYear, range = 1 - 12
DayOfMonth, range = 1 - 28/31
SecondOfMinute, range = 0 - 59
DayOfWeek, range = 1 - 7
...
MONTH_OF_YEAR.range() = 1 - 12
DAY_OF_MONTH.range() = 1 - 28/31
```

### 4.1 Unit vs. field — the distinction, worked through an example

> "2024년 8월 16일이다 라고 하면 각각의 필드는 이렇게 돼요: YEAR: 2024, MONTH_OF_YEAR: 8,
> DAY_OF_MONTH: 16."

- A **unit** (`ChronoUnit`) is just a generic measure of time: `DAYS`, `MONTHS`, `YEARS` — it doesn't
  say *where* in a calendar something sits, just how big a span is.
- A **field** (`ChronoField`) identifies **one specific slot** inside a composite date/time value —
  e.g. given `2024-08-16`, the `YEAR` field is `2024`, the `MONTH_OF_YEAR` field is `8`, and the
  `DAY_OF_MONTH` field is `16`.
- Why this distinction matters, worked through directly:

> "우리가 단순히 '일'이라고 하면 그러면 이게 16일이라고 하는 게 8월 16일인지 아니면 뭐 한 300일을
> 말하는 건지... 단순히 일이라고 하면은 그런 개념들이 될 수 있지만, 이 연, 월, 일을 같이 들어갔을
> 때 '일'은 뭘 뜻합니까? day of month, 그러니까 이 월 중에 있는 일이죠. 얘는 31이 넘을 수
> 없죠."

- A plain "day" count is ambiguous — is it the 16th day of the month, or the 229th day of the year?
  `ChronoField.DAY_OF_MONTH` disambiguates exactly which slot is meant, and constrains its valid
  range accordingly (never more than 31).
- Same idea for time: `SecondOfMinute` is explicitly **not** "how many seconds total" — it's "the
  seconds component when expressed as minutes:seconds," so its range is `0`–`59` by definition (you
  never say "3 minutes 100 seconds"; that's 4 minutes 40 seconds instead).

### 4.2 `ChronoField.range()` — valid value ranges per field

- Every `ChronoField` constant exposes `.range()`, showing the legal range of values for that
  field:
  - `MONTH_OF_YEAR.range()` → `1 - 12` (12 months in a year).
  - `DAY_OF_MONTH.range()` → `1 - 28/31` (varies: February can be as short as 28 days, other months
    as long as 31 — this range notation itself reflects that variability).
  - `SecondOfMinute` → `0 - 59`.
  - `DayOfWeek` → `1 - 7`.

> "ChronoField를 사용해야 날짜와 시간의 필드 중에 원하는 데이터를 조회할 수가 있어요."

---

## Summary

- All date/time types split into two interface families: a **point in time** implements `Temporal`
  (extends `TemporalAccessor`); an **interval/span** implements `TemporalAmount`.
  - `TemporalAccessor` = read-only access; `Temporal` (its sub-interface) adds read **and** write
    (manipulation) support.
  - `TemporalAmount`'s implementations (`Period`, `Duration`) are what make `.plus(period)` /
    `.plus(duration)` work on `Temporal` values.
- Two further supporting concepts, needed for the next lecture's query/manipulation methods:
  - **`TemporalUnit` / `ChronoUnit`**: a *unit* of time measurement (hours, days, months, ...).
    `getDuration()` converts a unit into its `Duration` equivalent; `between(a, b)` measures the gap
    between two `Temporal` values directly in that unit.
  - **`TemporalField` / `ChronoField`**: a specific *field/slot* within a composite date/time value
    (e.g. `MONTH_OF_YEAR`, `DAY_OF_MONTH`, `SecondOfMinute`) — disambiguates "which part" is meant
    and constrains its legal range accordingly (`.range()`).
- The key distinction: a **unit** measures a span generically; a **field** identifies one specific
  component's value within a structured date/time, with a range appropriate to that component.
- In practice, `ChronoUnit` and `ChronoField` are essentially the only implementations of
  `TemporalUnit`/`TemporalField` actually used.
- Next lecture puts both of these to work directly: querying and manipulating date/time values using
  fields and units.
