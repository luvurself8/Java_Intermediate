# Lecture 64 — Date and Time: Parsing and Formatting Date/Time Strings

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (10/13)

Previous: [Lecture 63 - Querying and Manipulating Date and Time 2](./Lecture%2063%20-%20Date%20and%20Time%20-%20Querying%20and%20Manipulating%20Date%20and%20Time%202.md)

A very commonly needed, practical pair of operations: converting a date/time object into a
human-readable string (**formatting**), and reading a string back into a date/time object
(**parsing**) — both built around `DateTimeFormatter`.

> "포맷팅은 날짜와 시간 데이터를 원하는 포맷의 문자열로 바꾸는 걸 말해요... 파싱은 문자열을 날짜와
> 시간 데이터로 변경하는 거예요."

| | Direction | Example |
|---|---|---|
| **Formatting** | `Date` → `String` | `LocalDate` → `"2024년 12월 31일"` |
| **Parsing** | `String` → `Date` | `"2030년 01월 01일"` → `LocalDate` |

---

## 1. Formatting and parsing a date: `FormattingMain1`

```java
package time;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class FormattingMain1 {

    public static void main(String[] args) {
        // 포맷팅: 날짜를 문자로
        LocalDate date = LocalDate.of(2024, 12, 31);
        System.out.println("date = " + date);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy년 MM월 dd일");
        String formattedDate = date.format(formatter);
        System.out.println("날짜와 시간 포맷팅 = " + formattedDate);

        // 파싱: 문자를 날짜로
        String input = "2030년 01월 01일";
        LocalDate parsedDate = LocalDate.parse(input, formatter);
        System.out.println("문자열 파싱 날짜와 시간: " + parsedDate);
    }
}
```

**Output:**

```
date = 2024-12-31
날짜와 시간 포맷팅 = 2024년 12월 31일
문자열 파싱 날짜와 시간: 2030-01-01
```

### 1.1 Why not just print the object directly?

> "그냥 출력하면 이런 형태로 나오거든요. 이게 약간 ISO 표준의 출력이라고 해요."

- Printing a `LocalDate` directly (`2024-12-31`) uses its default `toString()`, which follows the
  **ISO 8601** international standard format — not necessarily the format an application or user
  actually wants (e.g. `2024년 12월 31일`).
- It would be *possible* to manually build a custom string using `date.getYear()`,
  `date.getMonthValue()`, etc., concatenated by hand — but `DateTimeFormatter` is the dedicated,
  far more convenient tool for this.

### 1.2 Formatting: `DateTimeFormatter.ofPattern(...)` + `date.format(formatter)`

- `DateTimeFormatter.ofPattern("yyyy년 MM월 dd일")` (from `java.time.format`) builds a formatter
  object describing exactly how the output string should look — `yyyy` for the 4-digit year, `MM`
  for the 2-digit month, `dd` for the 2-digit day, with the literal Korean characters `년`/`월`/`일`
  mixed in directly.
- `date.format(formatter)` then applies that pattern, producing `"2024년 12월 31일"`.

### 1.3 Parsing: `LocalDate.parse(input, formatter)`

> "문자를 날짜로 변경하는 게 파싱입니다... 이 포맷터는 지금 이 모양이랑 이 모양이랑 같죠? 이 모양이
> 같아야 돼요."

- Going the other direction: `LocalDate.parse("2030년 01월 01일", formatter)` reads the string using
  the **same pattern** used for formatting, correctly extracting the year/month/day from their
  expected positions.
- The formatter's pattern must match the actual shape of the input string — if the pattern expects
  `yyyy년 MM월 dd일` but the string doesn't follow that exact shape, parsing fails.

### 1.4 Pattern letters: case matters

> "여기서 이제 M의 소문자는 분이에요. 그래서 M의 대문자가 이게 월을 뜻하거든요. 그래서 대문자로
> 쓰셔야 됩니다. 소문자 쓰시면 분이 돼요."

- `DateTimeFormatter` pattern letters are **case-sensitive**, and easy to mix up: uppercase `M` =
  month, lowercase `m` = minute. Getting this wrong silently produces a formatter that reads/writes
  the wrong field.
- The official pattern reference (linked in the slide) is the authoritative source to check any
  specific letter:

