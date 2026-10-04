# Lecture 34 — Wrapper Class: Limit of Primitive 2

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (2/11)

This lecture follows directly from [Lecture 33](./Lecture%2033%20-%20Wrapper%20Class%20-%20Limit%20of%20Primitive%201.md) and drills into the second limitation of primitives: **a primitive always has to have some value — it can never represent "no value."**

## 1. Primitives and `null`

A primitive must always hold a value. Sometimes, though, a program genuinely needs to express the state "there is no value here" — and because primitives always have *some* value, they have no way to represent that state cleanly.

## 2. The ambiguity problem, demonstrated with plain `int`

To make this concrete, the lecture writes a small lookup function over a plain `int[]` array: given a target value, scan the array and return the matching value if found.

```java
package lang.wrapper;

public class MyIntegerNullMain0 {

    public static void main(String[] args) {
        int[] intArr = {-1, 0, 1, 2, 3};
        System.out.println(findValue(intArr, -1)); //-1
        System.out.println(findValue(intArr, 0));
        System.out.println(findValue(intArr, 1));
        System.out.println(findValue(intArr, 100)); //-1
    }

    private static int findValue(int[] intArr, int target) {
        for (int value : intArr) {
            if (value == target) {
                return value;
            }
        }
        return -1;
    }
}
```

**Output:**

```
-1
0
1
-1
```

- `findValue()` returns the matching value if it's in the array, and returns `-1` if the value isn't found.
- Because `findValue()`'s return type is `int`, it is **forced** to return *some* `int` — a primitive can never come back as "nothing." So the method has to pick some sentinel number to mean "not found," and here it (somewhat arbitrarily) picks `-1` or `0`.

**Here's the actual bug this causes:** look at the output. Calling `findValue(intArr, -1)` returns `-1` — correctly, because `-1` really is in the array. But calling `findValue(intArr, 100)` *also* returns `-1` — even though `100` isn't in the array at all! From the caller's point of view, **both calls return the exact same `-1`**, so there is no way to tell, just by looking at the return value, whether:
  - `-1` was found in the array and legitimately returned, or
  - nothing was found, and `-1` is just the "not found" sentinel.

This is precisely the ambiguity problem that falls out of limitation #2 from Lecture 33: because a primitive `int` can't represent "no value" as a distinct, unmistakable state, any sentinel value you pick (`-1`, `0`, `Integer.MIN_VALUE`, ...) always risks colliding with a real, valid piece of data.

## 3. Solving it with an object: `null` as an explicit "no value"

Objects don't have this problem, because a reference type *can* hold `null` — a distinct value that unambiguously means "no object here," separate from any real value the type can hold. Reusing the `MyInteger` wrapper class from Lecture 33, the same lookup is rewritten to work over `MyInteger[]` instead of `int[]`:

```java
package lang.wrapper;

public class MyIntegerNullMain1 {

    public static void main(String[] args) {
        MyInteger[] intArr = {new MyInteger(-1), new MyInteger(0), new MyInteger(1)};
        System.out.println(findValue(intArr, -1)); //-1
        System.out.println(findValue(intArr, 0));
        System.out.println(findValue(intArr, 1));
        System.out.println(findValue(intArr, 100)); //-1
    }

    private static MyInteger findValue(MyInteger[] intArr, int target) {
        for (MyInteger myInteger : intArr) {
            if (myInteger.getValue() == target) {
                return myInteger;
            }
        }
        return null;
    }
}
```

**Output:**

```
-1
0
1
null
```

- The return type of `findValue()` is now `MyInteger` (an object reference), not `int`. Because it's a reference type, the method can return `null` to explicitly mean "no match was found" — distinct from any real `MyInteger` value, including one that happens to wrap `-1`.
- Running it: asking for `-1` returns a `MyInteger` wrapping `-1` (printed as `-1`, thanks to the `toString()` override from Lecture 33). Asking for `100` — which genuinely isn't in the array — now returns `null`, printed as the literal word `null`.
- Compare this to the previous version: there, "found `-1`" and "not found" were indistinguishable (both printed `-1`). Here, they are **unambiguous**: a real value prints as that value, and "not found" prints as `null`. No sentinel-value collision is possible.

## 4. Takeaway: primitives vs. reference types, and a caution about `null`

- **Primitives must always have a value.** Even "nothing-like" values such as `0` or `-1` are still real, valid values — they always exist and can't be distinguished from legitimate data.
- **Reference types (objects) can use `null`** to represent the genuine absence of a value, which primitives structurally cannot do.
- This is a real, double-edged trade-off, not a free win: returning/using `null` **incorrectly** is exactly how a `NullPointerException` happens (e.g. forgetting to check for `null` before calling a method on the result of `findValue()`). So `null` should be used carefully and deliberately — it solves the ambiguity problem from this lecture, but shifts the responsibility onto the caller to check for it before using the result.

Together, Lectures 33 and 34 establish *why* Java needs wrapper classes at all: primitives can't carry their own methods (Lecture 33) and can't represent "no value" (Lecture 34). The next lecture ("자바 래퍼 클래스" / Java's own wrapper classes) moves from this hand-rolled `MyInteger` example to the real wrapper types Java provides (`Integer`, `Long`, `Double`, ...) and how to actually use them.
