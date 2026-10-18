# Lecture 73 — Nested Classes, Inner Classes: Accessing an Outer Variable With the Same Name

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 08. 중첩 클래스, 내부 클래스1 (6/6)

Previous: [Lecture 72 - Nested Classes, Inner Classes - Inner Class in Practice](Lecture%2072%20-%20Nested%20Classes%2C%20Inner%20Classes%20-%20Inner%20Class%20in%20Practice.md)

The final lecture of this section asks a pointed question: what happens if the outer class, the
inner class, and even a local variable all happen to declare a field with the exact same name?
This is where Java's **shadowing** rules and the `this`/`OuterClass.this` qualifiers come in.

## The Example

```java
package nested;

public class ShadowingMain {

    public int value = 1;

    class Inner {
        public int value = 2;

        void go() {
            int value = 3;
            System.out.println("value = " + value);
            System.out.println("this.value = " + this.value);
            System.out.println("ShadowingMain.value = " +
                    ShadowingMain.this.value);
        }
    }

    public static void main(String[] args) {
        ShadowingMain main = new ShadowingMain();
        Inner inner = main.new Inner();
        inner.go();
    }
}
```

```
value = 3
this.value = 2
ShadowingMain.value = 1
```

- There are deliberately **three** variables all named `value`, at three different scopes:
  - `ShadowingMain.value` — the outer class's instance field (`1`).
  - `Inner.value` — the inner class's instance field (`2`).
  - the local variable `value` inside `go()` (`3`).
- `Inner` is a regular inner class, so it's created the same way covered in Lecture 71:
  `main.new Inner()`.

---

## Resolving `value` — Three Different Access Levels

### 1. Plain `value` → the local variable

```java
System.out.println("value = " + value);
```

- Prints `3`.
- When a name could refer to multiple things in scope, **the closest, most specific one wins** —
  here, that's the local variable declared directly inside `go()`.

> 프로그래밍에서 우선순위는 대부분 더 가깝거나, 더 구체적인 것이 우선권을 가진다. 쉽게 이야기해서
> 사람이 직관적으로 이해하기 쉬운 방향으로 우선순위를 설계한다. (In programming, priority usually
> goes to whatever is closer or more specific — designed to match how a person would intuitively
> read the code.)

### 2. `this.value` → the inner class's own instance field

```java
System.out.println("this.value = " + this.value);
```

- Prints `2`.
- Inside `go()`, `this` refers to the `Inner` instance the method is running on — so `this.value`
  explicitly reaches past the local variable and resolves to `Inner`'s own field.

### 3. `OuterClassName.this.value` → the outer class's instance field

```java
System.out.println("ShadowingMain.value = " +
        ShadowingMain.this.value);
```

- Prints `1`.
- `ShadowingMain.this` is special syntax meaning "the `ShadowingMain` instance that this `Inner`
  instance belongs to" — recall from Lecture 71 that every inner-class instance holds a reference
  back to the specific outer instance that created it. `OuterClassName.this` is exactly how you
  explicitly reach that stored reference when a name collision would otherwise hide it.

| Expression | Resolves to | Value |
|---|---|---|
| `value` | closest scope — the local variable in `go()` | `3` |
| `this.value` | the inner class's own instance field | `2` |
| `ShadowingMain.this.value` | the outer class's instance field | `1` |

---

## The Concept: Shadowing

> 이렇게 다른 변수들을 가려서 보이지 않게 하는 것을 섀도잉(Shadowing)이라 한다. (Hiding other
> variables like this, so they become invisible under the plain name, is called shadowing.)

- Each more-specific declaration of `value` "shadows" (hides) the less-specific ones when you
  refer to the plain name `value`.
- The local variable shadows `Inner.value`, which in turn shadows `ShadowingMain.value`.
- **Shadowing doesn't delete or destroy the hidden variables** — they still exist and still hold
  their own values. It only affects which one the plain, unqualified name resolves to.
- The qualifiers (`this.`, `OuterClassName.this.`) are exactly how you reach a variable that's
  currently being shadowed.

---

## The Instructor's Real Takeaway: Don't Rely on This

Despite walking through the mechanics in detail, the instructor's closing point is a caution
against actually using any of this in real code:

> 프로그래밍에서 가장 중요한 것은 명확성이다. 이렇게 이름이 같은 경우 처음부터 이름을 서로 다르게
> 지어서 명확하게 구분하는 것이 더 나은 방법이다. (The most important thing in programming is
> clarity. When names collide like this, it's always better to give them different names from the
> start, so they're clearly distinguished.)

- `this.value` and `OuterClassName.this.value` are useful to *understand* — you'll encounter code
  that relies on them, and shadowing can happen by accident even when it isn't intentional.
- But deliberately naming an inner-class field the same as an outer-class field, then relying on
  `this`/`OuterClassName.this` to disambiguate, makes code harder to read for no real benefit.
- The better practice: give variables distinct, descriptive names in the first place, so there is
  never a need to reach for this disambiguation syntax at all.

---

## Summary

- When an outer class, its inner class, and a local variable all declare a field with the same
  name, Java resolves the plain name using the **closest, most specific scope** — local variable
  beats inner-class field, which beats outer-class field.
- `this.fieldName` reaches the current (inner) class's own field, bypassing a shadowing local
  variable.
- `OuterClassName.this.fieldName` reaches the specific outer-class instance this inner instance
  belongs to, bypassing both the local variable and the inner class's own same-named field.
- This whole mechanism is called **shadowing**: a more specific/closer declaration hides a
  less-specific one under the same plain name, without destroying it.
- Despite being good to understand, intentionally relying on same-named fields plus
  `this`/`OuterClassName.this` to disambiguate is discouraged — clear, distinct variable names from
  the start are always the better design choice.
- This closes out Section 8 (중첩 클래스, 내부 클래스1): nested classes in general, static nested
  classes, inner classes, their practical refactoring payoffs, and now variable shadowing across
  outer/inner scopes.
