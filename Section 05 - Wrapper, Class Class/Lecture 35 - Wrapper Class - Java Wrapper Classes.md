# Lecture 35 — Wrapper Class: Java's Own Wrapper Classes

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (3/11)

This lecture moves on from the hand-built `MyInteger` in [Lecture 33](./Lecture%2033%20-%20Wrapper%20Class%20-%20Limit%20of%20Primitive%201.md)/[Lecture 34](./Lecture%2034%20-%20Wrapper%20Class%20-%20Limit%20of%20Primitive%202.md) to the **real wrapper classes Java provides**.

## 1. Wrapper classes = the object version of a primitive

The instructor frames it simply: everything explored with `MyInteger` so far was a warm-up for understanding what Java's real wrapper classes are — and the one-line summary is: **a wrapper class is a primitive type turned into an object.** Java wraps each primitive just slightly, and ships a corresponding wrapper class for every single one:

| Primitive | Wrapper class |
|---|---|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

The naming pattern is simple for most types — just capitalize the first letter (`long` → `Long`, `double` → `Double`) — except for two: `int` → `Integer` (the full word, since "int" is itself already a shortened form of "integer"), and `char` → `Character`.

## 2. Two characteristics shared by every wrapper class

1. **Immutable.** Every one of Java's built-in wrapper classes is immutable — once created, the value inside can never change. (This mirrors the design choice made for `MyInteger` back in Lecture 33.)
2. **Must be compared with `equals()`, not `==`.** Because wrapper instances are objects, creating two separate instances that happen to hold the same number and comparing them with `==` compares *references*, not values — and that comparison will not reliably match. (The exact nuance here — because of caching, see §4 — is demonstrated below.)

The instructor also points out that `MyInteger` was deliberately designed to look and behave almost exactly like Java's real wrapper classes: a `private final` primitive field, a set of convenience methods around it, and (as shown next) a `toString()` override — real wrapper classes follow this same shape internally, just with a lot more method surface.

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

Walking through this line by line:

### 3.1 `new Integer(10)` — works, but is deprecated

`new Integer(10)` creates a wrapper object the "obvious" way, directly calling the constructor. The IDE flags this with a warning (strikethrough / "Deprecated"), but it's not a compile error — the code still runs fine. "Deprecated" specifically means: *this will be removed at some point in the future, so stop using it starting now.* Java famously prioritizes backward compatibility extremely strongly, so a deprecated API like this one is unlikely to disappear any time soon — but since it's already marked as being on its way out, there's no reason to keep writing new code that uses it. The documentation itself tells you the replacement: use `valueOf(...)` instead.

### 3.2 `Integer.valueOf(10)` — the preferred way, and why

`Integer.valueOf(10)` produces the same logical value (`10`) as `new Integer(10)`, but internally it's smarter. Looking inside `valueOf()`'s implementation (the instructor briefly steps into the source), there's a check: if the requested value falls in the range **-128 to 127** — numbers that come up extremely often in real code — Java has **already pre-created** `Integer` objects for every value in that range and simply hands back the existing cached instance, instead of allocating a new object. This is directly analogous to the **String pool** from earlier in the course: just as commonly-used string literals are pooled and reused, commonly-used small integers are pre-built and reused too. Outside that cached range, `valueOf()` falls back to calling `new Integer(...)` internally.

Because of this, `valueOf()` isn't just "the non-deprecated option" — it can also be meaningfully faster, since for cached values it completely skips object allocation. The instructor is explicit that **this caching range and strategy is an implementation detail that could change in future Java versions** (e.g. the range could be widened) — so it shouldn't be relied on as a guaranteed contract, just understood as "Java is doing some optimization here for you."

The same pattern — `valueOf(...)` instead of `new` — applies uniformly to the other wrapper types too: `Long.valueOf(100)`, `Double.valueOf(10.5)`, etc.

### 3.3 Printing a wrapper directly — `toString()` is already overridden

