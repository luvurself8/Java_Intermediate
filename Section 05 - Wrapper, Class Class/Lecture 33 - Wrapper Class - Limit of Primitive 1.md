# Lecture 33 — Wrapper Class: Limit of Primitive 1

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스

### The limits of primitive types

Java is an object-oriented language, yet it still has primitive types (`int`, `double`, ...) that are *not* objects. Because they aren't objects, primitives have two key limits:

- **Not an object**: a primitive can't provide methods of its own, and can't be used wherever an object reference is required — e.g. collection frameworks and generics are built around object references, so they can't hold primitives directly.
- **Can't be `null`**: a primitive always has a value. Sometimes code needs to express "no value," but a primitive type has no way to represent that.

### Example: comparing with a plain primitive method

To see the first limit in action, consider a simple comparison that returns `-1`/`0`/`1` (smaller / equal / larger), implemented as a static method that takes primitives:

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

Here, `compareTo` is an *external* method that takes `value` as a parameter — but since the comparison is really about `value`'s own relationship to another value, it would be more natural for `value` itself to carry that behavior. The problem is `int` is a primitive, so it can't have methods of its own.

### Building a wrapper class by hand

`int` can't be turned into a class, but we can create a class that holds an `int` value and adds useful methods around it — effectively wrapping (hence "wrapper") the primitive inside an object:

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

- `MyInteger` wraps a single primitive `int value`.
- It exposes convenient methods for working with that value — here, `compareTo()` moves from being an external helper to a method the object owns.
- The class is designed to be immutable (`value` is `final`, with no setter).

Using it:

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

- `myInteger.compareTo()` now compares its own value against an external value — the comparison belongs to the object itself.
- `MyInteger` is an object, so it can conveniently call its own methods.
- Plain `int`, being a primitive, cannot have methods of its own — this is exactly the limit a wrapper class works around.

### Takeaway

A primitive like `int` is just a bare value with no behavior and no way to be "absent." Wrapping it inside a class (as `MyInteger` does) turns it into an object that can carry its own methods — this is the core idea behind Java's wrapper classes, explored further in the next lecture.
