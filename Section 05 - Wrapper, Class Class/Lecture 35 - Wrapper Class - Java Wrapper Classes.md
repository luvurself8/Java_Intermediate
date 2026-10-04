# Lecture 35 — Wrapper Class: Java's Own Wrapper Classes

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (3/11)

This lecture moves on from the hand-built `MyInteger` in
[Lecture 33](./Lecture%2033%20-%20Wrapper%20Class%20-%20Limit%20of%20Primitive%201.md) /
[Lecture 34](./Lecture%2034%20-%20Wrapper%20Class%20-%20Limit%20of%20Primitive%202.md)
to the **real wrapper classes Java provides**.

---

## 1. Wrapper classes = the object version of a primitive

One-line summary from the instructor:

> A wrapper class is a primitive type turned into an object.

Java wraps every primitive just slightly, and ships a matching wrapper class for each one:

| Primitive | Wrapper class |
|---|---|
| `byte`    | `Byte`      |
| `short`   | `Short`     |
| `int`     | `Integer`   |
| `long`    | `Long`      |
| `float`   | `Float`     |
| `double`  | `Double`    |
| `char`    | `Character` |
| `boolean` | `Boolean`   |

Naming pattern:

- Mostly: just capitalize the first letter (`long` → `Long`, `double` → `Double`).
- Two exceptions:
  - `int` → `Integer` (the full word — "int" is itself short for "integer").
  - `char` → `Character`.

---

## 2. Two characteristics shared by every wrapper class

1. **Immutable.**
   Once created, the value inside a wrapper object can never change.
   (Same design choice made for `MyInteger` in Lecture 33.)

2. **Must be compared with `equals()`, not `==`.**
   Wrapper instances are objects.
   Two separate instances holding the same number are still two different references.
   `==` compares references, not values — see the caching wrinkle in §3.6 for the one case where this is easy to get wrong.

`MyInteger` was deliberately built to mirror this shape:

- a `private final` primitive field
- convenience methods around it
- a `toString()` override (see §3.3)

Java's real wrapper classes follow the exact same shape internally — just with far more methods.

---

## 3. Creating and using wrapper objects: `WrapperClassMain`

```java
package lang.wrapper;

public class WrapperClassMain {

    public static void main(String[] args) {
        Integer newInteger = new Integer(10); //미래에 삭제 예정, 대신에 valueOf()를 사용
        Integer integerObj = Integer.valueOf(10); //-128 ~ 127 자주 사용하는 숫자 값 재사용, 불변
        Long longObj = Long.valueOf(100);
        Double doubleObj = Double.valueOf(10.5);

        System.out.println("newInteger = " + newInteger);
        System.out.println("integerObj = " + integerObj);
        System.out.println("longObj = " + longObj);
        System.out.println("doubleObj = " + doubleObj);

        System.out.println("내부 값 읽기");
        int intValue = integerObj.intValue();
        System.out.println("intValue = " + intValue);
        long longValue = longObj.longValue();
        System.out.println("longValue = " + longValue);

        System.out.println("비교");
        System.out.println("==: " + (newInteger == integerObj));
        System.out.println("equals: " + (newInteger.equals(integerObj)));
    }
}
```

**Output:**

```
newInteger = 10
integerObj = 10
longObj = 100
doubleObj = 10.5
내부 값 읽기
intValue = 10
longValue = 100
비교
==: false
equals: true
```

### 3.1 `new Integer(10)` — works, but is deprecated

- Creates a wrapper object the "obvious" way: calling the constructor directly.
- The IDE flags it (strikethrough / "Deprecated").
- Not a compile error — the code still runs fine.
- "Deprecated" means: *this will be removed eventually, stop using it starting now.*
- Java prioritizes backward compatibility extremely strongly, so this isn't disappearing soon — but there's no reason to write new code that still uses it.
- The documentation tells you the replacement directly: use `valueOf(...)`.

### 3.2 `Integer.valueOf(10)` — the preferred way, and why

- Produces the same logical value (`10`) as `new Integer(10)`, but is internally smarter.
- Looking inside `valueOf()`'s implementation: it checks whether the value falls in the range **-128 to 127**.
- Numbers in that range come up extremely often in real code.
- Java has already **pre-created** `Integer` objects for every value in that range.
- If the value is in range, `valueOf()` just hands back the existing cached instance — no new object is allocated.
- This is directly analogous to the **String pool** from earlier in the course: common values are pooled and reused.
- Outside the cached range, `valueOf()` falls back to calling `new Integer(...)` internally.