Printing `newInteger` or `integerObj` directly with `System.out.println` shows `10`, not something like `Integer@1b6d3586`. This is because `Integer` (like all the built-in wrapper classes) overrides `toString()` internally to render its wrapped value as text — exactly the same idea as the `toString()` override written by hand for `MyInteger` back in Lecture 33. The result is that working with a wrapper object *feels* almost exactly like working with the underlying primitive, even though it's a full object underneath.

### 3.4 Reading the value back out: `intValue()` / `longValue()`

To get the raw primitive back out of a wrapper object, call its `xxxValue()` method — `integerObj.intValue()` returns the primitive `int` stored inside, `longObj.longValue()` returns the primitive `long` inside. Peeking at `Integer`'s real source, the wrapped value is stored in a field declared as `private final int value` — i.e. exactly the same shape as the hand-rolled `MyInteger.value` field from Lecture 33. `intValue()` simply returns that field.

### 3.5 Comparing wrapper objects: `==` vs `equals()`

`newInteger` (created via `new Integer(10)`) and `integerObj` (created via `Integer.valueOf(10)`) both logically hold `10` — but they are **two distinct objects** in memory, created through two different paths. Comparing them with `==` compares object references, and since they're different objects, `newInteger == integerObj` evaluates to `false`. Comparing them with `.equals(...)` instead compares the *values* wrapped inside, which are both `10`, so `newInteger.equals(integerObj)` evaluates to `true`.

**Takeaway repeated from §2:** always use `.equals()` to compare wrapper objects for value equality — never `==`.

### 3.6 A wrinkle: the cache can make `==` "accidentally" pass

The instructor adds an important nuance by example: if *both* sides of the comparison are produced via `Integer.valueOf(10)` (rather than one via `new` and one via `valueOf`), and `10` falls inside the `-128..127` cached range, then both `valueOf(10)` calls return the **same cached object instance** — so in that specific case, `==` would actually return `true` too, simply because both variables happen to point at the identical pre-built cache entry. This is explicitly called out as not being something to rely on or treat as "important" day-to-day — it's just a byproduct of the caching optimization from §3.2, and the correct, dependable comparison is always `.equals()`, regardless of whether the values happen to be small enough to be cached.

## 4. Vocabulary: Boxing and Unboxing

Two terms are introduced here that will recur constantly for the rest of the section:

- **Boxing** — converting a primitive into its wrapper class, e.g. `new Integer(10)` or `Integer.valueOf(10)` taking the primitive `10` and putting it into an `Integer` object. The name comes from the image of putting an item into a box: the primitive number goes into the "Integer box."
- **Unboxing** — the reverse: taking the primitive value back out of a wrapper object, e.g. `integerObj.intValue()`. The name mirrors "boxing" — you're unpacking the box to get the original item back out.

(Autoboxing/auto-unboxing — the compiler doing this boxing/unboxing *automatically* without an explicit `valueOf()`/`xxxValue()` call — is the subject of the next lecture.)

## 5. Summary

- A wrapper class is simply a primitive value wrapped inside an immutable object, with one built into Java for every primitive type (`Integer`, `Long`, `Double`, `Boolean`, ...).
- Prefer `Xxx.valueOf(...)` over `new Xxx(...)` — the constructor form is deprecated, and `valueOf()` gets a free performance optimization for small, frequently-used values via internal caching (conceptually like the string pool).
- Wrapper objects print their wrapped value directly thanks to an overridden `toString()`, and expose `xxxValue()` methods to read the primitive back out.
- Always compare wrapper objects with `.equals()`; `==` compares references and can give misleading results — occasionally even "accidentally correct" ones, due to caching, which is exactly why it shouldn't be relied on.
- "Boxing" = primitive → wrapper object; "unboxing" = wrapper object → primitive. These terms set up the next lecture on **autoboxing**, where the compiler performs this conversion automatically.
