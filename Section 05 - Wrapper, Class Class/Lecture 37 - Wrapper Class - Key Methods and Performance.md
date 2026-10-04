# Lecture 37 — Wrapper Class: Key Methods and Performance

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (5/11)

Previous: [Lecture 36 - Autoboxing](./Lecture%2036%20-%20Wrapper%20Class%20-%20Autoboxing.md)

This is the last lecture specifically about wrapper classes — it covers the handful of useful
methods they provide, then pivots to a real performance comparison against primitives.

---

## 1. Key methods: `WrapperUtilsMain`

```java
package lang.wrapper;

public class WrapperUtilsMain {

    public static void main(String[] args) {
        Integer i1 = Integer.valueOf(10);//숫자, 래퍼 객체 반환
        Integer i2 = Integer.valueOf("10");//문자열, 래퍼 객체 반환
        int intValue = Integer.parseInt("10");//문자열 전용, 기본형 반환

        //비교
        int compareResult = i1.compareTo(20);
        System.out.println("compareResult = " + compareResult);

        //산술 연산
        System.out.println("sum: " + Integer.sum(10, 20));
        System.out.println("min: " + Integer.min(10, 20));
        System.out.println("max: " + Integer.max(10, 20));
    }
}
```

**Output:**

```
compareResult = -1
sum: 30
min: 10
max: 20
```

### 1.1 `valueOf()` — not just for numbers

- `Integer.valueOf(10)` — the familiar form, converts a number into a wrapper object.
- `Integer.valueOf("10")` — also works, and converts a **string** into a wrapper object.
- Both return an `Integer`.

### 1.2 `parseInt()` — string-only, returns a primitive

- `Integer.parseInt("10")` is a different tool: it's string-only, and its job is specifically to
  parse a string into a value.
- Unlike `valueOf("10")`, it returns a plain primitive `int`, not a wrapper object.

### 1.3 `parseInt()` vs `valueOf()` — which one to use?

> "valueOf는 래퍼 타입을 반환하고요, parseInt는 기본형을 반환해요."

- Need a **wrapper object** back? Use `valueOf(...)`.
- Need a **primitive** back? Use `parseInt(...)`.
- This same `parseXxx()` naming pattern exists on the other wrapper types too — e.g.
  `Long.parseLong(...)`.
- Overall, the instructor notes the wrapper classes don't actually have *that* many methods worth
  memorizing — these cover most of the common cases.

### 1.4 `compareTo()` — comparing values

- `i1.compareTo(20)` compares `i1`'s own value (`10`) against the argument (`20`).
- Returns:
  - `1` if `i1`'s value is **larger**
  - `0` if **equal**
  - `-1` if `i1`'s value is **smaller**
- Here, `10.compareTo(20)` → `10 < 20` → result is `-1`. Matches the output above.
- This is the same three-way comparison idea built by hand back in Lecture 33's `MyInteger`.

### 1.5 Static utility methods: `sum`, `min`, `max`

- `Integer.sum(10, 20)`, `Integer.min(10, 20)`, `Integer.max(10, 20)` are simple `static` helper
  methods on the wrapper class itself — not instance methods.
- They do exactly what they sound like: add, find the smaller, find the larger.
- Described as "utility-style" conveniences rather than core wrapper behavior.

---

## 2. Wrapper classes vs. performance

Having now seen all these convenient methods, a natural question comes up:

> "그렇다면 더 좋은 래퍼 클래스만 제공하면 되지, 기본형을 제공하는 이유는 뭘까요?"
> (If wrapper classes are this convenient, why does Java even keep primitives around?)

Since wrapper classes are objects, they offer far more functionality than bare primitives — so why
not just use wrappers everywhere and get rid of primitives entirely? To answer this, the lecture
runs a head-to-head performance test.

### 2.1 The benchmark: `WrapperVsPrimitive`

```java
package lang.wrapper;

public class WrapperVsPrimitive {

    public static void main(String[] args) {
        int iterations = 1_000_000_000; // 반복 횟수 설정, 10억
        long startTime, endTime;

        // 기본형 long 사용
        long sumPrimitive = 0;
        startTime = System.currentTimeMillis();
        for (int i = 0; i < iterations; i++) {
            sumPrimitive += i;
        }
        endTime = System.currentTimeMillis();
        System.out.println("sumPrimitive = " + sumPrimitive);
        System.out.println("기본 자료형 long 실행 시간: " + (endTime - startTime) + "ms");

        // 래퍼 클래스 Long 사용
        Long sumWrapper = 0L;
        startTime = System.currentTimeMillis();
        for (int i = 0; i < iterations; i++) {
            sumWrapper += i; // 오토 박싱 발생
        }
        endTime = System.currentTimeMillis();
        System.out.println("sumWrapper = " + sumWrapper);
        System.out.println("래퍼 클래스 Long 실행 시간: " + (endTime - startTime) + "ms");
    }
}
```

- `1_000_000_000` — the underscore here is just a Java numeric-literal separator for readability
  (equivalent to writing `1000000000`); it has no effect on the value.
- The test sums numbers from `0` up to one billion (`iterations`), once using a primitive `long`
  accumulator, once using a `Long` wrapper accumulator.
- `sumWrapper += i` triggers **autoboxing** on every single iteration (the `Long` gets unboxed to
  add, then the result gets reboxed back into a new `Long`) — this is the exact mechanism from
  Lecture 36, just happening a billion times in a row.

