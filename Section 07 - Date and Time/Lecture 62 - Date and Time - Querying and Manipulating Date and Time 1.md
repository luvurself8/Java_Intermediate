# Lecture 62 — Date and Time: Querying and Manipulating Date and Time (1)

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (8/13)

Previous: [Lecture 61 - Core Interfaces](./Lecture%2061%20-%20Date%20and%20Time%20-%20Core%20Interfaces.md)

This lecture puts `ChronoField` and `ChronoUnit` from Lecture 61 to work directly — querying
(`get`) and manipulating (`plus`) date/time values in a way that's consistent across **every**
`java.time` implementation.

---

## 1. Querying with `TemporalAccessor.get(TemporalField field)`

```java
package time;

import java.time.LocalDateTime;
import java.time.temporal.ChronoField;

public class GetTimeMain {

    public static void main(String[] args) {
        LocalDateTime dt = LocalDateTime.of(2030, 1, 1, 13, 30, 59);
        System.out.println("YEAR = " + dt.get(ChronoField.YEAR));
        System.out.println("MONTH_OF_YEAR = " + dt.get(ChronoField.MONTH_OF_YEAR));
        System.out.println("DAY_OF_MONTH = " + dt.get(ChronoField.DAY_OF_MONTH));
        System.out.println("HOUR_OF_DAY = " + dt.get(ChronoField.HOUR_OF_DAY));
        System.out.println("MINUTE_OF_HOUR = " + dt.get(ChronoField.MINUTE_OF_HOUR));
        System.out.println("SECOND_OF_MINUTE = " + dt.get(ChronoField.SECOND_OF_MINUTE));

        System.out.println("편의 메서드 제공");
        System.out.println("YEAR = " + dt.getYear());
        System.out.println("MONTH_OF_YEAR = " + dt.getMonthValue());
        System.out.println("DAY_OF_MONTH = " + dt.getDayOfMonth());
        System.out.println("HOUR_OF_DAY = " + dt.getHour());
        System.out.println("MINUTE_OF_HOUR = " + dt.getMinute());
        System.out.println("SECOND_OF_MINUTE = " + dt.getSecond());

        System.out.println("편의 메서드에 없음");
        System.out.println("MINUTE_OF_DAY = " + dt.get(ChronoField.MINUTE_OF_DAY));
        System.out.println("SECOND_OF_DAY = " + dt.get(ChronoField.SECOND_OF_DAY));
    }
}
```

**Output:**

```
YEAR = 2030
MONTH_OF_YEAR = 1
DAY_OF_MONTH = 1
HOUR_OF_DAY = 13
MINUTE_OF_HOUR = 30
SECOND_OF_MINUTE = 59

편의 메서드 사용
YEAR = 2030
MONTH_OF_YEAR = 1
DAY_OF_MONTH = 1
HOUR_OF_DAY = 13
MINUTE_OF_HOUR = 30
SECOND_OF_MINUTE = 59

편의 메서드에 없음
MINUTE_OF_DAY = 810
SECOND_OF_DAY = 48659
```

### 1.1 The core mechanism

> "날짜와 시간을 조회하려면 날짜와 시간 항목 중에 어떤 필드를 조회할지 선택해야 됩니다. 이때 날짜와
> 시간의 필드를 뜻하는 바로 ChronoField를 사용하면 됩니다."

- `dt.get(ChronoField.X)` is the one general-purpose way to pull **any** field out of **any**
  `TemporalAccessor` implementation.
- `get(...)` itself lives on the `TemporalAccessor` interface — `LocalDateTime` inherits it (and
  overrides it) from there, which is confirmed by jumping to the method definition.

> "TemporalAccessor는 특정 시점의 시간을 조회하는 기능을 제공해줘요... get에다가 나는 이 년, 월,
> 일, 시, 분, 초 중에 어떤 필드를 조회할 거라고 해서 넘기면... 템포럴 필드의 구현체인 크로노 필드를
> 인수로 여기다가 딱 전달하면 됩니다."

### 1.2 Convenience methods — same result, nicer syntax

- Writing `dt.get(ChronoField.YEAR)` every time is consistent, but verbose in practice. Each common
  field has a matching **convenience method**:
  - `ChronoField.YEAR` → `getYear()`
  - `ChronoField.MONTH_OF_YEAR` → `getMonthValue()` — **not** `getMonth()`, since that instead
    returns a `Month` enum object (e.g. `JANUARY`) rather than the raw integer `1`.
  - `ChronoField.DAY_OF_MONTH` → `getDayOfMonth()`
  - `ChronoField.HOUR_OF_DAY` → `getHour()`
  - `ChronoField.MINUTE_OF_HOUR` → `getMinute()`
  - `ChronoField.SECOND_OF_MINUTE` → `getSecond()`
