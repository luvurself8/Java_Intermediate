# Lecture 44 — String and Type Safety 1

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 6. 열거형 - ENUM (1/11)

This is the first lecture on Java's **Enum Type** (열거형). Before explaining what an enum
actually is, the course spends this lecture building up the exact problem that enums exist to
solve — using plain `String` to represent a fixed set of categories.

> "자바가 제공하는 열거형을 제대로 이해하려면 먼저 열거형이 생겨난 이유를 알아야 한다."

---

## 1. The business requirement

- Customers are split into **3 grades**.
- A purchase gets discounted based on the customer's grade; any fractional discount amount is
  dropped (integer division).
- `BASIC` → 10% discount
- `GOLD` → 20% discount
- `DIAMOND` → 30% discount
- Example: a `GOLD` member buying a ₩10,000 item gets a ₩2,000 discount.

The task: build a class that, given a grade and a price, calculates the discount amount.

---

## 2. First attempt: grade as a plain `String`

### 2.1 `DiscountService`

```java
package enumeration.ex0;

public class DiscountService {

    public int discount(String grade, int price) {
        int discountPercent = 0;

        if (grade.equals("BASIC")) {
            discountPercent = 10;
        } else if (grade.equals("GOLD")) {
            discountPercent = 20;
        } else if (grade.equals("DIAMOND")) {
            discountPercent = 30;
        } else {
            System.out.println(grade + ": 할인X");
        }

        return price * discountPercent / 100;
    }
}
```

- `price * discountPercent / 100` — price × percentage / 100 gives the discount amount.
- If anything other than the three known grades is passed in, the `else` branch prints
  "할인X" (no discount), and since `discountPercent` stays `0`, the discount amount naturally
  computes to `0`.
- Simplifying assumption for this example: `grade` is never `null`.

### 2.2 Running it with valid input: `StringGradeEx0_1`

```java
package enumeration.ex0;

public class StringGradeEx0_1 {

    public static void main(String[] args) {
        int price = 10000;

        DiscountService discountService = new DiscountService();
        int basic = discountService.discount("BASIC", price);
        int gold = discountService.discount("GOLD", price);
        int diamond = discountService.discount("DIAMOND", price);

        System.out.println("BASIC 등급의 할인 금액: " + basic);
        System.out.println("GOLD 등급의 할인 금액: " + gold);
        System.out.println("DIAMOND 등급의 할인 금액: " + diamond);
    }
}
```

**Output:**

```
BASIC 등급의 할인 금액: 1000
GOLD 등급의 할인 금액: 2000
DIAMOND 등급의 할인 금액: 3000
```

- Each grade correctly gets its matching discount. So far, so good.

---

## 3. Where it breaks: typing a grade wrong

The problem surfaces as soon as the input string isn't typed perfectly.

### 3.1 `StringGradeEx0_2` — three realistic mistakes

```java
package enumeration.ex0;

public class StringGradeEx0_2 {

    public static void main(String[] args) {
        int price = 10000;

        DiscountService discountService = new DiscountService();

        // 존재하지 않는 등급
        int vip = discountService.discount("VIP", price);
        System.out.println("VIP 등급의 할인 금액: " + vip);

        // 오타
        int diamondd = discountService.discount("DIAMONDD", price);
        System.out.println("DIAMONDD 등급의 할인 금액: " + diamondd);

        // 소문자 입력
        int gold = discountService.discount("gold", price);
        System.out.println("gold 등급의 할인 금액: " + gold);
    }
}
```

**Output:**

```
VIP: 할인X
VIP 등급의 할인 금액: 0

DIAMONDD: 할인X
DIAMONDD 등급의 할인 금액: 0

gold: 할인X
gold 등급의 할인 금액: 0
```

Each mistake, walked through as a real scenario the instructor narrates:

- **A grade that doesn't exist — `"VIP"`.** Trying to recall the highest grade, and misremembering
  it as "VIP" instead of "DIAMOND" — an entirely made-up value that was never one of the three
  valid grades to begin with.
- **A typo — `"DIAMONDD"`.** An extra `D` accidentally typed at the end. Perfectly plausible with
  a fast keyboard — and the string is still a string, so nothing about typing an extra character
  looks wrong to the compiler.
- **Wrong case — `"gold"`.** Forgetting that grades are all uppercase and typing lowercase instead.
  The instructor frames this with a mini "war story": shipping this to production, and then every
  `GOLD` member suddenly isn't getting their discount — a real incident, caused entirely by a
  single lowercase typo.

In every case: the code **compiles fine**, and **runs without crashing** — it just silently falls
through to "no discount," because none of the `if`/`else if` conditions matched.

---

## 4. Naming the actual problem

> "등급에 문자열을 사용하는 지금의 방식은 다음과 같은 문제가 있다."

Two related issues:

- **타입 안정성 부족 (lack of type safety)** — a string is easy to typo, and nothing stops an
  invalid value from being passed in.
- **데이터 일관성 (data consistency)** — the same grade could be written as `"GOLD"`, `"gold"`,
  `"Gold"`, etc. — multiple different-looking strings all meaning to represent the same thing,
  with nothing enforcing consistency.

### 4.1 Why `String` structurally can't prevent this

- **No way to restrict the allowed values.** Using a `String` to represent a state or category
  means any string can be typed in by mistake — e.g. representing days of the week
  (`"Monday"`, `"Tuesday"`, ...) as strings risks a typo like `"Munday"`, or an entirely invalid
  value like `"Funday"`.
- **No compile-time error detection.** An invalid value like this is never caught at compile
  time — it only shows up at **runtime**, which makes it much harder to debug. From the
  compiler's point of view, a `String` parameter happily accepts *any* string — there's no
  distinction between a valid grade and a typo; both are just "a string," so nothing looks wrong
  until the program actually runs (and, as in the `GOLD`→`gold` story, possibly not until it's
  already live in production).

### 4.2 What would actually be needed to fix this

> "이런 문제를 해결하려면 특정 범위로 값을 제한해야 한다."

- The fix would require restricting the method to only accept the **exact** strings `"BASIC"`,
  `"GOLD"`, `"DIAMOND"` — nothing else.
- But `String` as a type accepts *any* string value — from Java's own grammar/syntax rules, there
  is nothing wrong with passing `"VIP"` or `"gold"` to a method that takes a `String` parameter.
- Conclusion for this lecture: **using the `String` type alone cannot fundamentally solve this
  problem.** Something else is needed — which the next lecture starts building toward.

---

## Summary

- Motivating example: a grade-based discount system (`BASIC`/`GOLD`/`DIAMOND`), with grade passed
  around as a plain `String`.
- Using plain strings for a fixed set of categories lets three kinds of mistakes slip through
  silently, with no compiler error and no crash — they just silently fail to discount:
  - a nonexistent value (`"VIP"`)
  - a typo (`"DIAMONDD"`)
  - wrong casing (`"gold"` instead of `"GOLD"`)
- Root causes: **lack of type safety** (anything can be typed, nothing is restricted to the valid
  set) and **lack of data consistency** (the same logical value can be written multiple ways).
- These bugs are only caught at **runtime**, not compile time — meaning they can ship to
  production undetected, as in the instructor's "every GOLD member stopped getting discounts"
  story.
- `String` cannot structurally fix this: it accepts any value by design, so there's no way to
  restrict it to just the valid grade values using `String` alone.
- Next lecture ("String and Type Safety 2") continues exploring this problem before Java's actual
  `enum` solution is introduced.
