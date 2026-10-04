# Lecture 51 — Enum Refactoring 3

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 6. 열거형 - ENUM (8/11)

Previous: [Lecture 50 - Enum Refactoring 2](./Lecture%2050%20-%20Enum%20Refactoring%202.md)

> "인간의 욕심이 끝이 없기 때문에 한번 끝까지 가봅시다."

Lecture 50 left `DiscountService` down to one line: `price * grade.getDiscountPercent() / 100`.
This lecture pushes the refactor all the way — moving the calculation itself into `Grade`,
realizing `DiscountService` becomes unnecessary, and finally removing duplication across the
calling code.

---

## 1. The remaining smell: calculation still happens *outside* the data it uses

```java
public int discount(Grade grade, int price) {
    return price * grade.getDiscountPercent() / 100;
}
```

> "이 코드를 보면 할인율 계산을 위해 Grade가 가지고 있는 데이터인 디스카운트 퍼센트의 값을 꺼내서
> 사용하죠... 이 값을 굳이 꺼내서 계산할 필요가 있을까요?"

- `discount(...)` **pulls a value out** of `grade` (`getDiscountPercent()`) and then does the
  actual math (`price * ... / 100`) somewhere else, outside the object that owns the data.
- From an object-oriented design standpoint, this is backwards:

> "객체지향 관점에서 이렇게 자신의 데이터를 외부에 노출하는 것보다는 이 Grade 클래스가 자신의
> 할인율을 어떻게 계산하는지 스스로 관리하는 것이 캡슐화 원칙에는 더 맞아요."

- The discount percentage is `Grade`'s own data — so the computation that uses it should live
  **inside** `Grade` too, not be reassembled by some other class reaching in and grabbing the raw
  number back out. This is the encapsulation principle: an object should manage its own data and
  the logic that operates on it, rather than exposing internals for others to compute with.

---

## 2. Moving the calculation into `Grade` itself

```java
package enumeration.ref3;

public enum Grade {
    BASIC(10), GOLD(20), DIAMOND(30);

    private final int discountPercent;

    Grade(int discountPercent) {
        this.discountPercent = discountPercent;
    }

    public int getDiscountPercent() {
        return discountPercent;
    }

    //추가
    public int discount(int price) {
        return price * discountPercent / 100;
    }
}
```

- A new method, `discount(int price)`, is added directly to `Grade`.
- It uses `discountPercent` — the enum's **own field** — directly, with no getter needed
  internally.
- Since `Grade` is still just a class (as emphasized repeatedly across this section), adding a
  method like this is completely ordinary.

> "Enum도 class다, 이걸 이제 강조를 한 번 더 드리고요."

### 2.1 `DiscountService` shrinks to a pure pass-through

```java
package enumeration.ref3;

public class DiscountService {

    public int discount(Grade grade, int price) {
        return grade.discount(price);
    }
}
```

- `DiscountService.discount(...)` no longer does any actual calculation — it just **delegates**
  straight to `grade.discount(price)`.
- Confirmed with `EnumRefMain3_1`, same call pattern as every earlier version, same output:

```
BASIC 등급의 할인 금액: 1000
GOLD 등급의 할인 금액: 2000
DIAMOND 등급의 할인 금액: 3000
```

---

## 3. Realizing `DiscountService` isn't needed at all

> "Grade가 스스로 할인율을 계산하죠. DiscountService 클래스는 사실 이제 더 필요가 없어요."

Since `DiscountService.discount(...)` now does nothing but forward its arguments to
`grade.discount(price)`, there's no reason to keep it in the call path at all — callers can invoke
`discount(...)` directly on the `Grade` constant itself:

```java
package enumeration.ref3;

public class EnumRefMain3_2 {

    public static void main(String[] args) {
        int price = 10000;
        System.out.println("BASIC 등급의 할인 금액: " + Grade.BASIC.discount(price));
        System.out.println("GOLD 등급의 할인 금액: " + Grade.GOLD.discount(price));
        System.out.println("DIAMOND 등급의 할인 금액: " + Grade.DIAMOND.discount(price));
    }
}
```

**Output:**

```
BASIC 등급의 할인 금액: 1000
GOLD 등급의 할인 금액: 2000
DIAMOND 등급의 할인 금액: 3000
```

> "왜냐하면 어차피 할인율에 대한 데이터도 내가 가지고 있고, 이 할인율 계산하는 로직도 Grade가
> 가지고 있는 거예요. 이게 가능한 이유는 등급별로 할인율이 결정되기 때문이에요."

- `DiscountService` no longer appears anywhere in this version — `Grade.BASIC.discount(price)`
  does everything by itself: it already knows its own rate, and now it knows how to apply that
  rate to a price too.
- The instructor is explicit that this works specifically *because* discount rate is fully
  determined by grade — if discount logic depended on anything beyond the grade itself (e.g. a
  separate promotion system), a dedicated service class would make sense again. Here, it doesn't.

### 3.1 Why `DiscountService` is kept in the repo anyway

> "디스카운트 서비스는 이제 제거해도 되지만 앞에 예제에서 사용되기 때문에 복습을 위해서 남겨둘
> 겁니다."