Because of this:

- `valueOf()` isn't just "the non-deprecated option."
- It can be meaningfully faster for cached values, since it skips allocation entirely.

One caveat, stated explicitly by the instructor:

- This caching range/strategy is an **implementation detail**.
- It could change in future Java versions (e.g. a wider range).
- Don't treat it as a guaranteed contract — just as "Java optimizing this for you."

Same pattern applies to the other wrapper types too:

- `Long.valueOf(100)`
- `Double.valueOf(10.5)`
- ...and so on.

### 3.3 Printing a wrapper directly — `toString()` is already overridden

- Printing `newInteger` or `integerObj` directly shows `10`.
- Not something like `Integer@1b6d3586`.
- Why: `Integer` (like every built-in wrapper class) overrides `toString()` to render its wrapped value as text.
- Same idea as the hand-written `toString()` override on `MyInteger` in Lecture 33.
- Result: working with a wrapper object *feels* almost exactly like working with the underlying primitive — even though it's a full object underneath.

### 3.4 Reading the value back out: `intValue()` / `longValue()`

- `integerObj.intValue()` returns the primitive `int` stored inside.
- `longObj.longValue()` returns the primitive `long` stored inside.
- Peeking at `Integer`'s real source: the wrapped value lives in a field declared as `private final int value`.
- That's the exact same shape as `MyInteger.value` from Lecture 33.
- `intValue()` simply returns that field.

### 3.5 Comparing wrapper objects: `==` vs `equals()`

- `newInteger` (via `new Integer(10)`) and `integerObj` (via `Integer.valueOf(10)`) both logically hold `10`.
- But they are **two distinct objects**, created through two different paths.
- `newInteger == integerObj` → `false` (different references).
- `newInteger.equals(integerObj)` → `true` (same wrapped value).

**Takeaway (repeated from §2):** always use `.equals()` to compare wrapper objects for value equality — never `==`.

### 3.6 A wrinkle: the cache can make `==` "accidentally" pass

- If *both* sides are produced via `Integer.valueOf(10)` (instead of one via `new` and one via `valueOf`)...
- ...and `10` falls inside the `-128..127` cached range...
- ...then both `valueOf(10)` calls return the **same cached object instance**.
- In that specific case, `==` would actually return `true` too — simply because both variables point at the identical pre-built cache entry.

This is explicitly called out as:

- Not something to rely on, or treat as important day-to-day.
- Just a side effect of the caching optimization from §3.2.
- The correct, dependable comparison is always `.equals()` — regardless of whether the values happen to be cached.

---

## 4. Vocabulary: Boxing and Unboxing

Two terms introduced here recur for the rest of the section:

- **Boxing** — converting a primitive into its wrapper class.
  - E.g. `new Integer(10)` or `Integer.valueOf(10)`: the primitive `10` goes into an `Integer` object.
  - Name comes from the image of putting an item into a box.

- **Unboxing** — the reverse: taking the primitive value back out of a wrapper object.
  - E.g. `integerObj.intValue()`.
  - Name mirrors "boxing" — unpacking the box to get the original item back out.

(Autoboxing / auto-unboxing — the compiler doing this *automatically*, without an explicit `valueOf()` / `xxxValue()` call — is the subject of the next lecture.)

---

## 5. Summary

- A wrapper class is a primitive value wrapped inside an immutable object.
- Java ships one for every primitive: `Integer`, `Long`, `Double`, `Boolean`, ...
- Prefer `Xxx.valueOf(...)` over `new Xxx(...)`:
  - the constructor form is deprecated
  - `valueOf()` gets a free performance optimization for small, frequently-used values via internal caching (like the string pool)
- Wrapper objects print their wrapped value directly (overridden `toString()`), and expose `xxxValue()` methods to read the primitive back out.
- Always compare wrapper objects with `.equals()`.
  - `==` compares references and can mislead.
  - It can even look "accidentally correct" due to caching — which is exactly why it shouldn't be relied on.
- Vocabulary going forward:
  - **Boxing** = primitive → wrapper object
  - **Unboxing** = wrapper object → primitive
  - Sets up the next lecture: **autoboxing**, where the compiler performs this conversion automatically.
