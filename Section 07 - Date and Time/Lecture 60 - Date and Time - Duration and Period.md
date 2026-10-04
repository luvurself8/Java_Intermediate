# Lecture 60 — Date and Time: Duration and Period

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (6/13)

Previous: [Lecture 59 - Instant](./Lecture%2059%20-%20Date%20and%20Time%20-%20Instant.md)

Everything covered so far — `LocalDateTime`, `ZonedDateTime`, `Instant` — represents a **specific
point in time**. This lecture covers the other half of the time concept: a **span, or amount, of
time** — via `Period` and `Duration`.

---

## 1. Two kinds of "time": a point vs. a span

> "시간의 개념은 크게 두 가지로 표현할 수가 있거든요."

- **A specific point in time** (a moment): "This project must finish by 2013-08-16." "The next
  meeting is at 11:30." "My birthday is August 16th." — everything covered up to this lecture
  (`LocalDateTime`, `ZonedDateTime`, `Instant`) falls into this category.
- **A span/interval of time** (an amount): "I still have 4 more years to study." "This project has 3
  months left." "Boil the ramen for 3 minutes." The English phrase for this is **"amount of
  time."**
- `Period` and `Duration` both express this second concept — a span, not a point.

| | Unit | Used for | Key methods |
|---|---|---|---|
| `Period` | Years, Months, Days | Dates | `getYears()`, `getMonths()`, `getDays()` |
| `Duration` | Hours, Minutes, Seconds, Nanos | Time | `toHours()`, `toMinutes()`, `getSeconds()`, `getNano()` |

More examples from the slide:

- `Period` use cases: "This project will take about 3 months." "There are 183 days left until the
  anniversary." "The span between a project's start and end date."
- `Duration` use cases: "Boiling ramen takes 3 minutes." "The movie runs for 2 hours 30 minutes."
  "Seoul to Busan takes 4 hours."

---

## 2. Why `Period` uses `get...()` but `Duration` uses `to...()`

```java
public class Period {
    private final int years;
    private final int months;
    private final int days;
}
```

```java
public class Duration {
    private final long seconds;
    private final int nanos;
}
```

> "get은 보통 내가 가지고 있는 속성을 바로 반환하는 거예요. 근데 to는 뭐냐면 뭔가 계산을 해야 돼.
> 해서 반환한 거예요."

- `Period` literally stores `years`, `months`, and `days` as separate fields internally — so
  `getYears()` / `getMonths()` / `getDays()` just **return the stored value directly**, hence `get`.
- `Duration` only stores `seconds` and `nanos` internally — there's no `hours`/`minutes` field to
  return as-is. Getting an hour or minute value requires **computing** it (dividing seconds by 60 for
  minutes, by 3600 for hours), hence the method name `toHours()` / `toMinutes()` rather than `get`.
- `getSeconds()` and `getNano()` on `Duration` *are* plain `get`-style, since those two **are**
  actually stored fields.
- This naming distinction (`get` = direct field access, `to` = computed/converted value) is a small
  but deliberate API design signal worth noticing.

---

## 3. `Period` — date spans (years / months / days)

```java
package time;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.Period;

public class PeriodMain {

    public static void main(String[] args) {
        //생성
        Period period = Period.ofDays(10);
        System.out.println("period = " + period);

        //계산에 사용
        LocalDate currentDate = LocalDate.of(2030, 1, 1);
        LocalDate plusDate = currentDate.plus(period);
        System.out.println("currentDate = " + currentDate);
        System.out.println("plusDate = " + plusDate);

        //기간 차이
        LocalDate startDate = LocalDate.of(2023, 1, 1);
        LocalDate endDate = LocalDate.of(2023, 4, 2);
        Period between = Period.between(startDate, endDate);
        System.out.println("기간: " + between.getMonths() +"개월 " + between.getDays() + "일");
    }
}
```

**Output:**

```
period = P10D
currentDate = 2030-01-01
plusDate = 2030-01-11
기간: 3개월 1일
```

### 3.1 Creating a `Period`

- **`Period.of(years, months, days)`** and its shorthand siblings — **`ofDays(10)`**,
  `ofMonths(...)`, `ofYears(...)` — all build a `Period` representing a fixed span.
- `Period.ofDays(10)` prints as `P10D` — ISO-8601 duration notation: `P` (period) followed by the
  amount and unit (`10D` = 10 days).

### 3.2 Using a `Period` in a calculation

> "2030년 1월 1일에 10일을 더하면 2030년 1월 11일이 된다. 라고 표현할 때 특정 날짜에 10일이라는
> 기간을 더할 수 있다."

- `currentDate.plus(period)` adds the 10-day span directly onto a `LocalDate`: `2030-01-01` becomes
  `2030-01-11`.
- `plus(...)` here accepts a `Period` because of its parent interface, `TemporalAmount` (covered in
  the next lecture) — pressing Ctrl-P / Cmd-P on the parameter reveals the `TemporalAmount` type
  rather than `Period` directly.
