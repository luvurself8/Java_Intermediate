# Lecture 57 — Date and Time: LocalDateTime

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 7. 날짜와 시간 (3/13)

Previous: [Lecture 56 - Introducing Java's Date and Time Library](./Lecture%2056%20-%20Date%20and%20Time%20-%20Introducing%20Java's%20Date%20and%20Time%20Library.md)

First hands-on lecture of the section — real code for `LocalDate`, `LocalTime`, and `LocalDateTime`,
the three classes the instructor says cover roughly 95% of real-world Korean/domestic development
work.

> "대부분의 개발자분들은... 대부분은 거의 국내 애플리케이션을 개발해요. 글로벌 서비스를 하지 않는
> 이상은 그냥 이 로컬데이트를 쓰신다고 보면 됩니다... 제가 볼 때는 거의 95%는 로컬데이트타임을 쓸
> 가능성이 높습니다."

- These three are deliberately covered first and covered well, since the "global" classes
  (`ZonedDateTime`, etc., starting next lecture) are really just this same API with a time zone
  bolted on — understanding this part solidly makes everything after it easier.

---

## 1. `LocalDate` — date only

```java
package time;

import java.time.LocalDate;

public class LocalDateMain {

    public static void main(String[] args) {
        LocalDate nowDate = LocalDate.now();
        LocalDate ofDate = LocalDate.of(2013, 11, 21);
        System.out.println("오늘 날짜=" + nowDate);
        System.out.println("지정 날짜=" + ofDate);

        //계산(불변)
        ofDate = ofDate.plusDays(10);
        System.out.println("지정 날짜+10d = " + ofDate);
    }
}
```

**Output:**

```
오늘 날짜 = 2024-02-09
지정 날짜 = 2013-11-21
지정 날짜+10d = 2013-12-01
```

- **`now()`**: creates a `LocalDate` for the current date.
- **`of(year, month, day)`**: creates a `LocalDate` for any specific date you want.
- **`plusDays(10)`**: adds 10 days — but only returns the *result*, it doesn't mutate `ofDate`.

### The immutability trap, demonstrated live

> "자, 여기서 내가 10일을 더 하고 싶어요... 자, 일단 제가 그냥 한번 출력해 볼게요. 불변인지 아닌지
> 지정 날짜, plusDays(10) 해볼게요. 지금 아무 일도 안 벌어졌죠? ... 왜냐면 불변이기 때문에. 반환값
> 받아야 된다."

- The instructor first calls `ofDate.plusDays(10)` **without** assigning the result, then prints
  `ofDate` again — and nothing changed. `ofDate` is still the original date.
- The fix: `ofDate = ofDate.plusDays(10);` — capturing the **returned**, new instance.
- This is the same immutability pattern seen repeatedly across the course (`String`, wrapper
  classes) — and the instructor calls this out explicitly:

> "제가 그렇게 불변을 계속 설명을 드렸던 이유가 지금 보세요. 공부했던 스트링도 불변이죠. 래퍼도 다
> 불변이죠. 그 다음에 지금 날짜까지 다 불변이란 말이에요. 그래서 이 불변에 대한 개념을 알아야 이런
> 거에 대해서도 제대로 딱 이해할 수가 있습니다."

- **Every date/time class in `java.time` is immutable.** Any "change" operation returns a brand-new
  object — the original is never touched, and the return value must always be captured or the
  change is silently lost.

---

## 2. `LocalTime` — time only

```java
package time;

import java.time.LocalTime;

public class LocalTimeMain {

    public static void main(String[] args) {
        LocalTime nowTime = LocalTime.now();
        LocalTime ofTime = LocalTime.of(9, 10, 30);

        System.out.println("현재 시간 = " + nowTime);
        System.out.println("지정 시간 = " + ofTime);

        //계산(불변)
        LocalTime ofTimePlus = ofTime.plusSeconds(30);
        System.out.println("지정 시간+30s = " + ofTimePlus);
    }
}
```

**Output:**

```
현재 시간 = 11:52:51.219602
지정 시간 = 09:10:30
지정 시간+30s = 09:11:00
```

- Same shape as `LocalDate`: `now()` for the current time, `of(hour, minute, second[, nano])` for a
  specific time.
