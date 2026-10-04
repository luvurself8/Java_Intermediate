# Lecture 49 — Enum Refactoring 1

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 6. 열거형 - ENUM (6/11)

Previous: [Lecture 48 - Enum Key Methods](./Lecture%2048%20-%20Enum%20Key%20Methods.md)

The mechanics of enums are done — this lecture starts actually **using** that knowledge to clean
up the discount example. Since enums themselves are still new, this first refactoring pass is done
on the hand-built `ClassGrade` from Lecture 46 (a plain class) — it's more comfortable to refactor
something already familiar before touching a real `enum`.

> "이게 뭔가 마음에 안 든단 말이에요." (something about this code doesn't sit right)

---

## 1. The itch: the discount logic doesn't belong in `DiscountService`

Looking back at `DiscountService.discount(...)` from Lecture 46:

```java
if (classGrade == ClassGrade.BASIC) {
    discountPercent = 10;
} else if (classGrade == ClassGrade.GOLD) {
    discountPercent = 20;
} else if (classGrade == ClassGrade.DIAMOND) {
    discountPercent = 30;
} else {
    System.out.println("할인X");
}
```

The key observation:

> "지금 이 코드에서 이 할인율은 각각의 회원 등급별로 판단된다... 할인율이라는 게 결국 회원의
> 등급을 따라가는 거예요."

- The discount percentage is **entirely determined by** which grade it is — `BASIC` is always
  10%, `GOLD` is always 20%, `DIAMOND` is always 30%.
- Right now, that mapping lives as `if`/`else if` logic inside `DiscountService` — a completely
  separate class from `ClassGrade` itself.
- Since the discount rate is a property *of* the grade, the grade class itself should be the one
  responsible for knowing and managing it — not an external service re-deriving it every time via
  a chain of comparisons.

---

## 2. Giving `ClassGrade` its own `discountPercent` field

```java
package enumeration.ref1;

public class ClassGrade {
    public static final ClassGrade BASIC = new ClassGrade(10); //x001
    public static final ClassGrade GOLD = new ClassGrade(20); //x002
    public static final ClassGrade DIAMOND = new ClassGrade(30); //x003

    private final int discountPercent;

    private ClassGrade(int discountPercent) {
        this.discountPercent = discountPercent;
    }

    public int getDiscountPercent() {
        return discountPercent;
    }
}
```

What changed, step by step:

- **Added a field**: `private final int discountPercent` — each `ClassGrade` instance now carries
  its own discount rate.
- **Constructor now takes a parameter**: `private ClassGrade(int discountPercent)` — still
  `private` (external code still can't construct a `ClassGrade`), but now it requires a value to
  initialize `discountPercent`.
- **Each constant passes its own rate at construction time**:
  `BASIC = new ClassGrade(10)`, `GOLD = new ClassGrade(20)`, `DIAMOND = new ClassGrade(30)`.
  - So `BASIC`'s instance holds `10` internally, `GOLD`'s holds `20`, `DIAMOND`'s holds `30` —
    fixed in at the moment each constant is created.
- **Added a getter**: `getDiscountPercent()` so other code can read the rate back out.
- **Still immutable**: `discountPercent` is `final`, and can only ever be set once, through the
  constructor — exactly the same "set once at construction, never mutate" design used throughout
  this course.

> "상수를 정의할 때 각각의 등급에 따른 할인율이 정해지는 거예요." (the discount rate for each grade
> gets fixed in at the moment the constant itself is defined)

---

## 3. The payoff: `DiscountService` collapses to one line

```java
package enumeration.ref1;

public class DiscountService {

    public int discount(ClassGrade classGrade, int price) {
        return price * classGrade.getDiscountPercent() / 100;
    }
}
```

- The entire `if`/`else if`/`else` chain is **gone**.
- `discount(...)` no longer needs to know or care whether `classGrade` is `BASIC`, `GOLD`, or
  `DIAMOND` at all — it just asks the grade object for its own discount percentage and uses it
  directly.

### 3.1 How the instructor got there live: the `if`-chain collapses step by step

- Looking at the original logic (`if (classGrade == ClassGrade.BASIC) { discountPercent = 10; }
  ...`), the realization: this whole block **is** just "the logic for computing
  `discountPercent`" — and that logic now already exists, inside `ClassGrade` itself, as
  `getDiscountPercent()`.
- So every branch of the `if`/`else if` can be replaced by one call:
  `classGrade.getDiscountPercent()`.
- From there, instead of assigning that into a local `discountPercent` variable and then using the
  variable on the next line, the two can be merged directly — demonstrated live using IntelliJ's
  **Inline Variable** refactor (`Ctrl-Alt-N` / `⌥⌘N`), collapsing
  `int discountPercent = classGrade.getDiscountPercent(); return price * discountPercent / 100;`
  straight down into the single return statement shown above.

> "가격에다가 등급의 할인율을 가지고 오고 거기에다가 100 나누면 그게 바로 할인이 되는 거죠."

### 3.2 Confirming it still works: `ClassGradeRefMain1`

```java
package enumeration.ref1;

public class ClassGradeRefMain1 {

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

- Identical results to every earlier version — only the internal implementation got simpler.

---

## 4. Why this matters: grade and discount rate were never two separate things

> "기존에는 등급과 할인율이 떨어져 있었죠. 근데 기존에 로직을 보니까 어? 아 이거 지금 두 개를
> 떨어뜨려 놓을 게 아니네. 등급에 따라서 할인율이 정해지니까 그냥 등급이 할인율을 관리하면
> 되겠네."

- The original design had "grade" and "its discount percentage" living in two unrelated places:
  the grade was just an identity (`BASIC`/`GOLD`/`DIAMOND`), and a separate `if`-chain elsewhere
  had to re-derive "what percentage does this grade mean" every single time it was needed.
- But those two concepts were never actually independent — a discount percentage **is** a property
  of a grade, one-to-one. Keeping them apart was itself the design flaw producing all that `if`-chain
  bulk.
- Merging them — letting `ClassGrade` directly carry its own `discountPercent` — eliminates the
  need for any conditional logic to rediscover the mapping; it's just data living where it
  logically belongs.

> "이렇게 하는 것만으로도 이제 두 개를 하나 합쳐놨더니 그런 복잡한 if문들을 다 제거하고 이렇게
> 깔끔하게 코드를 바꿀 수 있었던 거죠. 저는 이런 할 때마다 너무 기분이 좋더라고요."

---

## Summary

- The discount percentage was always fully determined by the grade — so it belongs **inside** the
  grade type itself, not as separate `if`/`else if` logic living in `DiscountService`.
- Refactored `ClassGrade` to carry its own `private final int discountPercent`, set once through
  its (still `private`) constructor — each constant (`BASIC(10)`, `GOLD(20)`, `DIAMOND(30)`) now
  bakes in its own rate at the moment it's defined, plus a `getDiscountPercent()` getter to read it
  back.
- `DiscountService.discount(...)` shrinks from a multi-branch `if`/`else if`/`else` chain down to
  a single expression: `price * classGrade.getDiscountPercent() / 100`.
- IntelliJ's **Inline Variable** refactor (`Ctrl-Alt-N`) is the tool used to collapse the
  intermediate `discountPercent` local variable directly into the return statement.
- Output is unchanged from every earlier version — this refactor is purely internal cleanup, not a
  behavior change.
- Underlying lesson: when a value is always fully determined by another value (grade →
  discount %), don't keep them in separate places connected by conditional logic — let the owning
  type manage its own associated data directly.
- This refactor was done on the **hand-built `ClassGrade` class** on purpose, since enums are
  still new — next lecture applies this exact same idea directly to the real `Grade` **enum**.