**Output (instructor's M2 MacBook — will vary by machine):**

```
sumPrimitive = 499999999500000000
기본 자료형 long 실행 시간: 318ms
sumWrapper = 499999999500000000
래퍼 클래스 Long 실행 시간: 1454ms
```

- Both loops compute the exact same sum — correctness is identical.
- The primitive `long` version: ~0.3s (318ms).
- The wrapper `Long` version: ~1.5s (1454ms).
- That's roughly a **5x** slowdown using the wrapper type for this workload.
- Exact numbers vary by system — the instructor notes results can differ "완전히" (completely)
  between machines.

### 2.2 Why is the wrapper version slower?

- A primitive just occupies the raw size of its type in memory — an `int` is typically 4 bytes,
  nothing more.
- A wrapper instance is an **object**:
  - It still holds that same primitive value internally as a field (e.g. 4 bytes for the `int`
    inside an `Integer`).
  - On top of that, the JVM needs extra object metadata to manage it as an object at all.
  - Depending on JVM version/system, this adds roughly **8–16 extra bytes** per instance.
- Net effect: a wrapper object typically takes up roughly **3x–5x more memory** than the bare
  primitive it wraps.
- Wrapper objects are also **immutable** (per Lecture 35) — so every `+=` in the loop isn't
  mutating anything in place, it's creating a **new** `Long` object each time and discarding the
  old one. A primitive, by contrast, just overwrites its own 4/8 bytes directly.
- More memory per value + constant object creation + the autoboxing/unboxing overhead itself all
  add up to meaningfully slower raw arithmetic.

---

## 3. Should you actually worry about this 5x difference?

This is the part the instructor spends the most time on — and the headline warning is:

> "여기서 우리가 이제 함정에 빠지면 안 돼요." (Don't fall into a trap here.)

It's tempting to see "5x slower" and panic — but the actual scale matters enormously:

- This 5x gap was measured over **one billion** iterations.
- `0.3s / 1,000,000,000` or `1.5s / 1,000,000,000` — per individual operation, both are
  vanishingly tiny.
- Shrinking `iterations` from a billion down to 10,000 or so, the whole loop — either version —
  finishes in a fraction of a millisecond. The 5x ratio only becomes visible at huge scale.
- Modern computers are simply extremely fast at raw in-memory arithmetic.

### 3.1 When optimizing for this actually matters

- If you're doing **CPU-heavy computation** as a special case, or
- genuinely running **tens of thousands to hundreds of thousands of operations in a tight,
  continuous loop** (e.g. large batch-processing jobs),

...then switching to primitives for that hot path is worth considering.

### 3.2 When it doesn't matter (the common case)

- For a typical, everyday application, optimizing this kind of thing away gains you
  "사막의 모래알 하나" — one grain of sand in an entire desert.
- If — looking at the code — the **wrapper type actually makes the code easier to maintain**,
  choose the wrapper type. The potential performance win from switching to a primitive is simply
  too small to matter in the vast majority of real applications.

---

## 4. Maintainability vs. optimization

The instructor broadens this into general advice, since this exact "primitive vs. wrapper" choice
comes up constantly in day-to-day development:

- 20–30 years ago, when hardware was far slower, leaning toward performance in a case like this
  would have made more sense.
- Today, prioritize **readable, maintainable code first.**
- Why: modern hardware is so fast that shaving a few in-memory operations rarely produces any
  real, user-visible benefit.
- Performance optimization almost always trades away simplicity — it typically requires **more**
  code and **more** complexity, which itself becomes a maintenance burden.
- The real risk: optimizing something that *feels* impressive (e.g. "I made this 5x faster!") while
  it contributes nothing meaningful to the application's actual overall performance — i.e.
  **unnecessary optimization**.

### 4.1 Network calls matter far more than in-memory operations

- This point lands especially hard for backend/web developers specifically.
- A single network call (e.g. a database query, or a call to another server) can cost **tens of
  thousands of times** more than a single in-memory operation.
- Reducing one network round-trip is frequently far more impactful than reducing thousands —
  even millions — of in-memory operations.

### 4.2 Recommended workflow

1. Write clean, maintainable code first.
2. Only afterward, run real performance/load tests under realistic traffic.
3. Let the test results show you where the actual bottleneck is.
4. Optimize **that** specific spot — not speculatively, upfront, everywhere.

> "쓸데없는 최적화를 하면 안 된다... 성능 최적화는 이후에 실제 테스트를 해보면서."

(Experienced engineers may sometimes spot a genuine hotspot upfront from experience — but that
comes from accumulated experience and real measurement, not from guessing.)

---

## Summary

- Wrapper classes offer a small, focused set of extra methods:
  - `valueOf(...)` — number *or* string → wrapper object
  - `parseInt(...)` / `parseXxx(...)` — string → primitive, string-only
  - `compareTo(...)` — three-way comparison (`1` / `0` / `-1`)
  - `sum` / `min` / `max` — simple static arithmetic helpers
- Benchmarking primitive `long` vs. wrapper `Long` over 1 billion additions showed roughly a
  **5x slowdown** for the wrapper version (318ms vs. 1454ms on the instructor's machine).
- Root cause: wrapper objects use more memory (object metadata overhead on top of the raw value)
  and, being immutable, allocate a brand-new object on every operation — plus ongoing autoboxing
  overhead.
- That 5x gap is almost always irrelevant in real applications — it only matters at genuinely huge
  iteration counts (tens of thousands+ in a tight, continuous loop), such as large batch jobs.
- Default to whichever is more **maintainable**; only optimize toward primitives after measuring a
  real, specific bottleneck — don't optimize speculatively.
- In backend/web contexts especially, a single network round-trip costs vastly more than any of
  this in-memory overhead — that's usually a far better place to focus performance effort.
- Next lecture moves to the **`Class` class** — a different topic from wrapper types, covering
  class metadata and reflection basics.
