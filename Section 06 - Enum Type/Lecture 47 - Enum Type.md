# Lecture 47 — Enum Type

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 6. 열거형 - ENUM (4/11)

Previous: [Lecture 46 - Type-Safe Enum Pattern](./Lecture%2046%20-%20Type-Safe%20Enum%20Pattern.md)

Lecture 46 built the type-safe enum pattern by hand — a class, a fixed set of `static final`
instances, and a `private` constructor. This lecture introduces Java's actual **`enum`** keyword,
which gives you that exact same pattern essentially for free.

> "자바는 앞서 설명한 타입 안전 열거형 패턴을 매우 편리하게 사용할 수 있는 Enum 타입을 제공한다."

---

## 1. What `enum` means, restated precisely

- `enum` is short for **enumeration** — "열거," listing out a fixed set of items.
- "Enumeration" means defining a set of **named constants** — in this case, `BASIC`, `GOLD`,
  `DIAMOND`.
- In code, these constants represent a predefined, closed set of values.
- In other words: "member grade" can only ever be one of `BASIC` / `GOLD` / `DIAMOND` — nothing
  else.
- Java's `enum` provides type safety, improves code readability, and represents a predictable,
  fixed set of values.

---

## 2. Defining an enum: `Grade`

```java
package enumeration.ex3;

public enum Grade {
    BASIC, GOLD, DIAMOND
}
```

- Where `ClassGrade` used the keyword `class`, here it's simply **`enum`** instead.
- The three constant names are just listed directly, comma-separated.
- That's the entire definition — no `static final` fields, no manual `new`, no `private`
  constructor to write.

> "이전에 있던 타입 안전 열거형 패턴을 직접 구현은 이렇게 했죠. 근데 여러분 이거랑 거의 똑같은
> 코드거든요. 앞에 enum이라고만 적어주시면 이렇게 바꾸면 됩니다."

### 2.1 What this is equivalent to

The instructor shows that `Grade` written as an `enum` is effectively the same as writing it out
by hand as a class:

```java
public class Grade extends Enum {
    public static final Grade BASIC = new Grade();
    public static final Grade GOLD = new Grade();
    public static final Grade DIAMOND = new Grade();

    //private 생성자 추가
    private Grade() {}
}
```

- **An enum is still a class.** Writing `enum` instead of `class` doesn't create some entirely
  different kind of thing — it's a class with some built-in constraints and conveniences layered
  on top.
- **It automatically extends `java.lang.Enum`.** This base class provides several useful built-in
  features (covered in the next lecture on enum methods).
- **It cannot be constructed externally.** Just like the hand-built `ClassGrade`, there's an
  implicit `private` constructor — you simply never have to write it yourself.

---

## 3. Confirming the instances with `EnumRefMain`

```java
package enumeration.ex3;

public class EnumRefMain {

    public static void main(String[] args) {
        System.out.println("class BASIC = " + Grade.BASIC.getClass());
        System.out.println("class GOLD = " + Grade.GOLD.getClass());
        System.out.println("class DIAMOND = " + Grade.DIAMOND.getClass());

        System.out.println("ref BASIC = " + refValue(Grade.BASIC));
        System.out.println("ref GOLD = " + refValue(Grade.GOLD));
        System.out.println("ref DIAMOND = " + refValue(Grade.DIAMOND));
    }

    private static String refValue(Object grade) {
        return Integer.toHexString(System.identityHashCode(grade));
    }
}
```

**Output:**

```
class BASIC = class enumeration.ex3.Grade
class GOLD = class enumeration.ex3.Grade
class DIAMOND = class enumeration.ex3.Grade

ref BASIC = x001
ref GOLD = x002
ref DIAMOND = x003
```

### 3.1 Why a helper method (`refValue`) was needed just to see the reference

- Printing `Grade.BASIC` directly does **not** show its reference value, because `enum` overrides
  `toString()` automatically — it prints the constant's own name (e.g. `"BASIC"`) instead.
- To actually inspect the underlying reference, the instructor builds `refValue(Object grade)`:
  - `System.identityHashCode(grade)` — returns a numeric identity value for the object, something
    covered earlier in the `Object`-class section of the course.
  - `Integer.toHexString(...)` — converts that number to hexadecimal, matching the familiar
    `@xxxxx`-style reference format normally seen when printing an object.
- Result: `BASIC`/`GOLD`/`DIAMOND` all report the same type (`Grade`), but three distinct reference
  values (`x001`, `x002`, `x003`) — confirming this is exactly the same underlying structure as the
  hand-built `ClassGrade` from Lecture 46: one type, three separately-created instances.

---

## 4. Using the enum in `DiscountService`

