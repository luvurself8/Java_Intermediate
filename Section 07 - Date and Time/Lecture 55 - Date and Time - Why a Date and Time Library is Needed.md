# Lecture 55 — Date and Time: Why a Date and Time Library is Needed

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (1/13)

This is the first lecture of Section 7. It has no code at all — the whole point is to build intuition
for *why* date/time handling is deceptively hard, so that reaching for a well-designed library (rather
than hand-rolling date math) feels obviously correct by the end.

> "날짜와 시간을 계산하는 것은 단순하게 생각해보면 되게 쉬울 것 같죠? 근데 실제로는 매우 복잡하고
> 어렵습니다."

---

## 1. Calculating the difference between two dates

- Simple subtraction doesn't work for dates, because every month has a different number of days.
- Example: how many days are between **2024-01-01** and **2024-02-01**?
  - You have to know January has 31 days to get the right answer — it's not just "subtract the day
    numbers."
- Scale that up to arbitrary date ranges across many months, and the bookkeeping gets complicated
  fast.

---

## 2. Leap years

- A calendar year is defined as 365 days, but Earth actually takes about **365.2425 days** (365 days,
  5 hours, 48 minutes, 45 seconds) to orbit the sun.
- That mismatch accumulates over time, so every 4 years a leap day is inserted (February 29th) to
  keep the calendar roughly aligned with the solar year.
- But the leap year rule itself is **not** just "every 4 years" — it's more nuanced:
  - Divisible by 4 → leap year.
  - **Except** divisible by 100 → **not** a leap year.
  - **Except** divisible by 400 → leap year again.
- Worked examples from the slide: **2000 and 2020 are leap years**, but **1900 and 2100 are not**.
- So answering "how many days between 2024-01-01 and 2024-03-01?" requires knowing 2024 is a leap
  year (so February has 29 days that year) — another small rule you have to get exactly right.

> "여러분, 이 달력이라는 게 쉽지 않습니다."

---

## 3. Daylight Saving Time (DST)

- Also called 서머타임 ("summer time"). Common in many countries, but **not observed in South Korea
  since 1988**.
- The idea: during roughly March–October, the sun rises earlier, so clocks are shifted forward (or
  back) by an hour to better align working hours with daylight.
- The exact rule varies **by country and region** — start date, end date, and whether it's observed
  at all are all different depending on where you are.
- Example from the slide: in some regions, DST starts the last Sunday of March and ends the last
  Sunday of October. Any date/time calculation that spans this window has to add or subtract an hour
  correctly, or the result is off by an hour.
- **Personal anecdote from the instructor**: working with developers in Berlin, meeting times kept
  shifting by an hour without an obvious reason — it turned out to be DST changing Berlin's offset
  from UTC+1 to UTC+2, which shifted the usable overlap between Korean and German working hours (made
  scheduling meetings slightly easier during DST, since the time gap shrank from 8 hours to 7).

---

## 4. Time zone calculations, GMT, and UTC

- The world is divided into many time zones, each defined as an **offset from UTC**.
- Example time zones from the slide: `Europe/London`, `GMT`, `UTC`, `US/Arizona (-07:00)`,
  `America/New_York (-05:00)`, `Asia/Seoul (+09:00)`, `Asia/Dubai (+04:00)`, `Asia/Istanbul (+03:00)`,
  `Asia/Shanghai (+08:00)`, `Europe/Paris (+01:00)`, `Europe/Berlin (+01:00)`.

### GMT vs UTC

- **London / UTC / GMT all point to the same `00:00` reference time zone** — both are anchored to
  the Greenwich Observatory in London at longitude 0°.
- **GMT (Greenwich Mean Time)**: the original standard — when the sun crosses the Greenwich
  Observatory, that moment is defined as noon. This was historically the world's time standard.
- **UTC (Coordinated Universal Time)**: introduced later to replace GMT as the international
  standard. It's measured using atomic clocks, and since Earth's rotation speed actually varies
  slightly over time, UTC is adjusted with **leap seconds** to stay accurate.
- In everyday use, GMT and UTC are close enough to be treated as the same thing — but for scientific
  and precise international standards, **UTC is preferred**. The UK itself still colloquially uses
  "GMT," but under the hood today's systems run on UTC.

> "우리는 UTC를 쓰면 돼요." — for all practical purposes, just use UTC as the reference.

### Worked example: scheduling a meeting across time zones

- **Scenario**: someone in Seoul wants to schedule a meeting with someone in Berlin.
  - Seoul's time zone: `Asia/Seoul`, UTC+9.
  - Berlin's time zone: `Europe/Berlin`, UTC+1.
- **Time zone difference**: 9 − 1 = **8 hours** — Seoul is 8 hours ahead of Berlin.
- **Calculation**: if the meeting is at **9:00 PM in Seoul**, then Berlin's local time is
  `9:00 PM − 8 hours = 1:00 PM`.
- **But that's not the end of it** — DST complicates this further:
  - If Berlin is observing DST, its offset shifts from UTC+1 to **UTC+2**.
  - That changes the Seoul–Berlin gap from 8 hours down to **7 hours**.
  - Berlin's DST window (per the slide): last Sunday of March through the last Sunday of October.
  - So the *same* meeting-time calculation gives a different answer depending on the time of year —
    the offset isn't a fixed constant, it's a rule that depends on the calendar date.

---

## 5. The takeaway: don't calculate this yourself

> "내가 직접 여러분 날짜를 계산하면 99.9%의 확률로 문제가 발생합니다."

