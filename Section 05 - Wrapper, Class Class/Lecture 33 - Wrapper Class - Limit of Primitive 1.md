# Lecture 33 — Wrapper Class: Limit of Primitive 1

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (1/11)

## 1. The limits of primitive types

Java is an object-oriented language, but it still keeps primitive types (`int`, `long`, `double`, `boolean`, ...) around for performance. These are **not objects**, and not being objects creates two concrete limitations:

### 1.1 A primitive is not an object

Because a primitive value isn't an object, it can't take advantage of anything object-oriented programming offers:

- **No methods of its own.** An object can expose useful methods on itself (e.g. `"hello".length()`), but a primitive like `int` cannot — there is no `int` type to attach a method to, it's just a raw bit pattern in memory.
- **Can't be used where an object reference is required.** Collection frameworks (`List`, `Map`, `Set`, ...) and generics (`List<T>`) are built entirely around object references — internally they store and pass around references to `Object`. A primitive has no reference form, so you cannot write `List<int>` — only `List<Integer>` compiles. (We'll see exactly how wrapper types bridge this gap in a later lecture on generics.)

### 1.2 A primitive cannot be `null`

A primitive variable always holds *some* value — `int` defaults to `0`, `boolean` to `false`, and so on. There's no bit pattern that means "no value." But real programs frequently need to express the concept of "nothing here" / "not found" / "not set yet" — and primitives simply have no way to represent that state. This becomes very concrete in the null-handling example in the next lecture.

## 2. Seeing the first limit in code: a comparison without object methods

To make the "no methods of its own" limitation tangible, the lecture builds a simple three-way comparison (`-1` smaller, `0` equal, `1` larger) as a **static helper method** that takes two primitive `int`s:

```java
package lang.wrapper;

public class MyIntegerMethodMain0 {

    public static void main(String[] args) {
        int value = 10;
        int i1 = compareTo(value, 5);
        int i2 = compareTo(value, 10);
        int i3 = compareTo(value, 20);
        System.out.println("i1 = " + i1);
        System.out.println("i2 = " + i2);
        System.out.println("i3 = " + i3);
    }

    public static int compareTo(int value, int target) {
        if (value < target) {
            return -1;
        } else if (value > target) {
            return 1;
        } else {
            return 0;
        }
    }
}
```

**Output:**

```
i1 = 1
i2 = 0
i3 = -1
```

The important observation here is *not* the arithmetic — it's the shape of the code. `compareTo(value, target)` is an **external** method: it has to be handed `value` as a parameter in order to compare it against `target`. Conceptually, though, the comparison is about `value`'s own relationship to another number — it would read far more naturally as `value`.compareTo(target)`, i.e. as a method `value` calls on itself. That's impossible here, because `int` is a primitive and primitives can't own methods. This gap is exactly what motivates building a wrapper class.

## 3. Building a wrapper class by hand

Since `int` itself can't be turned into a class, the lecture instead builds a *new* class that **wraps** a single `int` value inside an object and layers useful methods on top of it — this wrapping idea is literally where the term "wrapper class" comes from.

```java
package lang.wrapper;

public class MyInteger {

    private final int value;

    public MyInteger(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public int compareTo(int target) {
        if (value < target) {
            return -1;
        } else if (value > target) {
            return 1;
        } else {
            return 0;
        }
    }

    @Override
    public String toString() {
        return String.valueOf(value); //숫자를 문자로 변경
    }
}
```

Breaking down the design choices:

- **`private final int value`** — `MyInteger` holds exactly one piece of data: a plain `int`. There's no setter and the field is `final`, so once a `MyInteger` is created its value can never change. This is a deliberate **immutable** design (the course emphasizes immutability elsewhere too — objects that can't change state after construction are easier to reason about and safe to share).
- **Convenience methods around that value** — `getValue()` exposes the raw primitive back out, and `compareTo(int target)` re-implements the exact same three-way comparison as before, but now as an *instance method* that belongs to the object.
- **`toString()` override** — printing a `MyInteger` object directly (e.g. via `System.out.println`) would otherwise print something like `lang.wrapper.MyInteger@1b6d3586` (the default `Object.toString()`). Overriding `toString()` to return `String.valueOf(value)` makes the object print as its underlying number instead, which matters for the null-handling example coming up in the next lecture.

Using the class:

```java
package lang.wrapper;

public class MyIntegerMethodMain1 {

    public static void main(String[] args) {
        MyInteger myInteger = new MyInteger(10);
        int i1 = myInteger.compareTo(5);
        int i2 = myInteger.compareTo(10);
        int i3 = myInteger.compareTo(20);
        System.out.println("i1 = " + i1);
        System.out.println("i2 = " + i2);
        System.out.println("i3 = " + i3);
    }
}
```

**Output:**

```
i1 = 1
i2 = 0
i3 = -1
```

Same result as `MyIntegerMethodMain0`, but notice the difference in *how* it's called:

- `myInteger.compareTo(5)` — the comparison is now a method call **on** the object itself, comparing its own internal value against an external one. This reads naturally, the same way `"abc".compareTo("abd")` does for `String`.
- This is only possible because `MyInteger` **is** an object — it can conveniently own and call its own methods.
- By contrast, plain `int` — being a primitive — cannot have methods of its own; `compareTo()` had to live outside of it as a static helper.

## 4. Takeaway

A primitive like `int` is nothing more than a bare value sitting in memory: no behavior attached, and no way to represent "no value." Wrapping that primitive inside a class — as `MyInteger` does here — turns it into a full object that can carry its own methods (like `compareTo()`) and be designed deliberately (here, as immutable). This hand-built `MyInteger` is a simplified stand-in for what Java's real `Integer` wrapper class does. The next lecture ("Limit of Primitive 2") tackles the second limitation head-on — the inability of primitives to represent "no value" — by extending this same `MyInteger` example to show what `null` gives you that a primitive never can.