```java
package enumeration.ex3;

public class DiscountService {

    public int discount(Grade grade, int price) {
        int discountPercent = 0;

        //enum switch 변경 가능
        if (grade == Grade.BASIC) {
            discountPercent = 10;
        } else if (grade == Grade.GOLD) {
            discountPercent = 20;
        } else if (grade == Grade.DIAMOND) {
            discountPercent = 30;
        } else {
            System.out.println("할인X");
        }

        return price * discountPercent / 100;
    }
}
```

- Structurally **identical** to the `ClassGrade` version from Lecture 46 — only the parameter type
  changed from `ClassGrade` to `Grade`, and comparisons are still `==` (still safe, for the same
  reason: a fixed, closed set of instances).
- Note in the source: `//enum switch 변경 가능` — a quick pointer that this `if`/`else if` chain
  could instead be written as a `switch` statement, since enums can be used directly in `switch`.
  (Not expanded on further in this lecture — just flagged as a capability to come back to.)

### 4.1 Running it: `EnumEx3_1`

```java
package enumeration.ex3;

public class EnumEx3_1 {
    public static void main(String[] args) {
        int price = 10000;
        DiscountService discountService = new DiscountService();
        int basic = discountService.discount(Grade.BASIC, price);
        int gold = discountService.discount(Grade.GOLD, price);
        int diamond = discountService.discount(Grade.DIAMOND, price);

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

> "열거형의 사용법이 앞서 타입 안전 열거형 패턴을 직접 구현한 코드와 같은 것을 확인할 수 있다."

---

## 5. Confirming it can't be constructed externally

```java
package enumeration.ex3;

public class EnumEx3_2 {
    public static void main(String[] args) {
        int price = 10000;
        DiscountService discountService = new DiscountService();

        // new Grade() — 컴파일 오류, enum 타입은 인스턴스로 생성할 수 없음
    }
}
```

- Attempting `new Grade()` produces a **compile error**: enum types cannot be instantiated.
- This confirms the `private`-constructor behavior is baked in automatically — no explicit
  `private Grade() {}` had to be written for this protection to exist.

---

## 6. Advantages of `enum` (recap, now backed by the implementation)

- **Improved type safety.** Since an enum consists only of a predefined, fixed set of constants,
  there is no possibility of an invalid value ever being assigned — any attempt produces a
  compile error.
- **Conciseness and consistency.** Using an enum makes code shorter and clearer, and guarantees
  data consistency (no `"gold"` vs `"GOLD"` ambiguity — there's exactly one `Grade.GOLD`).
- **Extensibility.** Adding a brand-new grade (say, a `VIP` tier) is as simple as adding one more
  name to the enum's constant list — no boilerplate class/constant/constructor work required, the
  way Lecture 46's hand-built pattern would have needed.
- **Usable in `switch` statements** — noted here as a nice side benefit, to be explored later.

---

## 7. A nice bonus: `static import`

With the instructor reusing the earlier call sites written as `Grade.BASIC`, a convenience is
introduced: Java supports **static import** for constants, and enum constants qualify as
constants.

- In IntelliJ: place the cursor on `Grade.BASIC`, Alt+Enter → "Add static import," and the call
  site shortens from `Grade.BASIC` down to just `BASIC`.
- Since enum constants are constants, they can be statically imported just like any other
  `public static final` field.
- The instructor notes this is used **heavily** in real-world code:

> "제가 실무에서는 이런 상수의 스태틱 임포트를 정말 많이 써요. 이러면 읽기가 훨씬 더 편해지거든요."

- Reading `discountService.discount(BASIC, price)` reads more naturally than
  `discountService.discount(Grade.BASIC, price)` — "the member's grade is BASIC" rather than
  "the member's grade is Grade-dot-BASIC."
- This isn't enum-specific syntax — it's regular Java static-import behavior that happens to apply
  cleanly to enum constants, since each one is itself a `public static final` field under the hood.

---

## Summary

- `enum` is Java's built-in language support for the type-safe enum pattern from Lecture 46 — same
  guarantees, dramatically less code.
- `public enum Grade { BASIC, GOLD, DIAMOND }` is effectively equivalent to manually writing a
  class with three `public static final` instances and a `private` constructor — an enum **is**
  a class, one that automatically extends `java.lang.Enum` and cannot be instantiated externally.
- `toString()` is automatically overridden to print the constant's name, so inspecting the actual
  reference value requires a workaround (e.g. `System.identityHashCode(...)` +
  `Integer.toHexString(...)`).
- Using `Grade` in `DiscountService` is structurally identical to the Lecture 46 `ClassGrade`
  version — `==` comparisons, same logic — just a different parameter type.
- `new Grade()` is a compile error — enums can't be constructed from outside, with no extra code
  required to enforce it.
- Advantages carry over directly from Lecture 46 (type safety, consistency) plus **extensibility**
  (adding a new grade is a one-line change) and usability inside `switch` statements.
- Bonus: enum constants can be statically imported, which the instructor uses heavily in practice
  for more natural-reading call sites.
- Next lecture covers the **key methods** enums provide (inherited from `java.lang.Enum`).