- IntelliJ itself nudges toward these — suggesting the shorter convenience call in place of the
  verbose `get(ChronoField...)` form, since it's both easier to write and more readable.

> "자주 사용하는 메서드는 그래서 인텔리제이에서도 가독성도 훨씬 좋아요. 이 편의 메서드가 그래서
> 인텔리제이에서도 바꾸라고 아까 추천해 줬던 거고요."

### 1.3 When there's no convenience method

> "편의 메서드에 없는 경우도 있어요... 자주 사용하지 않는 특별한 기능은 편의 메서드를 제공하지
> 않아요."

- `MINUTE_OF_DAY` (minutes elapsed so far within the current day) and `SECOND_OF_DAY` (same idea, in
  seconds) have **no** dedicated convenience method — there's no `getMinuteOfDay()` on
  `LocalDateTime`.
- For these, falling back to `dt.get(ChronoField.MINUTE_OF_DAY)` directly is the only option — and
  it works exactly the same way as any other field lookup.
- Result for `13:30:59`: `MINUTE_OF_DAY = 810` (13×60 + 30 = 810 minutes since midnight),
  `SECOND_OF_DAY = 48659` (810×60 + 59 seconds).
- General guidance: use the convenience method when one exists (better readability), and fall back
  to `get(ChronoField...)` directly for anything less common that doesn't have one.

---

## 2. Manipulating with `Temporal.plus(long amountToAdd, TemporalUnit unit)`

```java
package time;

import java.time.LocalDateTime;
import java.time.Period;
import java.time.temporal.ChronoUnit;

public class ChangeTimePlusMain {

    public static void main(String[] args) {
        LocalDateTime dt = LocalDateTime.of(2018, 1, 1, 13, 30, 59);
        System.out.println("dt = " + dt);

        LocalDateTime plusDt1 = dt.plus(10, ChronoUnit.YEARS);
        System.out.println("plusDt1 = " + plusDt1);

        LocalDateTime plusDt2 = dt.plusYears(10);
        System.out.println("plusDt2 = " + plusDt2);

        Period period = Period.ofYears(10);
        LocalDateTime plusDt3 = dt.plus(period);
        System.out.println("plusDt3 = " + plusDt3);
    }
}
```

**Output:**

```
dt = 2018-01-01T13:30:59
plusDt1 = 2028-01-01T13:30:59
plusDt2 = 2028-01-01T13:30:59
plusDt3 = 2028-01-01T13:30:59
```

All three produce the **exact same result** — three different ways of expressing "add 10 years":

### 2.1 `plus(long amountToAdd, TemporalUnit unit)` — the general form

> "날짜와 시간을 조작하려면 어떤 시간 단위를 변경할지 선택해야 돼요. 이때 날짜와 시간의 단위를 뜻하는
> 바로 ChronoUnit이 이때 사용이 됩니다."

- `plus(...)` lives on the `Temporal` interface (the sub-interface of `TemporalAccessor` covered
  in Lecture 61 that adds manipulation on top of reading).
- Takes two arguments: how much to add (`10`), and which unit that amount is measured in
  (`ChronoUnit.YEARS`).
- Since it's immutable, the return value must be captured — same rule as every date/time class.
- A matching `minus(...)` also exists, following the same shape.

### 2.2 `plusYears(10)` — the convenience form

- Same mechanism as the `get` convenience methods: frequently-used units get their own
  `plusXxx(...)` method so callers don't have to spell out `ChronoUnit.YEARS` by hand.
- Only the commonly-used units get a dedicated convenience method — less common ones (just like
  `MINUTE_OF_DAY` above) would need the general `plus(amount, unit)` form instead.

### 2.3 `plus(Period period)` — using an interval directly

> "Period나 Duration은 시간의 간격을 뜻하기 때문에 특정 시점의 시간에 Duration이나 Period를 넣으면
> 그만큼 더 더해지겠죠."

- Since `Period` (and `Duration`) are `TemporalAmount` implementations (Lecture 61), they can be
  passed directly into `plus(...)` as a ready-made span — `Period.ofYears(10)` here plays exactly the
  same role as `10` + `ChronoUnit.YEARS`.
- This is the same pattern already seen using `Period`/`Duration` with `LocalDate`/`LocalTime` back
  in Lecture 60 — now shown explicitly side-by-side with the other two equivalent approaches.

---

## 3. The payoff: one consistent API across every implementation