- `plusSeconds(30)` on `09:10:30` correctly rolls over the minute to `09:11:00` — 30 more seconds
  past the 30-second mark crosses into the next minute automatically.
- Also immutable — the return value (`ofTimePlus`) had to be captured, same as `LocalDate`.
- The instructor notes this section is quick because it's really just applying the same API shape
  already learned for `LocalDate`:

> "이거는 그냥 만들어진 거에 대한 API 사용법을 배운 거기 때문에 그렇게 어렵진 않으실 거예요."

---

## 3. `LocalDateTime` — the composition of both

### The design: it's literally built out of the other two

```java
public class LocalDateTime {
    private final LocalDate date;
    private final LocalTime time;
    ...
}
```

> "LocalDateTime은 LocalDate와 LocalTime을 내부에 가지고 날짜와 시간을 모두 표현합니다. 이거 잘
> 설계가 됐어요... 뭔가 레고 블록을 조립하듯이 탁탁 조립돼 있잖아요. 얘는 연월일, 얘는 시분초. 그럼
> 두 개 합쳐서 쓰면 되겠네 해서 이렇게 딱 만든 거죠."

- `LocalDateTime` doesn't reimplement date/time logic from scratch — it holds one `LocalDate` field
  and one `LocalTime` field internally, and combines them.
- The instructor highlights this as an example of genuinely good class design: two already-solid,
  focused classes composed together like Lego blocks, rather than one monolithic class trying to do
  everything itself.

### Full example

```java
package time;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;

public class LocalDateTimeMain {

    public static void main(String[] args) {
        LocalDateTime nowDt = LocalDateTime.now();
        LocalDateTime ofDt = LocalDateTime.of(2016, 8, 16, 8, 10, 1);
        System.out.println("현재 날짜시간 = " + nowDt);
        System.out.println("지정 날짜시간 = " + ofDt);

        //날짜와 시간 분리
        LocalDate localDate = ofDt.toLocalDate();
        LocalTime localTime = ofDt.toLocalTime();
        System.out.println("localDate = " + localDate);
        System.out.println("localTime = " + localTime);

        //날짜와 시간 합체
        LocalDateTime localDateTime = LocalDateTime.of(localDate, localTime);
        System.out.println("localDateTime = " + localDateTime);

        //계산(불변)
        LocalDateTime ofDtPlus = ofDt.plusDays(1000);
        System.out.println("지정 날짜시간+1000d = "+ ofDtPlus);
        LocalDateTime ofDtPlus1Year = ofDt.plusYears(1);
        System.out.println("지정 날짜시간+1년 = " + ofDtPlus1Year);

        //비교
        System.out.println("현재 날짜시간이 지정 날짜시간보다 이전인가? " + nowDt.isBefore(ofDt));
        System.out.println("현재 날짜시간이 지정 날짜시간보다 이후인가? " + nowDt.isAfter(ofDt));
        System.out.println("현재 날짜시간과 지정 날짜시간이 같은가? " + nowDt.isEqual(ofDt));
    }
}
```

**Output:**

```
현재 날짜시간 = 2024-02-09T11:54:54.389163
지정 날짜시간 = 2016-08-16T08:10:01
localDate = 2016-08-16
localTime = 08:10:01
localDateTime = 2016-08-16T08:10:01
지정 날짜시간+1000d = 2019-05-13T08:10:01
지정 날짜시간+1년 = 2017-08-16T08:10:01
현재 날짜시간이 지정 날짜시간보다 이전인가? false
현재 날짜시간이 지정 날짜시간보다 이후인가? true
현재 날짜시간과 지정 날짜시간이 같은가? false
```

### 3.1 Splitting apart: `toLocalDate()` / `toLocalTime()`

- Since `LocalDateTime` holds a `LocalDate` and a `LocalTime` internally, pulling either one back
  out is trivial — `toLocalDate()` / `toLocalTime()` literally just return the already-stored field.

### 3.2 Putting back together: `LocalDateTime.of(localDate, localTime)`

- `of(...)` has an overload that accepts an existing `LocalDate` and `LocalTime` directly and
  combines them into a new `LocalDateTime` — no need to re-specify year/month/day/hour/minute/second
  individually if you already have both halves.
