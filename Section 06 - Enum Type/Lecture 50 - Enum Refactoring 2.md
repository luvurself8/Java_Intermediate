# Lecture 50 — Enum Refactoring 2

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 6. 열거형 - ENUM (7/11)

Previous: [Lecture 49 - Enum Refactoring 1](./Lecture%2049%20-%20Enum%20Refactoring%201.md)

Lecture 49 moved `discountPercent` into the hand-built `ClassGrade` class, deliberately avoiding
`enum` syntax so the idea would be easier to absorb first. This lecture applies the **exact same
refactor**, but now directly to the real `Grade` **enum**.

> "열거형도 클래스입니다." (an enum is still a class)

---

## 1. Giving the `Grade` enum its own `discountPercent` field

```java
package enumeration.ref2;

public enum Grade {
    BASIC(10), GOLD(20), DIAMOND(30);

    private final int discountPercent;

    Grade(int discountPercent) {
        this.discountPercent = discountPercent;
    }

    public int getDiscountPercent() {
        return discountPercent;
    }

}
```

Walking through each piece, and how it differs syntactically from the Lecture 49 version:

### 1.1 The semicolon after the constant list

```java
BASIC(10), GOLD(20), DIAMOND(30);
```

- As soon as anything beyond a bare constant list follows (a field, constructor, or method), the
  constant list must end with a **semicolon**.
- This tells the compiler "the enum constant declarations are finished here" — everything after
  belongs to the enum's body, not to the list of constants.
- If the enum only ever contains the bare constant names (like `Grade` in Lecture 47), this
  semicolon isn't required.

### 1.2 The constructor has no access modifier at all

```java
Grade(int discountPercent) {
    this.discountPercent = discountPercent;
}
```

- No `public`, no `private` written here — and that's correct, not an oversight.
- An enum constructor's access is **implicitly `private`** — it's not legal to write `public` or
  any other modifier on it. Writing `private` explicitly is allowed but redundant (the IDE will
  even show it as unnecessary); the lecture leaves it out entirely to match the idiomatic way
  enum constructors are written.
- Compare this to Lecture 49's `ClassGrade`, which required writing `private ClassGrade(int
  discountPercent) { ... }` explicitly — with a real `enum`, that restriction comes for free and
  doesn't need to be spelled out.

### 1.3 Calling the constructor: no `new`, just parentheses after each constant

```java
BASIC(10), GOLD(20), DIAMOND(30);
```

- Since an enum can't be constructed with `new` from outside (confirmed back in Lecture 47), there
  is also no `new Grade(10)` written here for `BASIC` either.
- Instead, the constructor call is implicit: writing `BASIC(10)` directly invokes the
  one-argument constructor for the `BASIC` constant, passing `10` as `discountPercent`. Same for
  `GOLD(20)` and `DIAMOND(30)`.
- This is the core syntactic difference from Lecture 49's version, where `ClassGrade.BASIC` had to
  be built out with an explicit `new ClassGrade(10)` call. With a real enum, that constructor call
  is folded directly into the constant declaration itself — `(10)` right after `BASIC` is enough.

### 1.4 The getter is just a normal method

```java
public int getDiscountPercent() {
    return discountPercent;
}
```

- Nothing special here — since an enum is a class, it can have ordinary instance methods exactly
  like any other class.

> "디스카운트 퍼센트 필드를 추가하고 생성자를 통해서 필드의 값을 저장합니다... 열거형을 상수로
> 지정하는 것 외에 일반적인 방법으로 생성이 불가능합니다. 따라서 생성자의 접근 제어자를 선언할 수
> 없게 막혀있습니다."

---

## 2. `DiscountService` collapses the same way as Lecture 49

```java
package enumeration.ref2;

public class DiscountService {

    public int discount(Grade grade, int price) {
        return price * grade.getDiscountPercent() / 100;
    }
}
```

- Exactly the same transformation as before: the old `if (grade == Grade.BASIC) { discountPercent
  = 10; } ...` chain is deleted entirely, replaced by directly asking `grade` for its own discount
  percentage.
- The instructor walks through the same two refactoring moves live:
  - Replace the whole `if`/`else if` chain with `grade.getDiscountPercent()`.
  - Use IntelliJ's **Inline Variable** shortcut again to fold the intermediate `discountPercent`
    local variable straight into the `return` expression, landing on the one-liner above.
- Along the way, the parameter name is also renamed from a leftover `classGrade` (copied from the
  Lecture 46/49 `ClassGrade`-based code) to the more fitting `grade`, since the type is now
  `Grade`.

### 2.1 Confirming it still works: `EnumRefMain2`

```java
package enumeration.ref2;

public class EnumRefMain2 {

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

- Identical results to every previous version of this example — again, this refactor changes
  nothing about behavior, only how the logic is organized internally.

---

## 3. The point of doing Lecture 49 first

> "처음부터 이렇게 Grade를 바로 코드를 작성하게 되면 좀 이해가 어려울 수 있거든요. 처음 enum을
> 쓰시는 분들은... 그래서 제가 Ref1번에서 이 클래스를 가지고 직접 구현했던 거에다가 이렇게 코드
> 작성한 거를 한번 리팩토링을 해봤습니다."

- The instructor is explicit that this two-step teaching order was deliberate: doing the same
  refactor first on a plain class (`ClassGrade`, Lecture 49) before doing it on a real `enum`
  (`Grade`, this lecture) makes the enum version land more easily — the underlying idea (grade
  owns its own discount rate) is identical, only the syntax changes.
- The main takeaway to carry forward: **an enum is a class**, so it's entirely natural for it to
  have its own fields, a constructor, and methods, carrying whatever data and behavior logically
  belongs to each constant — not just serve as a bare, data-less label.

---

## Summary

- Applied the exact same refactor from Lecture 49 (move `discountPercent` into the grade type
  itself) directly onto the real `Grade` enum, confirming enums support fields, constructors, and
  methods just like any other class.
- Enum-specific syntax differences from a hand-written class version:
  - the constant list must end with a `;` once anything else (fields/constructor/methods) follows
  - the constructor has **no** access modifier at all (implicitly `private`, and explicit
    modifiers aren't permitted)
  - each constant invokes its constructor via parentheses directly after its name
    (`BASIC(10)`) — there's no `new` anywhere, since enums can't be constructed externally
- `DiscountService.discount(...)` again collapses from an `if`/`else if` chain down to
  `price * grade.getDiscountPercent() / 100`, via the same Inline Variable refactor used in
  Lecture 49.
- Output is unchanged — purely an internal cleanup.
- Big picture: this lecture existed specifically to make Lecture 49's idea land naturally on a
  *real* enum, for anyone still new to `enum` syntax — the underlying design lesson (grade owns
  its own discount rate) is identical either way.
- Next lecture, "Enum Refactoring 3," continues refining this example further.
