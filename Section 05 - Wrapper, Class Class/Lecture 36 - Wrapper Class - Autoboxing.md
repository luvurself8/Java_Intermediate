# Lecture 36 — Wrapper Class: Autoboxing

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (4/11)

Previous: [Lecture 35 - Java Wrapper Classes](./Lecture%2035%20-%20Wrapper%20Class%20-%20Java%20Wrapper%20Classes.md)

---

## 1. Recap: boxing and unboxing, done manually

Lecture 35 ended by defining the vocabulary:

- **Boxing** = primitive → wrapper, via `Xxx.valueOf(...)`
- **Unboxing** = wrapper → primitive, via `xxxValue()`

This lecture starts by writing that out explicitly, one direction at a time.

### 1.1 Boxing: primitive → wrapper

```java
package lang.wrapper;

public class AutoboxingMain1 {

    public static void main(String[] args) {
        // Primitive -> Wrapper
        int value = 7;
        Integer boxedValue = Integer.valueOf(value);

        // Wrapper -> Primitive
        int unboxedValue = boxedValue.intValue();

        System.out.println("boxedValue = " + boxedValue);
        System.out.println("unboxedValue = " + unboxedValue);
    }
}
```

**Output:**

```
boxedValue = 7
unboxedValue = 7
```

- `Integer.valueOf(value)` takes the primitive `7` and puts it "into the box" — an `Integer` object.
- `boxedValue.intValue()` does the reverse: pulls the primitive `int` back out of the box.
- Boxing uses `valueOf()`.
- Unboxing uses `xxxValue()` — `intValue()` for `Integer`, `longValue()` for `Long`, `doubleValue()` for `Double`, and so on.

---

## 2. The problem: this conversion happens *constantly*

- In real-world code, converting primitive ↔ wrapper turns out to be extremely common.
- Developers kept having to write `valueOf(...)` / `xxxValue()` over and over, every single time
  a primitive needed to go into a wrapper-typed variable (or vice versa).
- This got tedious enough that it became a widely voiced complaint among Java developers.

> "개발자들이 오랜 기간 개발을 하다 보니까 기본형을 래퍼 클래스로 변환하거나 또는 래퍼 클래스를
> 기본으로 변환해야 되는 일이 굉장히 자주 발생하는 거예요... 많은 개발자들이 이런 부분에 대해서
> 불편함을 호소했어요."

Java's answer, introduced all the way back in **Java 1.5 (Java 5)**:

- **Autoboxing** — the compiler automatically inserts the `valueOf(...)` call for you.
- **Auto-unboxing** — the compiler automatically inserts the `xxxValue()` call for you.

This is a genuinely old feature (Java 5 dates to 2004) — it's not something new or experimental.

---

## 3. Seeing it in code: `AutoboxingMain2`

Take the exact same logic as `AutoboxingMain1`, but drop the explicit `valueOf()` / `intValue()`
calls:

```java
package lang.wrapper;

public class AutoboxingMain2 {

    public static void main(String[] args) {
        // Primitive -> Wrapper
        int value = 7;
        Integer boxedValue = value; // 오토 박싱(Auto-boxing)

        // Wrapper -> Primitive
        int unboxedValue = boxedValue; // 오토 언박싱(Auto-Unboxing)

        System.out.println("boxedValue = " + boxedValue);
        System.out.println("unboxedValue = " + unboxedValue);
    }
}
```

**Output:**

```
boxedValue = 7
unboxedValue = 7
```

Identical output to `AutoboxingMain1`. What changed is *how little code it takes*:

- `Integer boxedValue = value;` — assigning a plain `int` directly into an `Integer`-typed
  variable. This compiles, and this is autoboxing.
- `int unboxedValue = boxedValue;` — assigning an `Integer` directly into an `int`-typed variable.
  This compiles too, and this is auto-unboxing.

### 3.1 What the compiler actually does under the hood

The two programs are not just "equivalent in effect" — they are **identical after compilation**.
The compiler silently rewrites the autoboxing version back into the manual version:

```java
// What you write:
Integer boxedValue = value;              // 오토 박싱(Auto-boxing)
// What the compiler inserts at compile time:
Integer boxedValue = Integer.valueOf(value); //컴파일 단계에서 추가

// What you write:
int unboxedValue = boxedValue;            // 오토 언박싱(Auto-Unboxing)
// What the compiler inserts at compile time:
int unboxedValue = boxedValue.intValue();    //컴파일 단계에서 추가
```

- This rewriting happens **at compile time**, not at runtime — the `.class` bytecode produced for
  `AutoboxingMain1` and `AutoboxingMain2` ends up effectively the same.
- Because of this, **`AutoboxingMain1` and `AutoboxingMain2` behave completely identically** — the
  only difference is how much boilerplate the developer had to type.

---

## 4. Why Java added this to the language itself

- The conversion need was coming up so often that Java decided to bake it directly into the
  compiler, rather than leave it as a manual chore for every developer, every time.
- The instructor frames it as Java "listening" to a long-standing complaint:
  - normally, a language sticks to its principles and makes the developer write every conversion
    explicitly
  - but this particular conversion was happening *so* frequently that it was worth special-casing
    directly into the language

> "그 코드는 그냥 컴파일러가 해줄게, 그냥 너희는 이렇게만 써. 귀찮으니까 그러면 알았어, 해줄게."

- End result: converting between primitives and wrapper types became dramatically more convenient,
  since the compiler quietly handles the mechanical part.
- Autoboxing / auto-unboxing is used **extremely often** in everyday Java code — this isn't a
  rarely-used corner feature.

---

## Summary

- **Boxing** (primitive → wrapper) and **unboxing** (wrapper → primitive) can always be written
  manually with `valueOf()` / `xxxValue()`.
- **Autoboxing** / **auto-unboxing** (Java 5+) let the compiler insert those same calls
  automatically, whenever a primitive is assigned to a wrapper-typed variable or vice versa.
- `AutoboxingMain1` (manual) and `AutoboxingMain2` (automatic) produce **identical behavior**,
  because the compiler expands the automatic version into the manual version at compile time.
- This feature exists because primitive ↔ wrapper conversion was so common in practice that
  requiring it to be written out by hand every time became a real developer pain point.
- Next lecture moves on to the wrapper classes' **key methods and performance** characteristics.