- The instructor calls this the same "Lego block" design paying off again — split apart, then
  snapped back together, with no information lost either way.

### 3.3 Calculating: `plusDays`, `plusYears`, and the rest

- `ofDt.plusDays(1000)` on `2016-08-16T08:10:01` → `2019-05-13T08:10:01`.
- `ofDt.plusYears(1)` on the same starting value → `2017-08-16T08:10:01`.
- Both return a **new** `LocalDateTime` — `ofDt` itself is never modified, consistent with every
  other class in this API.
- A wide range of `plusXxx(...)` methods exist beyond these two (days, years, months, hours,
  minutes, seconds, etc.) — same immutable pattern throughout.

### 3.4 Comparing: `isBefore`, `isAfter`, and `isEqual`

- `isBefore(other)` / `isAfter(other)`: straightforward chronological comparisons.
  - `nowDt.isBefore(ofDt)` → `false` (now, 2024, is *not* before 2016).
  - `nowDt.isAfter(ofDt)` → `true` (now is after 2016, obviously).
- `isEqual(other)`: checks whether two date/times represent the **same point in time** —
  deliberately a *different* method from the inherited `equals(Object)`.

#### `isEqual()` vs `equals()` — why both exist

> "isEqual는 뭐냐면 단순히 비교 대상이 그냥 시간적으로만 같으면 true를 반환해요. 그러니까 뭐 객체도
> 달라도 되고 타임존이 다 달라도 돼요... equals는 객체 타입, 타임존 등등 내부의 구성요소가 모두
> 같아야 true를 반환합니다."

| | Compares | Time zone must match? | Example |
|---|---|---|---|
| `isEqual(other)` | Whether the two moments are **chronologically the same point in time**, after accounting for any offset/zone difference | No | Seoul's 9:00 AM and UTC's 0:00 (midnight) — same instant → `true` |
| `equals(other)` | Whether every internal field matches — object type, time zone/offset, etc. | Yes | Same Seoul 9:00 AM vs. UTC 0:00 example → `false`, because the time zone data itself differs |

- Worked example from the transcript: if it's `9:00 AM` in Seoul, it's `0:00` (midnight) in the UK —
  these are genuinely **the same instant**, just expressed in two different time zones (like a phone
  call where one side says "it's 9 AM here" and the other says "it's midnight here" — both describing
  the exact same moment).
  - `isEqual(...)` between these two returns `true` — it computes and compares purely by elapsed
    time, ignoring everything else.
  - `equals(...)` between the same two returns `false` — because the time zone data differs between
    them, even though the represented instant is identical.
- Practical guidance: when you want to know "do these two values refer to the same moment in time?"
  — reach for `isEqual()`. `equals()` is a stricter, structural comparison (useful for things like
  using these objects as map keys or in collections), not a "same point in time" check.

---

## Summary

- `LocalDate` (date only), `LocalTime` (time only), and `LocalDateTime` (both) cover an estimated
  ~95% of real-world (domestic/non-global) Java development — understood well, everything with time
  zones later is just this same API plus a zone.
- Both `LocalDate` and `LocalTime` follow the same shape: `now()` for the current value, `of(...)`
  for a specific value, and `plusXxx(...)` methods for calculation.
- **Every date/time type is immutable** — `plusDays`, `plusSeconds`, etc. always return a *new*
  instance; forgetting to capture the return value silently does nothing, exactly like `String` and
  the wrapper classes earlier in the course.
- `LocalDateTime` is implemented by literally composing a `LocalDate` field and a `LocalTime` field
  — a deliberately highlighted example of clean, "Lego block" class design.
  - Split apart with `toLocalDate()` / `toLocalTime()`.
  - Reassemble with `LocalDateTime.of(localDate, localTime)`.
- Comparisons: `isBefore()` / `isAfter()` for simple chronological ordering.
- `isEqual()` vs `equals()` is the key distinction to remember:
  - `isEqual()` — true if the two values represent the **same instant**, regardless of time zone or
    object type (e.g. Seoul 9 AM == UTC midnight).
  - `equals()` — true only if **every internal field matches**, including time zone/offset — the
    same two values above would return `false` here.
- Next lecture: `ZonedDateTime` — bringing time zones into everything just covered here.