- Same immutability rule as everywhere else in this API: `plus(...)` returns a **new** `LocalDate`;
  `currentDate` itself is untouched.

### 3.3 Finding the span between two dates: `Period.between(start, end)`

> "지금 이 기간의 차이를 구해보고 싶어요. 그럼 이거 그냥 계산하기 되게 어렵죠? ... Period.between
> 이라고 있습니다."

- Computing "how much time is between these two dates" by hand would require juggling variable
  month lengths (exactly the kind of problem Lecture 55 warned about) — `Period.between(...)` does
  it directly.
- `Period.between(2023-01-01, 2023-04-02)` → a `Period` representing **3 months, 1 day** —
  retrieved via `between.getMonths()` and `between.getDays()`.

---

## 4. `Duration` — time spans (hours / minutes / seconds / nanos)

```java
package time;

import java.time.Duration;
import java.time.LocalTime;

public class DurationMain {

    public static void main(String[] args) {
        Duration duration = Duration.ofMinutes(30);
        System.out.println("duration = " + duration);

        LocalTime lt = LocalTime.of(1, 0);
        System.out.println("lt = " + lt);

        //계산에 사용
        LocalTime plusTime = lt.plus(duration);
        System.out.println("더한 시간: " + plusTime);

        //시간 차이
        LocalTime start = LocalTime.of(9, 0);
        LocalTime end = LocalTime.of(10, 0);
        Duration between = Duration.between(start, end);
        System.out.println("차이: " + between.getSeconds() + "초");
        System.out.println("근무 시간: " + between.toHours() + "시간" + between.toMinutesPart() + "분");
    }
}
```

**Output:**

```
duration = PT30M
lt = 01:00
더한 시간: 01:30
차이: 3600초
근무 시간: 1시간0분
```

### 4.1 Creating a `Duration`

- **`Duration.of(...)`** and shorthand siblings — **`ofMinutes(30)`**, `ofSeconds(...)`,
  `ofHours(...)` — build a `Duration` representing a fixed time span.
- `Duration.ofMinutes(30)` prints as `PT30M` — same ISO-8601 style as `Period`, but with a `T` before
  the time component (distinguishing a *time* duration from a *date* period), and `M` here means
  **minutes** (not months, since there's a `T` present).

### 4.2 Using a `Duration` in a calculation

- `lt.plus(duration)` adds the 30-minute span to `01:00`, producing `01:30` — same "Lego block"
  composition pattern seen throughout this API.

### 4.3 Finding the span between two times: `Duration.between(start, end)`

- `Duration.between(09:00, 10:00)` → a `Duration` representing exactly 1 hour.
- Reading it back out:
  - **`getSeconds()`** → `3600` (directly stored field — 1 hour = 3600 seconds).
  - **`toHours()`** → `1` — a **computed, total** value (the whole duration expressed in hours).
  - **`toMinutesPart()`** → `0` — **not** the total number of minutes; this is the *remainder* after
    extracting whole hours — i.e. "and how many minutes on top of that," capped below 60.

> "toMinutes 하면 그냥 시간을 다 돌려주는 거고... 미니츠 파트 하면 부분이라는 걸 느껴지시죠? 시간
> 빼고 남은 분."

- The distinction between `toMinutes()` (total minutes across the whole duration) and
  `toMinutesPart()` (just the leftover minutes after whole hours are already accounted for) matters
  for formatting something like "1시간 0분" (1 hour 0 minutes) correctly — using `toMinutes()` instead
  of `toMinutesPart()` here would have returned `60` (the full duration in minutes), not the
  intended "0 minutes left over."

---

## Summary

- Time has two distinct concepts: a **point in time** (everything covered through `Instant`) and a
  **span/amount of time** — `Period` and `Duration` represent the latter.
- `Period`: date-based span (years/months/days) — fields are stored directly, so its accessors are
  `get...()`.
- `Duration`: time-based span (hours/minutes/seconds/nanos) — only `seconds`/`nanos` are actually
  stored, so hour/minute accessors are computed and named `to...()` instead of `get...()`.
- Both support:
  - Creation via `of(...)` / shorthand factories (`ofDays`, `ofMinutes`, etc.).
  - Being added directly to a point-in-time value: `localDate.plus(period)`,
    `localTime.plus(duration)`.
  - Computing the span between two point-in-time values: `Period.between(start, end)`,
    `Duration.between(start, end)`.
- `Duration` has a `toMinutes()` (total minutes across the whole span) vs. `toMinutesPart()`
  (remaining minutes after whole hours are subtracted) distinction — important to use the right one
  when formatting output like "1시간 0분."
- Both print using ISO-8601-style notation: `Period.ofDays(10)` → `P10D`;
  `Duration.ofMinutes(30)` → `PT30M` (the `T` marks it as a time-based duration, distinguishing its
  `M` for minutes from `Period`'s `M` for months).
- Next lecture: the core interfaces behind all of these date/time types (`TemporalAccessor`,
  `Temporal`, `TemporalAmount`).
