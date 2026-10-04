# Lecture 45 — String and Type Safety 2

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 6. 열거형 - ENUM (2/11)

Previous: [Lecture 44 - String and Type Safety 1](./Lecture%2044%20-%20String%20and%20Type%20Safety%201.md)

Lecture 44 ended with a production incident: a `gold`/`GOLD` casing typo silently broke discounts
for every gold member. This lecture tries the obvious first fix — **string constants** — and shows
exactly why it still isn't enough.

---

## 1. The "obvious" fix: turn the literals into constants

> "아, 상수를 만들면 되겠다라는 생각이 드는 거예요."

The idea: instead of typing `"BASIC"` / `"GOLD"` / `"DIAMOND"` as raw literals everywhere, define
them **once** as named constants, and have all the calling code reference the constant instead of
re-typing the string.

### 1.1 `StringGrade` — the constants holder

```java
package enumeration.ex1;

public class StringGrade {
    public static final String BASIC = "BASIC";
    public static final String GOLD = "GOLD";
    public static final String DIAMOND = "DIAMOND";
}
```

- `public static final` is what makes each of these a **constant**:
  - `static` — declared at the class level, not per-instance.
  - `final` — the reference/value can never be reassigned once set.
- Accessing `StringGrade.BASIC` always yields the exact string `"BASIC"` — but now the *name* of
  the constant is what callers type, not the raw literal.

### 1.2 `DiscountService`, updated to use the constants

```java
package enumeration.ex1;

public class DiscountService {

    //StringGrade를 참고하세요.
    public int discount(String grade, int price) {
        int discountPercent = 0;

        if (grade.equals(StringGrade.BASIC)) {
            discountPercent = 10;
        } else if (grade.equals(StringGrade.GOLD)) {
            discountPercent = 20;
        } else if (grade.equals(StringGrade.DIAMOND)) {
            discountPercent = 30;
        } else {
            System.out.println(grade + ": 할인X");
        }

        return price * discountPercent / 100;
    }
}
```

- Logic is otherwise identical to Lecture 44's version — only the literals became constant
  references.
- Note the comment `//StringGrade를 참고하세요.` ("see StringGrade") — this becomes important later.

### 1.3 Calling it with the constants: `StringGradeEx1_1`

```java
package enumeration.ex1;

public class StringGradeEx1_1 {

    public static void main(String[] args) {
        int price = 10000;

        DiscountService discountService = new DiscountService();
        int basic = discountService.discount(StringGrade.BASIC, price);
        int gold = discountService.discount(StringGrade.GOLD, price);
        int diamond = discountService.discount(StringGrade.DIAMOND, price);

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

### 1.4 Why this genuinely helps

- Typing `StringGrade.` in the IDE triggers **autocomplete** — the available constants show up in
  a dropdown, so there's nothing to misremember or mistype.
- Typing the constant name wrong (e.g. a typo in `BASIC`) is now a **compile-time error** —
  the code simply won't compile, instead of silently failing at runtime.
- Casing mistakes like `gold` vs `GOLD` are no longer possible either, since you're selecting a
  known constant name from autocomplete, not typing a literal string by hand.

> "문자열 상수를 사용한 덕분에 전체적으로 코드가 더 명확해졌고... 실수로 상수의 이름을 잘못
> 입력하면 컴파일 시점에 오류가 발생한다는 점입니다. 따라서 오류를 쉽고 빠르게 찾을 수 있어요."

---

## 2. But this still doesn't fix the root problem

> "인간의 욕심이 끝이 없습니다... String 상수를 사용해도 지금까지 발생한 문제들을 근본적으로
> 해결할 수는 없어요."

The method signature is still:

```java
public int discount(String grade, int price) {}
```

It still takes a plain `String`. Nothing about the method's type **requires** the caller to use
`StringGrade.GOLD` — it only *suggests* it, via a comment.

### 2.1 Proving it: `StringGradeEx1_2`

```java
package enumeration.ex1;

public class StringGradeEx1_2 {

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

- Nothing stops a caller from bypassing `StringGrade` entirely and typing a raw literal string
  directly, exactly as before.
- Same three failure modes from Lecture 44 (nonexistent value, typo, wrong case) still compile and
  run without error — they just silently produce no discount, same as before.

### 2.2 The deeper issue: nothing in the type signature communicates the constraint

> "discount()를 호출하는 개발자가 StringGrade가 어디 있는지 어떻게 알 수 있을까? 코드를 보면
> 분명히 String은 다 입력할 수 있다고 되어있다."

```java
public int discount(String grade, int price) {}
```

- Looking at just this signature, there's **no signal at all** that `grade` is supposed to be one
  of a specific, limited set of values.
- The only thing hinting at the restriction is a source-code comment (`//StringGrade를
  참고하세요.`) sitting above the method — a comment a caller might never even see, let alone read.
- The type itself (`String`) permits literally any string — so the method's own declared type is
  actively lying about what values are actually valid.

---

## 3. "Whose fault is this?" — a short case study

The lecture poses this as a mini scenario with two developers:

- **Developer A** (senior) — wrote `DiscountService`, and added a comment saying "see
  `StringGrade`" above the `discount` method.
- **Developer B** (newer) — needs to call `discount(...)`, sees a planner/PM say the grade is
  "Gold," and — without noticing the comment — just passes the literal string `"gold"` directly.
  Ships to production. Every gold-tier member's discount silently breaks the next day.

When the team investigates and traces it back to Developer B's code, the senior developer's first
reaction is to blame Developer B for not reading the comment.

**The instructor's verdict: the senior developer is at fault, not the newer one.**

> "애초에 여기에 받을 수 있는 걸 String을 쓰면 안 돼요. 여기는 Basic, 골드, 다이아몬드만 쓸 수 있는
> 다른 무언가를 써야 돼요. 이미 자바에서는 그런 기법들을 다 제공을 해요."

- The real design mistake was accepting a plain `String` for `grade` in the first place.
- A method's parameter type should make the valid set of inputs **self-evident from the type
  itself** — not rely on a comment that a caller may never read.
- Java already provides language-level tools to restrict a parameter to "only these specific
  values" — the responsibility for using them belongs to whoever designed the API
  (`DiscountService`), not whoever calls it.

---

## Summary

- Turning the grade literals into `public static final String` constants (`StringGrade`) is a real
  improvement: autocomplete prevents typos and wrong casing, and misspelling a constant **name**
  is now a compile error.
- It does **not** fix the underlying problem: `discount(String grade, ...)` still accepts *any*
  string, so a caller can bypass the constants entirely and pass a raw, invalid literal — and the
  code will compile and silently do nothing, exactly as before.
- The only thing warning callers to use `StringGrade` was a source comment — invisible from the
  type signature itself, and easy to miss.
- Root cause, restated more precisely than in Lecture 44: the **parameter type** (`String`) itself
  fails to express the actual constraint ("only `BASIC`/`GOLD`/`DIAMOND` are valid") — and that is
  a design mistake in the API, not a mistake by whoever calls it.
- The fix requires a type that can **enforce** this restriction at the language level, instead of
  just hinting at it. Next lecture introduces exactly that: the **type-safe enum pattern**.