| Symbol | Meaning | Example |
|---|---|---|
| `yyyy` / `u` | Year | `2024` |
| `MM` / `M` | Month (uppercase!) | `12`, `Dec` |
| `dd` | Day of month | `31` |
| `HH` | Hour, 24-hour format | `13` |
| `hh` | Hour, 12-hour format | `01` |
| `mm` | Minute (lowercase!) | `30` |
| `ss` | Second | `59` |
| `a` | AM/PM marker | `PM` |
| `E` | Day of week | `Tue`, `Tuesday` |
| `z` / `V` / `O` / `X` | Time zone name / ID / offset variants | `PST`, `Asia/Seoul`, `+09:00` |

(Full reference: [Oracle's `DateTimeFormatter` pattern docs](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#patterns))

---

## 2. Formatting and parsing a date **and** time: `FormattingMain2`

```java
package time;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class FormattingMain2 {

    public static void main(String[] args) {
        // 포맷팅: 날짜와 시간을 문자로
        LocalDateTime now = LocalDateTime.of(2024, 12, 31, 13, 30, 59);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String formattedDateTime = now.format(formatter);
        System.out.println("날짜와 시간 포맷팅: " + formattedDateTime);

        // 파싱: 문자를 날짜와 시간으로
        String dateTimeString = "2030-01-01 11:30:00";
        LocalDateTime parsedDateTime = LocalDateTime.parse(dateTimeString, formatter);
        System.out.println("문자열 파싱 날짜와 시간: " + parsedDateTime);
    }
}
```

**Output:**

```
날짜와 시간 포맷팅: 2024-12-31 13:30:59
문자열 파싱 날짜와 시간: 2030-01-01T11:30
```

### 2.1 A very commonly used pattern

- `"yyyy-MM-dd HH:mm:ss"` is called out explicitly as an extremely common real-world format —
  combining date and time with a 24-hour clock (`HH`, uppercase) and zero-padded minute/second
  (`mm`, `ss`, lowercase).
- The mechanics are identical to the date-only example: `format(formatter)` to produce a string,
  `LocalDateTime.parse(string, formatter)` to read one back — every `java.time` type that supports
  formatting has a matching `parse(...)` method taking the same formatter.

### 2.2 Why the parsed output shows `11:30` instead of `11:30:00`

> "이거는 초가 없어가지고 생략을 해준 것 같아요... 초과 00이니까 여기서는 표준 출력에서는 생략이
> 됐네요."

- Printing `now` directly (not shown via the final `format(...)` call, but demonstrated live) shows
  Java's default `toString()` for `LocalDateTime`, which again follows **ISO 8601** — separating
  date and time with a literal `T` (e.g. `2024-12-31T13:30:59`).
- When seconds are exactly `:00`, the default `toString()` output omits them entirely — which is
  exactly why `parsedDateTime` prints as `2030-01-01T11:30` rather than `2030-01-01T11:30:00`: the
  **value** is still `11:30:00` internally, the default string representation just doesn't show a
  trailing zero-second component.

---

## Summary

- **Formatting** converts a date/time object into a string (`Date` → `String`); **parsing** does the
  reverse (`String` → `Date`) — both are built around `DateTimeFormatter`
  (`java.time.format.DateTimeFormatter`).
- `DateTimeFormatter.ofPattern("...")` builds a formatter from a pattern string; `date.format(formatter)`
  applies it to produce a string; `SomeType.parse(string, formatter)` reads a string back using the
  same pattern.
- The pattern used for formatting and the one used for parsing a given string must describe the
  **same shape** — if they don't match, parsing fails.
- Pattern letters are case-sensitive and easy to confuse: uppercase `M` = month vs. lowercase `m` =
  minute; uppercase `HH` = 24-hour vs. lowercase `hh` = 12-hour. Check the official pattern reference
  when in doubt.
- `"yyyy-MM-dd HH:mm:ss"` is called out as an especially common real-world format worth
  remembering.
- Printing a date/time object directly (without a custom formatter) uses the **ISO 8601** standard
  via its default `toString()` — which also quietly omits a trailing `:00` seconds component when
  present, even though the underlying value still has it.
- Next lecture: practice problems and solutions for everything covered across Section 7.
