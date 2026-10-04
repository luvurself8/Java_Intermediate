# Lecture 46 — Type-Safe Enum Pattern

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 6. 열거형 - ENUM (3/11)

Previous: [Lecture 45 - String and Type Safety 2](./Lecture%2045%20-%20String%20and%20Type%20Safety%202.md)

Lecture 45 ended on the real diagnosis: the problem was never the caller — it was that
`DiscountService.discount(...)` accepted a plain `String` in the first place. This lecture builds,
by hand, the classic fix many Java developers converged on before `enum` existed: the
**type-safe enum pattern**.

---

## 1. What "열거(enumeration)" actually means

- `enum` is short for **enumeration**, which translates to "열거" — to **list out / enumerate**
  a set of items.
- In this case, enumerating the three member grades: `BASIC`, `GOLD`, `DIAMOND`.

> "타입 안전 열거형 패턴을 사용하면 이렇게 나열한 항목만 사용할 수 있다는 게 핵심이에요. 나열한
> 항목인 베이직, 골드, 다이아몬드 외에 다른 거는 사용할 수 없다라는 거예요."

- The entire point of the pattern: **only the enumerated items can be used** — nothing outside
  that fixed list is accepted, unlike `String`, which accepts any value at all.

---

## 2. Step 1 — A class that only has 3 possible instances: `ClassGrade`

```java
package enumeration.ex2;

public class ClassGrade {
    public static final ClassGrade BASIC = new ClassGrade();   //x001
    public static final ClassGrade GOLD = new ClassGrade();    //x002
    public static final ClassGrade DIAMOND = new ClassGrade(); //x003
}
```

- A class is created to represent "grade" as a concept — `ClassGrade`.
- Three constants are declared **inside** it, each one a separately-created instance of
  `ClassGrade` itself: `BASIC`, `GOLD`, `DIAMOND`.
- `static` places these constants at the class level (created once, at class-loading time).
- `final` prevents the reference from ever being reassigned.
- Conceptually: 3 `ClassGrade` instances get created in the heap when the class loads, and each
  constant holds a reference to one of them — labeled here as `x001`, `x002`, `x003` for clarity.

### 2.1 Confirming this with `ClassRefMain`

```java
package enumeration.ex2;

public class ClassRefMain {

    public static void main(String[] args) {
        System.out.println("class BASIC = " + ClassGrade.BASIC.getClass());
        System.out.println("class GOLD = " + ClassGrade.GOLD.getClass());
        System.out.println("class DIAMOND = " + ClassGrade.DIAMOND.getClass());

        System.out.println("ref BASIC = " + ClassGrade.BASIC);
        System.out.println("ref GOLD = " + ClassGrade.GOLD);
        System.out.println("ref DIAMOND = " + ClassGrade.DIAMOND);
    }
}
```

**Output:**

```
class BASIC = class enumeration.ex2.ClassGrade
class GOLD = class enumeration.ex2.ClassGrade
class DIAMOND = class enumeration.ex2.ClassGrade

ref BASIC = enumeration.ex2.ClassGrade@x001
ref GOLD = enumeration.ex2.ClassGrade@x002
ref DIAMOND = enumeration.ex2.ClassGrade@x003
```

- `getClass()` on all three returns the same type, `ClassGrade` — because all three were built
  from `new ClassGrade()`.
- But printing the object reference itself shows **three different reference values**
  (`@x001`, `@x002`, `@x003`) — three genuinely separate instances, each one just happens to live
  behind a constant with a meaningful name.
- Same type, three distinct instances — exactly matching the diagram: one `ClassGrade` class, with
  `BASIC`/`GOLD`/`DIAMOND` each pointing at their own separately-created object.

---

## 3. Step 2 — Making `DiscountService` accept `ClassGrade` instead of `String`

```java
package enumeration.ex2;

public class DiscountService {

    public int discount(ClassGrade classGrade, int price) {
        int discountPercent = 0;

        if (classGrade == ClassGrade.BASIC) {
            discountPercent = 10;
        } else if (classGrade == ClassGrade.GOLD) {
            discountPercent = 20;
        } else if (classGrade == ClassGrade.DIAMOND) {
            discountPercent = 30;
        } else {
            System.out.println("할인X");
        }

        return price * discountPercent / 100;
    }
}
```

- The parameter type changed from `String grade` to **`ClassGrade classGrade`**.
- This single change is the whole point: **the type itself now restricts what can be passed in.**
  Only a `ClassGrade` value can be given — nothing else compiles.
