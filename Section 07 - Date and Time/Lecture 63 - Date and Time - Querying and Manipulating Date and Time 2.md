# Lecture 63 — Date and Time: Querying and Manipulating Date and Time (2)

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (9/13)

Previous: [Lecture 62 - Querying and Manipulating Date and Time 1](./Lecture%2062%20-%20Date%20and%20Time%20-%20Querying%20and%20Manipulating%20Date%20and%20Time%201.md)

Lecture 62 covered `get` (read) and `plus`/`minus` (add/subtract). This lecture adds the third
manipulation method — `with(...)` — for directly **replacing** one field's value, plus
`TemporalAdjusters` for complex date calculations that would otherwise be painful to write by hand.

---

## 1. `with(TemporalField field, long newValue)` — replacing one field

```java
package time;

import java.time.DayOfWeek;
import java.time.LocalDateTime;
import java.time.temporal.ChronoField;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class ChangeTimeWithMain {

    public static void main(String[] args) {
        LocalDateTime dt = LocalDateTime.of(2018, 1, 1, 13, 30, 59);
        System.out.println("dt = " + dt);

        LocalDateTime changedDt1 = dt.with(ChronoField.YEAR, 2020);
        System.out.println("changedDt1 = " + changedDt1);

        LocalDateTime changedDt2 = dt.withYear(2020);
        System.out.println("changedDt2 = " + changedDt2);

        //TemporalAdjuster 사용
        //다음주 금요일
        LocalDateTime with1 = dt.with(TemporalAdjusters.next(DayOfWeek.FRIDAY));
        System.out.println("기준 날짜: " + dt);
        System.out.println("다음 금요일: " + with1);

        //이번 달의 마지막 일요일
        LocalDateTime with2 = dt.with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
        System.out.println("같은 달의 마지막 일요일 = " + with2);
    }
}
```

**Output:**

```
dt = 2018-01-01T13:30:59
changedDt1 = 2020-01-01T13:30:59
changedDt2 = 2020-01-01T13:30:59
기준 날짜: 2018-01-01T13:30:59
다음 금요일: 2018-01-05T13:30:59
같은 달의 마지막 일요일 = 2018-01-28T13:30:59
```

### 1.1 The core idea

> "with는 뭔가 바꾸는 거죠... 커피 with 설탕. 기존의 거에다가 뭔가 살짝 하나를 바꿔서 새로 넘어온
> 얘를 바꾸는 거예요."

- Recall from earlier in the course: "with" has been used throughout for "take the existing thing,
  swap in one new piece, and produce a new result" (the same spirit as "coffee *with* sugar").
- `dt.with(ChronoField.YEAR, 2020)` keeps every other field (month, day, hour, minute, second)
  exactly as-is, and replaces **only** the year — producing a brand-new `LocalDateTime` with `2018`
  swapped for `2020`.
- Same immutability rule applies: the return value must be captured.

### 1.2 Convenience method: `withYear(2020)`

- Just like `getYear()` / `plusYears()`, a `withYear(int)` convenience method exists for commonly
  replaced fields — same result as `with(ChronoField.YEAR, 2020)`, less verbose.
- General guidance, same as before: prefer the convenience method when one exists.

---

## 2. Beyond simple fields: `TemporalAdjusters` for complex date math

> "복잡하게 날짜를 수정할 때가 있어요. 다음 주 금요일. 이거 어떻게 구하지? 이거 계산하려면 복잡하게
> 계산해야 되거든요."

- Plain `with(field, value)` only swaps a single, simple field — it can't directly answer something
  like "what date is next Friday?" or "what's the last Sunday of this month?" Computing those by
  hand would mean manually walking the calendar day by day.

### 2.1 The `TemporalAdjuster` interface — and why you don't need to implement it yourself

```java
public interface TemporalAdjuster {
    Temporal adjustInto(Temporal temporal);
}
```

> "원래대로 하면 이 인터페이스를 직접 구현해야 되지만, 자바는 이미 필요한 구현체들을
> TemporalAdjusters에 다 만들어뒀습니다. 우리는 단순히 이 구현체들을 모아둔 TemporalAdjusters를
> 사용하면 돼요."

- `with(...)` has an overload that accepts a `TemporalAdjuster` — an interface with one method,
  `adjustInto(Temporal)`, that describes an arbitrary date adjustment rule.
- In principle, anyone could implement this interface themselves for a custom rule — but Java
  already ships a large collection of common, ready-made implementations bundled inside the
  **`TemporalAdjusters`** (plural) utility class, so there's rarely a need to write one from scratch.

### 2.2 Worked examples

- **Next Friday**: `dt.with(TemporalAdjusters.next(DayOfWeek.FRIDAY))`
  - `2018-01-01` (a Monday) → `2018-01-05` — the next upcoming Friday after the base date.
- **Last Sunday of the same month**: `dt.with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY))`
  - `2018-01-01` → `2018-01-28` — the final Sunday within January 2018.

### 2.3 `TemporalAdjusters` — commonly available methods

| Method | What it computes |
|---|---|
| `dayOfWeekInMonth` | Adjusts based on which occurrence (1st, 2nd, ...) of a given weekday falls in the month |
| `firstDayOfMonth` | First day of the current month |
| `firstDayOfNextMonth` | First day of next month |
| `firstDayOfNextYear` | First day of next year |
| `firstDayOfYear` | First day of the current year |
| `firstInMonth` | First occurrence of a given weekday within the current month |
| `lastDayOfMonth` | Last day of the current month |
| `lastDayOfNextMonth` | Last day of next month |
| `lastDayOfNextYear` | Last day of next year |
| `lastDayOfYear` | Last day of the current year |
| `lastInMonth` | Last occurrence of a given weekday within the current month |
| `next` | Nearest occurrence of a given weekday **after** the current date |
| `nextOrSame` | Same as `next`, but returns the current date itself if it already matches |
| `previous` | Nearest occurrence of a given weekday **before** the current date |
| `previousOrSame` | Same as `previous`, but returns the current date itself if it already matches |

> "여러분 날짜 계산하고 고생하면 안되고 이런거 갖다 쓰셔야 됩니다."

- Practical guidance: don't try to memorize this whole list now — just know it exists, skim it once
  to build awareness, and come back to look up the specific adjuster needed whenever a complex
  date-calculation requirement actually comes up in real work.

---

## Summary

- `Temporal.with(TemporalField field, long newValue)` replaces exactly one field's value while
  leaving everything else untouched, returning a new instance (immutable, so the result must be
  captured) — e.g. `dt.with(ChronoField.YEAR, 2020)`, or its convenience form `dt.withYear(2020)`.
- Between the three manipulation primitives covered across Lectures 62–63:
  - `get(field)` — read a value.
  - `plus(amount, unit)` / `minus(...)` — add/subtract by an amount and unit.
  - `with(field, value)` — directly replace one field's value.
- `with(...)` also has an overload accepting a `TemporalAdjuster` — a one-method interface
  (`adjustInto(Temporal)`) representing an arbitrary date-adjustment rule.
- Rather than implementing `TemporalAdjuster` by hand, use Java's built-in implementations bundled
  in the **`TemporalAdjusters`** utility class — `next(DayOfWeek)`, `lastInMonth(DayOfWeek)`,
  `firstDayOfMonth()`, `firstDayOfNextYear()`, and many more cover the vast majority of
  real-world "find this complicated date" needs without any manual calendar math.
- Next lecture: parsing and formatting date/time values as strings.