- The class is left in place (unused going forward) purely so earlier examples that still
  reference it keep compiling — useful for anyone reviewing earlier lectures' code side-by-side.

---

## 4. Removing duplication in the calling/printing code

Looking at `EnumRefMain3_2`, the three `println` lines are nearly identical, differing only in
which `Grade` constant is used:

```java
System.out.println("BASIC 등급의 할인 금액: " + Grade.BASIC.discount(price));
System.out.println("GOLD 등급의 할인 금액: " + Grade.GOLD.discount(price));
System.out.println("DIAMOND 등급의 할인 금액: " + Grade.DIAMOND.discount(price));
```

The instructor extracts this repeated shape into a helper method:

```java
package enumeration.ref3;

public class EnumRefMain3_3 {

    public static void main(String[] args) {
        int price = 10000;
        printDiscount(Grade.BASIC, price);
        printDiscount(Grade.GOLD, price);
        printDiscount(Grade.DIAMOND, price);
    }

    private static void printDiscount(Grade grade, int price) {
        System.out.println(grade.name() + " 등급의 할인 금액: " + grade.discount(price));
    }
}
```

**Output:**

```
BASIC 등급의 할인 금액: 1000
GOLD 등급의 할인 금액: 2000
DIAMOND 등급의 할인 금액: 3000
```

- `printDiscount(Grade grade, int price)` takes the grade and price, and handles both getting the
  constant's name and printing the discount line.
- `grade.name()` (from Lecture 48's enum key methods) supplies the label text — "BASIC", "GOLD",
  "DIAMOND" — directly from the enum constant, instead of it being typed out by hand three times.
- Each call site (`printDiscount(Grade.BASIC, price)`, etc.) is now a single short line — the
  repeated `println(...)` shape only needs to exist once.

---

## 5. Going further: looping over *all* grades with `values()`

> "새로운 등급이 추가되더라도 main() 코드의 변경 없이 모든 등급의 할인을 출력해보자."

The three `printDiscount(...)` calls in `EnumRefMain3_3` still have to be written out once per
grade — meaning adding a new grade (e.g. `VIP`) would require adding a fourth call by hand. The
final step removes even that:

```java
package enumeration.ref3;

public class EnumRefMain3_4 {

    public static void main(String[] args) {
        int price = 10000;
        Grade[] grades = Grade.values();
        for (Grade grade : grades) {
            printDiscount(grade, price);
        }
    }

    private static void printDiscount(Grade grade, int price) {
        System.out.println(grade.name() + " 등급의 할인 금액: " + grade.discount(price));
    }
}
```

**Output:**

```
BASIC 등급의 할인 금액: 1000
GOLD 등급의 할인 금액: 2000
DIAMOND 등급의 할인 금액: 3000
```

- `Grade.values()` (from Lecture 48) returns every declared constant as an array.
- Looping over that array and calling `printDiscount(grade, price)` for each one means **every**
  grade gets printed, automatically — with no reference to `BASIC`/`GOLD`/`DIAMOND` by name
  anywhere in `main()`.
- Demonstrated live: adding a new constant to `Grade` (e.g. a `VIP(40)` grade) and rerunning this
  exact same `EnumRefMain3_4` code, with **zero changes**, immediately prints the new grade's
  discount too — confirming the loop genuinely needs no maintenance as the enum grows.

> "ENUM.values()를 사용하면 ENUM 열거형의 모든 상수를 다 배열로 구할 수 있습니다. 웹 환경 같은
> 데서... 셀렉트 박스 같은 거 만들 때 이걸 값을 다 출력해서 보여줄 수가 있겠죠."

- A practical real-world use case mentioned here: generating a UI dropdown/select box (e.g. for a
  gender selector, or any fixed-choice field) directly from `values()`, so the UI automatically
  stays in sync with however many options the enum currently declares.

---

## Summary

- Moved the actual discount calculation from `DiscountService` into `Grade` itself —
  `Grade.discount(int price)` now uses its own `discountPercent` field directly, consistent with
  encapsulation (an object manages its own data *and* the logic that uses it).
- `DiscountService.discount(...)` became a trivial one-line delegate to `grade.discount(price)` —
  and once that's true, it's clear the class itself is no longer needed: callers can just invoke
  `Grade.BASIC.discount(price)` directly.
- This only works because discount rate is **fully determined by grade alone** — if more context
  were needed, a dedicated service class would still make sense.
- `DiscountService` is left in the codebase (unused by the new examples) solely so earlier
  lecture examples that reference it still compile.
- Extracted a `printDiscount(Grade grade, int price)` helper to eliminate duplicated `println`
  lines, using `grade.name()` to supply the label automatically.
- Final step: loop over `Grade.values()` instead of listing each constant by name — adding a new
  grade to the enum (e.g. `VIP(40)`) then requires **zero changes** to the printing code; the new
  grade is included automatically.
- Real-world payoff mentioned: `values()` is a natural fit for generating UI elements (e.g. a
  select/dropdown) that should always reflect the full, current set of enum options.
- This closes the discount-example refactoring arc. Next lecture moves into **practice problems
  and solutions** applying everything learned about enums so far.