- Comparisons switched from `.equals(...)` to **`==`**.

### 3.1 Why `==` is correct here (and safe)

- `classGrade == ClassGrade.BASIC` compares **references**, not contents — but that's exactly
  right here, because there are only ever **3 instances of `ClassGrade` in existence**, and the
  argument passed in must be one of those same 3 instances (the constants defined on `ClassGrade`
  itself).
- Walking through an example: if a caller passes `ClassGrade.GOLD` (reference `x002`), then inside
  `discount(...)`, `classGrade` *is* that same `x002` reference. Comparing it against
  `ClassGrade.BASIC` (`x001`) → `false`; comparing it against `ClassGrade.GOLD` (`x002`) → `true`,
  since it's literally the same object in memory.
- So reference comparison (`==`) is reliable precisely because the whole design guarantees there
  are only 3 possible instances to ever compare against.

### 3.2 Using it: `ClassGradeEx2_1`

```java
package enumeration.ex2;

public class ClassGradeEx2_1 {

    public static void main(String[] args) {
        int price = 10000;

        DiscountService discountService = new DiscountService();
        int basic = discountService.discount(ClassGrade.BASIC, price);
        int gold = discountService.discount(ClassGrade.GOLD, price);
        int diamond = discountService.discount(ClassGrade.DIAMOND, price);

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

- Passing anything other than `ClassGrade.BASIC` / `.GOLD` / `.DIAMOND` (e.g. trying to type the
  raw string `"GOLD"`) is now a **compile error** — "cannot find symbol" — since the parameter
  type is `ClassGrade`, not `String`.

---

## 4. The remaining hole: nothing stops `new ClassGrade()`

> "이 방식도 한 가지 단점이 있습니다... 생각하셨으면 대단한 겁니다."

Even with the parameter locked down to `ClassGrade`, there's still a gap: a developer calling
`DiscountService` could reasonably look at the signature, see `ClassGrade` is needed, and think
"I should construct one of these myself":

```java
package enumeration.ex2;

public class ClassGradleEx2_2 {

    public static void main(String[] args) {
        int price = 10000;

        DiscountService discountService = new DiscountService();

        ClassGrade newClassGrade = new ClassGrade(); //x009
        int result = discountService.discount(newClassGrade, price);
        System.out.println("newClassGrade 등급의 할인 금액: " + result);
    }
}
```

**Output:**

```
할인X
newClassGrade 등급의 할인 금액: 0
```

- `new ClassGrade()` is a perfectly valid, compiling call — it creates a brand-new instance
  (conceptually `x009`), completely unrelated to `x001`/`x002`/`x003`.
- Since `classGrade == ClassGrade.BASIC/GOLD/DIAMOND` are all comparing against the "real" three
  instances, this fresh `x009` matches none of them — it silently falls into the `else` branch,
  "할인X," and the discount comes out `0`.

**Important framing from the instructor:**

> "이 개발자가 타입을 보고 new 해서 넘기는 건 그 개발자의 잘못은 아닙니다."

- The developer who wrote `new ClassGrade()` did nothing wrong — the type signature genuinely
  allows it. Just like in Lecture 45, the responsibility is on the **class design**, not the
  caller.
- This time, though, the actual fix is small.

---

## 5. Closing the hole: a `private` constructor

> "이 문제를 해결하려면 외부에서 이 클래스 그레이드를 생성할 수 없도록 막으면 된다. 기본 생성자를
> private으로 변경하자."

```java
package enumeration.ex2;

public class ClassGrade {
    public static final ClassGrade BASIC = new ClassGrade();
    public static final ClassGrade GOLD = new ClassGrade();
    public static final ClassGrade DIAMOND = new ClassGrade();

