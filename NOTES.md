# Java Intermediate 1 — Study Notes

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308)

Personal notes taken while working through the lectures. Appended one section per lecture.

---

## Lecture 33 — Wrapper Class

### What is a wrapper class?

Java's primitive types (`int`, `long`, `double`, `boolean`, etc.) are not objects — they can't be used where an object is required, such as in generics (`List<Integer>` but not `List<int>`) or whenever a value needs to be `null`. Wrapper classes (`Integer`, `Long`, `Double`, `Boolean`, ...) solve this by wrapping a primitive value inside an object, so primitives can be treated as first-class objects when needed.

### Creating a wrapper object

```java
Integer newInteger = new Integer(10);     // deprecated — avoid
Integer integerObj = Integer.valueOf(10); // preferred factory method
Long longObj = Long.valueOf(100);
Double doubleObj = Double.valueOf(10.5);
```

`new Integer(...)` is deprecated in modern Java. `valueOf(...)` is the preferred way to obtain a wrapper instance, because it can reuse cached instances instead of always allocating a new object.

### The `Integer` cache and `==` vs `equals()`

`Integer.valueOf(...)` caches and reuses instances for the commonly used range **-128 to 127**. This means two `valueOf` calls in that range can return the *same* object, while values outside the range (or objects created with `new`) are distinct objects. This makes `==` unreliable for comparing wrapper values:

```java
Integer newInteger = new Integer(10);
Integer integerObj = Integer.valueOf(10);

newInteger == integerObj;        // false — different object references
newInteger.equals(integerObj);   // true  — same underlying value
```

**Takeaway:** always compare wrapper objects with `.equals()`, never `==`, since `==` compares object identity and depends on caching behavior that's easy to get wrong.

### Unwrapping (reading the primitive value back out)

Each wrapper exposes a `xxxValue()` method to extract the primitive value it holds:

```java
int intValue = integerObj.intValue();
long longValue = longObj.longValue();
```

### Autoboxing and auto-unboxing

Since Java 5, the compiler can automatically convert between a primitive and its wrapper, so the manual `valueOf`/`xxxValue()` calls above aren't strictly necessary:

```java
int value = 7;
Integer boxedValue = value;        // autoboxing: int -> Integer
int unboxedValue = boxedValue;     // auto-unboxing: Integer -> int
```

This is just syntactic sugar — under the hood the compiler still inserts the `valueOf`/`intValue()` calls for you.

### Performance cost of boxing

Because wrapper types are objects, using them instead of primitives in performance-sensitive code (e.g. tight loops) causes repeated autoboxing/unboxing and extra object overhead. Benchmarking a loop that sums 1 billion numbers with a primitive `long` vs. a wrapper `Long` accumulator shows the wrapper version is noticeably slower, since every `+=` on the `Long` triggers unboxing, addition, and reboxing into a new `Long` object.

**Takeaway:** prefer primitives for hot loops and bulk numeric work; reach for wrapper types when you actually need object semantics (nullability, generics, collections).

### Useful wrapper utility methods

```java
Integer i1 = Integer.valueOf(10);     // number -> wrapper object
Integer i2 = Integer.valueOf("10");   // string -> wrapper object
int intValue = Integer.parseInt("10"); // string -> primitive (parsing only)

i1.compareTo(20);        // comparison, like Comparable
Integer.sum(10, 20);     // arithmetic helpers
Integer.min(10, 20);
Integer.max(10, 20);
```

`valueOf(String)` returns a wrapper *object*, while `parseInt(String)` returns a raw primitive — pick based on whether you need an object or not.

### Why wrapper types matter: representing "no value"

Primitives always have a value — `int` can't be "absent," it defaults to `0`. Wrapper types can be `null`, which lets them represent the concept of "no value" cleanly. For example, a lookup method can return `null` when nothing matches, instead of having to pick a sentinel primitive value (like `-1`) that might collide with a real value:

```java
private static MyInteger findValue(MyInteger[] intArr, int target) {
    for (MyInteger myInteger : intArr) {
        if (myInteger.getValue() == target) {
            return myInteger;
        }
    }
    return null; // not possible with a plain int return type
}
```

### Summary

Wrapper classes turn primitives into objects, enabling generics, collections, and nullability, at the cost of extra memory/object overhead and some comparison gotchas (`==` vs `equals()`, the -128~127 cache). Use autoboxing/unboxing for convenience in everyday code, but fall back to primitives when performance in tight loops matters.
