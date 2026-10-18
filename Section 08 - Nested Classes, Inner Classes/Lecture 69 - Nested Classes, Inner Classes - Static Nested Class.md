# Lecture 69 — Nested Classes, Inner Classes: Static Nested Class

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 08. 중첩 클래스, 내부 클래스1 (2/6)

Previous: [Lecture 68 - Nested Classes, Inner Classes - What Are They](Lecture%2068%20-%20Nested%20Classes%2C%20Inner%20Classes%20-%20What%20Are%20They.md)

The instructor calls the static nested class "the easiest one," so this lecture starts with a
single concrete code example and works out, by direct experiment, exactly what a static nested
class can and can't access.

## The Example

```java
package nested.nested;

public class NestedOuter {

    private static int outClassValue = 3;
    private int outInstanceValue = 2;

    static class Nested {
        private int nestedInstanceValue = 1;

        public void print() {
            // 자신의 멤버에 접근
            System.out.println(nestedInstanceValue);

            // 바깥 클래스의 인스턴스 멤버에 접근에는 접근할 수 없다.
            //System.out.println(outInstanceValue);

            // 바깥 클래스의 클래스 멤버에는 접근할 수 있다. private도 접근 가능
            System.out.println(NestedOuter.outClassValue);
        }
    }
}
```

```java
package nested.nested;

public class NestedOuterMain {

    public static void main(String[] args) {
        NestedOuter outer = new NestedOuter();
        NestedOuter.Nested nested = new NestedOuter.Nested();
        nested.print();

        System.out.println("nestedClass = " + nested.getClass());
    }
}
```

```
1
3
nestedClass = class nested.nested.NestedOuter$Nested
```

- `Nested` declares `static class Nested { ... }` inside `NestedOuter` — the `static` keyword
  marks it as a **static nested class**.
- The instructor builds it up member-by-member in the recording to test three specific access
  cases, each commented in the source above.

---

## What a Static Nested Class Can and Can't Access

Walking through `Nested.print()` line by line:

### 1. Its own members — obviously fine

```java
System.out.println(nestedInstanceValue);
```

- `Nested` is just a class like any other — of course it can access its own fields.

### 2. The outer class's *instance* members — not allowed

```java
//System.out.println(outInstanceValue);  // compile error if uncommented
```

- This line is commented out because it doesn't compile.
- **Why**: `Nested` is `static`, so it belongs to the class area, not to any particular instance of
  `NestedOuter`. It is similar in spirit to a `static` method — static code has no implicit
  reference to an instance, so it simply cannot "see" instance fields.
- Concretely: the static nested class has no reference at all to any `NestedOuter` instance, so
  there's no instance for it to even look up `outInstanceValue` on.

### 3. The outer class's *static* (class) members — allowed, even if `private`

```java
System.out.println(NestedOuter.outClassValue);
```

- This works, and it would work with `NestedOuter.outClassValue` even outside the class, **except**
  that `outClassValue` is `private` — normally only code inside `NestedOuter` could reach it.
- The reason the nested class can reach it anyway: `private` means "accessible only within the
  same class," and a nested class is, structurally, *inside* that same class. The private-access
  boundary is drawn around the whole outer class body, which includes anything nested within it.
- You can also write this shortened, without the `NestedOuter.` prefix — `outClassValue` alone
  resolves to the outer class's field.

> 정적 중첩 클래스도 바깥 클래스와 같은 클래스 안에 있는 거예요. 따라서 중첩 클래스는 바깥 클래스의
> private 접근 제어자에 접근할 수가 있습니다. (A static nested class is inside the same class body
> as the outer class, so it can reach the outer class's `private` members.)

---

## Creating and Using the Static Nested Class

From `NestedOuterMain`:

```java
NestedOuter outer = new NestedOuter();
NestedOuter.Nested nested = new NestedOuter.Nested();
nested.print();
```

- A static nested class is instantiated as `new OuterClass.NestedClass()`.
- Its type is referred to as `OuterClass.NestedClass` (e.g. `NestedOuter.Nested`).
- **Key observation emphasized in the recording**: the `outer` instance created on line 1 and the
  `nested` instance created on line 2 have **no relationship whatsoever**. Creating one does not
  require or depend on the other — they're simply two separate classes that happen to be declared
  in a nested structure.
  - Proof of this: the `NestedOuter outer = new NestedOuter();` line can be deleted entirely, and
    the program behaves identically. The `Nested` instance never used `outer` for anything.
- Printing `nested.getClass()` shows `nested.nested.NestedOuter$Nested` — Java represents a nested
  class's fully-qualified name as `OuterClass$NestedClass`, joined with `$`. The package
  (`nested.nested`) comes first, then the outer class, then `$`, then the nested class name.

---

## Visualizing It: Instance Independence

The instructor walks through this with a diagram of the running program's memory:

- Once both `NestedOuter` and `NestedOuter.Nested` instances exist, they sit in memory as two
  completely separate objects with no reference connecting them.
- Inside `Nested.print()`:
  - Access to `nestedInstanceValue` → fine, it's the nested instance's own field.
  - Access to `outInstanceValue` → **impossible**. The nested class holds no reference back to any
    `NestedOuter` instance, so there's no way to even locate that field, let alone read it.
  - Access to `NestedOuter.outClassValue` → fine, because static fields live in the class area
    (not inside any particular instance), and any code can reach `ClassName.staticField` the same
    way, nested or not.

---

## The Core Insight: A Static Nested Class Is Basically Just a Separate Class

This is the lecture's main takeaway, stated directly by the instructor:

> 정적 중첩 클래스는 사실 다른 클래스를 그냥 중첩해 둔 것일 뿐이에요. (A static nested class is,
> in truth, just an unrelated class that happens to be nested inside another one.)

- `NestedOuter.outClassValue`-style access to a static field works the exact same way whether or
  not `Nested` is a nested class at all — any code can reach a public static field via
  `ClassName.fieldName`.
- Conceptually, declaring the static nested class is nearly identical to just writing two
  completely separate top-level classes:

  ```java
  class NestedOuter {
  }

  class Nested {
  }
  ```

- **The one and only real difference**: because `Nested` is physically declared inside
  `NestedOuter`'s body, it falls within the same `private`-access boundary — so it can reach
  `NestedOuter`'s `private` static members, which a genuinely separate top-level class could not.
- Everything else about a static nested class is pure code organization — it's "nested" in the
  loose, structural sense from Lecture 68 (positioned inside), not "inner" in the strict sense
  (belonging to an instance).

---

## Summary

- A static nested class is declared with `static class Nested { ... }` inside an outer class.
- It can access its own members freely, just like any class.
- It **cannot** access the outer class's instance members — it has no reference to any outer
  instance, because `static` code belongs to the class area, not an instance.
- It **can** access the outer class's static members, including `private` ones, because it sits
  inside the same class body and therefore the same `private` access boundary.
- It's instantiated as `new OuterClass.NestedClass()` and typed as `OuterClass.NestedClass`.
- An instance of the outer class and an instance of its static nested class have **no relationship**
  — creating one is entirely independent of the other.
- `getClass()` on an instance shows the nested class's runtime name as `OuterClass$NestedClass`.
- Bottom line: a static nested class behaves almost exactly like two unrelated top-level classes —
  the only real benefit it adds is access to the outer class's `private` static members.