    //private 생성자 추가
    private ClassGrade() {}
}
```

- Adding a `private` constructor means the constructor can only be called from **inside**
  `ClassGrade` itself.
- The three `static final` constants still work fine — they're defined inside `ClassGrade`, so
  they have access to the now-private constructor.
- Any code **outside** `ClassGrade` attempting `new ClassGrade()` now fails to **compile** —
  exactly the outcome needed.

### 5.1 Confirming it blocks the problem call

Re-running the earlier `new ClassGrade()` attempt now fails at compile time:

```java
/*
        ClassGrade newClassGrade = new ClassGrade(); //생성자 private으로 막아야 함
        int result = discountService.discount(newClassGrade, price);
        System.out.println("newClassGrade 등급의 할인 금액: " + result);
*/
```

(commented out in the source, since it no longer compiles)

- A developer hitting this compile error gets immediate, unambiguous feedback: "I can't construct
  this myself — I'm clearly meant to use one of the predefined constants instead."
- This is a much better failure mode than the silent "할인X" / `0` from before — the mistake is
  now caught **the moment it's written**, instead of discovered later at runtime.

---

## 6. The complete pattern

> "클래스를 정의해서 이 클래스에 대한 타입만 쓸 수 있게 하고, 그리고 여기 안에 어떤 항목만 쓸 수
> 있는지 나열하는 거예요. 그리고 밖에서 생성하지 못하게 하면 딱 이것만 쓸 수 있게 되죠."

The type-safe enum pattern, put together, is exactly three ingredients:

1. Define a dedicated class (`ClassGrade`) to represent the category/type.
2. Inside it, declare a fixed, enumerated set of `public static final` instances
   (`BASIC`, `GOLD`, `DIAMOND`) — these are the *only* valid values.
3. Make the constructor `private`, so no code outside the class can create any further instances.

With all three in place, a method that takes a `ClassGrade` parameter can **only** ever receive
one of the three predefined constants — nothing else is possible, and the compiler enforces it.

---

## 7. Advantages

- **타입 안정성 향상 (improved type safety).** Only the predefined objects can be used, which
  fundamentally prevents invalid values from ever being passed in.
- **데이터 일관성 (data consistency).** Since only the enumerated objects are ever used, there's no
  risk of inconsistent representations (no more `"gold"` vs `"GOLD"` vs `"Gold"`).
- **Restricted instance creation.** The class pre-creates a fixed, small number of instances, and
  external code is limited to using only those — guaranteeing only the predefined values are ever
  in circulation.
- **The single biggest win: this is enforced at *compile time*, not runtime.**

> "가장 좋은 오류, 컴파일 오류는 내가 지금 코딩하면서 바로 알 수 있잖아요... 가장 나쁜 오류는
> 실행 도중에, 고객이 실행해서 발견하는 거예요."

- A compile error shows up immediately, in the IDE, while writing the code.
- A runtime error — the worst kind — might only be discovered after shipping, by an actual user
  running into it in production (exactly the "GOLD members lost their discount" incident from
  Lecture 44).
- If a method requires a specific type like `ClassGrade`, only instances of that exact type can
  ever be passed to it — here, only `BASIC`, `GOLD`, or `DIAMOND`.

---

## 8. The catch: this pattern requires a lot of boilerplate

> "이 패턴을 구현하려면 다음과 같이 많은 코드를 작성해야 한다. 그리고 private 생성자를 추가하는 등
> 유의해야 하는 부분들도 있다."

- Every single enumerated type needs: its own class, one `static final` field per value, and a
  `private` constructor — all written out by hand, every time.
- It's easy to forget a step (e.g. forgetting the `private` constructor), which reopens exactly
  the hole this pattern exists to close.
- Despite the verbosity, this pattern was genuinely popular and widely used in real projects,
  historically — developers accepted the extra code because the safety guarantee was worth it.

---

## Summary

- The **type-safe enum pattern** fixes the core problem from Lectures 44–45 by changing the
  parameter type itself from `String` to a dedicated class (`ClassGrade`), so only specific,
  predefined values can ever be passed.
- Built from three ingredients: (1) a dedicated class, (2) a small, fixed set of
  `public static final` instances declared inside it, (3) a `private` constructor so no other
  instances can ever be created from outside.
- Comparisons use `==` (reference equality) instead of `.equals(...)` — safe here specifically
  because the private constructor guarantees only the enumerated instances exist.
- Without the `private` constructor, a well-meaning caller can still do `new ClassGrade()` and
  silently break the logic — not their fault, since the type permits it; the `private` constructor
  closes this gap and turns that mistake into a compile error instead.
- Main advantage: invalid values are now rejected **at compile time** — the best kind of error —
  instead of silently failing at runtime, possibly in production.
- Main drawback: it requires writing a fair amount of boilerplate for every enumerated type, with
  easy-to-forget details (like the `private` constructor).
- Because of that verbosity, Java eventually added a built-in language feature that gives you this
  exact pattern "for free" — the real `enum` keyword, introduced in the next lecture.