- You are not expected to memorize or master leap-year rules, DST windows, or time zone tables.
- The one thing to internalize: **manually computing date/time math yourself is nearly guaranteed to
  go wrong somewhere** — there are too many edge cases (leap years, leap seconds, DST transitions,
  region-specific rules) for hand-rolled logic to reliably get right.
- This is exactly why essentially every modern programming language ships a dedicated date/time
  library that abstracts all of this complexity away, so application code can stay simple, correct,
  and efficient.

---

## 6. A brief history of Java's date/time libraries

Java has iterated on this problem repeatedly over the years:

### JDK 1.0 — `java.util.Date`

- **Problems:**
  - No real time zone support — couldn't handle global/multi-region time correctly.
  - Very limited date arithmetic — hard to add/subtract dates or compute differences.
  - **Not immutable** — `Date` objects could be mutated after creation, which caused bugs from
    unexpected side effects.

### JDK 1.1 — `java.util.Calendar`

- Introduced specifically to fix `Date`'s problems: improved time zone support, and more operations
  for date/time arithmetic.
- **Still problematic:**
  - Complicated and unintuitive to use.
  - Performance issues in some use cases.
  - Still **not immutable** — mutable `Calendar` objects brought the same side-effect and thread-safety
    concerns as `Date`.
- The instructor notes this is explained *only* because legacy codebases may still contain it — not
  because it's something to reach for today.

### Joda-Time (open source)

- `Date` and `Calendar` were frustrating enough that an outside developer eventually built an
  independent open-source library, **Joda-Time**, to solve the usability, performance, and
  immutability problems properly.
- It became hugely popular because it was genuinely pleasant to use — but it was never part of the
  Java standard library, so every project that wanted it had to pull it in as an external dependency.

### Java 8 — `java.time` (JSR-310)

- Java's official response: bring this functionality into the standard library properly.
- Rather than just reimplementing the idea independently, **Java brought in the original author of
  Joda-Time** to help design the new standard — JSR-310, the `java.time` package.
- Result: a standard API that fixes all the earlier problems — good usability, good performance,
  thread safety, proper time zone handling, and full immutability (every `java.time` type is an
  immutable value object, eliminating the side-effect and thread-safety issues of `Date`/`Calendar`).
- Core classes introduced: `LocalDate`, `LocalTime`, `LocalDateTime`, `ZonedDateTime`, `Instant` (all
  covered in upcoming lectures).
- In effect, `java.time` brought most of Joda-Time's design directly into the core platform —
  though the original Joda-Time author noted a few things he'd actually want to change given a
  second chance, so `java.time` isn't a pure copy; it's a refined version shaped by community
  feedback too.

> "실용적인 Joda-Time에 많은 자바 커뮤니티의 의견을 반영해서 좀 더 안정적이고 표준적인 날짜와 시간
> 라이브러리인 java.time 패키지가 성공적으로 완성되었다."

### Aside: the same pattern happened with ORM → JPA

- Java's original standard ORM technology was similarly unpleasant to use.
- An external developer built **Hibernate** as an open-source alternative, which became more popular
  than the official standard.
- Java again brought in Hibernate's author to help design a new official standard: **JPA**.
- Both `java.time` and JPA are now considered Java's fully-adopted, mainstream standard technologies.
- The instructor's personal reflection: this pattern — adopting real-world, battle-tested, popular
  tools into the official standard rather than designing everything top-down — is part of why Java
  has stayed relevant for so long, and why its more recent update cadence feels encouraging.

> "실용적인 거를 가져와서 딱 뭔가 표준으로 잘 다듬어서 만드는 거. 이게 진짜 좋은 사례들인 것 같아요."

---

## Summary

- Date/time math looks trivial but is genuinely hard, due to several independent sources of
  complexity:
  - **Date differences** require knowing variable month lengths.
  - **Leap years** follow a non-obvious rule (÷4 yes, ÷100 no, ÷400 yes again) — e.g. 2000/2020 are
    leap years, 1900/2100 are not.
  - **DST** shifts clocks by an hour during part of the year, with start/end dates that vary by
    country/region (Korea has not observed it since 1988).
  - **Time zones** are defined as offsets from UTC, and those offsets themselves can shift when DST
    applies (e.g. Berlin: UTC+1 → UTC+2).
- **GMT vs UTC**: both anchored to Greenwich; GMT is the historical standard, UTC is the modern
  atomic-clock-based standard (adjusted with leap seconds) — in practice, just use UTC.
- Worked example: Seoul (UTC+9) is normally 8 hours ahead of Berlin (UTC+1); scheduling 9 PM Seoul =
  1 PM Berlin — but that 8-hour gap shrinks to 7 hours whenever Berlin DST is active.
- Conclusion: manual date/time calculation is essentially guaranteed to introduce bugs — always use
  a proper library.
- Java's date/time library history: `java.util.Date` (JDK 1.0, no timezones, mutable) →
  `java.util.Calendar` (JDK 1.1, still clunky and mutable) → **Joda-Time** (popular open-source fix,
  but not a standard library) → **`java.time` / JSR-310** (Java 8, built together with Joda-Time's
  original author, fully immutable, thread-safe, the current standard).
- Same adopt-the-popular-open-source-tool pattern happened with ORM: Hibernate → JPA.
- Next lecture: an introduction to Java's `java.time` package itself (`LocalDate`, `LocalTime`,
  `LocalDateTime`, `ZonedDateTime`, `Instant`, etc.).