> "덕분에 LocalDateTime, LocalDate, LocalTime, ZonedDateTime, Instant와 같은 수많은 구현에 관계없이
> 일관성 있는 방법으로 시간을 조회하고 조작할 수 있습니다... 설계가 진짜 잘 된 거죠."

- Because `TemporalAccessor.get(field)` and `Temporal.plus(amount, unit)` are defined once at the
  interface level, **every** implementation — `LocalDateTime`, `LocalDate`, `LocalTime`,
  `ZonedDateTime`, `Instant`, and more — supports the exact same calling pattern, regardless of its
  internal structure.
- Convenience methods (`getYear()`, `plusYears()`, etc.) exist purely as ergonomic shortcuts on top
  of this shared foundation — not a different mechanism.

---

## 4. Not every field is supported everywhere: `isSupported(...)`

```java
package time;

import java.time.LocalDate;
import java.time.temporal.ChronoField;

public class IsSupportedMain1 {

    public static void main(String[] args) {
        LocalDate now = LocalDate.now();
        int minute = now.get(ChronoField.SECOND_OF_MINUTE);
        System.out.println("minute = " + minute);
    }
}
```

**Output:**

```
Exception in thread "main"
java.time.temporal.UnsupportedTemporalTypeException: Unsupported field: SecondOfMinute
```

> "LocalDate는 분의 초 이런 필드를 지원하지 않기 때문에 요거를 조회할 수 없습니다."

- `LocalDate` only carries year/month/day — it has **no** notion of seconds at all. Asking for
  `ChronoField.SECOND_OF_MINUTE` on it throws `UnsupportedTemporalTypeException` at runtime (not
  caught at compile time).

### 4.1 Checking support first with `isSupported(...)`

```java
package time;

import java.time.LocalDate;
import java.time.temporal.ChronoField;

public class IsSupportedMain2 {

    public static void main(String[] args) {
        LocalDate now = LocalDate.now();
        boolean supported = now.isSupported(ChronoField.SECOND_OF_MINUTE);
        System.out.println("supported = " + supported);
        if (supported) {
            int minute = now.get(ChronoField.SECOND_OF_MINUTE);
            System.out.println("minute = " + minute);
        }
    }
}
```

**Output:**

```
supported = false
```

> "이런 문제를 예방하기 위해서 TemporalAccessor와 Temporal 인터페이스는 현재 타입에서 특정 시간
> 단위나 필드를 사용할 수 있는지 없는지 확인할 수 있는 메서드를 제공해줘요."

- Both core interfaces expose an `isSupported(...)` check:
  - `TemporalAccessor`: `boolean isSupported(TemporalField field)`
  - `Temporal`: `boolean isSupported(TemporalUnit unit)`
- Each implementation defines for itself which fields/units it actually supports — `LocalDate`
  correctly reports `false` for `SECOND_OF_MINUTE`, since it genuinely has no time-of-day data at
  all.
- Guarding a `get(...)` call behind `isSupported(...)` avoids the exception entirely — only attempt
  the lookup/manipulation if the implementation confirms it's valid.

---

## Summary

- Querying any point-in-time value uses `TemporalAccessor.get(TemporalField field)` — pass a
  `ChronoField` constant to pull out exactly the field you want, consistently across every
  implementation.
- Manipulating uses `Temporal.plus(long amountToAdd, TemporalUnit unit)` (and `minus(...)`) — pass a
  `ChronoUnit` constant alongside the amount; also immutable, so the result must be captured.
- Both have **convenience methods** (`getYear()`, `plusYears()`, etc.) for commonly-used
  fields/units — more readable, and what IDEs will suggest — but **not every** field/unit has one;
  fall back to the general `get(field)` / `plus(amount, unit)` form for anything uncommon (e.g.
  `MINUTE_OF_DAY`, `SECOND_OF_DAY`).
- `Period`/`Duration` can also be passed directly into `plus(...)` since they implement
  `TemporalAmount` — functionally identical to `plus(amount, ChronoUnit)`.
- The real payoff: because these methods are defined once on the shared interfaces, **every**
  `java.time` type — `LocalDateTime`, `LocalDate`, `LocalTime`, `ZonedDateTime`, `Instant`, and
  more — supports querying and manipulation in exactly the same consistent way, regardless of its
  internal implementation.
- Not every field/unit is valid for every type — e.g. `LocalDate` has no time component, so
  `get(ChronoField.SECOND_OF_MINUTE)` throws `UnsupportedTemporalTypeException`. Use
  `isSupported(field)` / `isSupported(unit)` beforehand to check safely rather than catching the
  exception.
- Next lecture: date/time manipulation continues with the `with(...)` method.
