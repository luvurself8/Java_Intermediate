# Lecture 40 — Math, Random Class

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (8/11)

Previous: [Lecture 39 - System Class](./Lecture%2039%20-%20System%20Class.md)

This lecture covers two utility classes together: `Math` (mathematical operations) and `Random`
(more flexible random-value generation). It's the last "new topic" lecture in the section —
Lectures 41–42 move on to problems/solutions, and 43 is the section summary.

---

## 1. `Math` — too many methods to memorize

> "Math는 수많은 수학 문제를 해결해주는 클래스이다. 너무 많은 기능을 제공하기 때문에 대략 이런
> 것이 있구나 하는 정도면 충분하다. 실제 필요할 때 검색하거나 API 문서를 찾아보자."

- `Math` solves a huge range of math problems — far too many methods to memorize.
- The right mindset: know roughly what categories of methods exist, and look up the exact one you
  need when you actually need it (search / API docs).

### 1.1 The categories (from the slide deck)

| Category | Methods |
|---|---|
| Basic operations | `abs(x)` absolute value · `max(a, b)` largest · `min(a, b)` smallest |
| Exponents & logarithms | `exp(x)` eˣ · `log(x)` natural log · `log10(x)` log base 10 · `pow(a, b)` aᵇ |
| Rounding & precision | `ceil(x)` round up · `floor(x)` round down · `rint(x)` round to nearest int · `round(x)` round |
| Trigonometric | `sin(x)` · `cos(x)` · `tan(x)` |
| Other useful methods | `sqrt(x)` square root · `cbrt(x)` cube root · `random()` random value between 0.0 and 1.0 |

### 1.2 Trying the common ones: `MathMain`

```java
package lang.math;

public class MathMain {

    public static void main(String[] args) {
        // 기본 연산 메서드
        System.out.println("max(10, 20): " + Math.max(10, 20)); // 최대값
        System.out.println("min(10, 20): " + Math.min(10, 20)); // 최소값
        System.out.println("abs(-10): " + Math.abs(-10)); // 절대값

        // 반올림 및 정밀도 메서드
        System.out.println("ceil(2.1): " + Math.ceil(2.1)); // 올림
        System.out.println("floor(2.1): " + Math.floor(2.1)); // 내림
        System.out.println("round(2.5): " + Math.round(2.5)); // 반올림

        // 기타 유용한 메서드
        System.out.println("sqrt(4): " + Math.sqrt(4)); //제곱근
        System.out.println("random(): " + Math.random()); //0.0 ~ 1.0 사이의 double 값
    }
}
```

**Output:**

```
max(10, 20): 20
min(10, 20): 10
abs(-10): 10
ceil(2.1): 3.0
floor(2.7): 2.0
round(2.5): 3
sqrt(4): 2.0
random(): 0.0063470845922609 65
```

- `max` / `min` — straightforward, return the larger/smaller of the two arguments.
- `abs` — absolute value; `-10` and `10` both become `10`.
- `ceil` — always rounds **up**: `2.1` → `3.0`.
- `floor` — always rounds **down**, dropping the decimal part: cuts straight to the next lower
  whole number.
- `round` — standard rounding: `2.5` → `3`.
- `sqrt` — square root.
- `random()` — returns a `double` between `0.0` and `1.0`. Running it repeatedly produces a
  different value each time, confirmed by rerunning it live in the lecture.

### 1.3 A note on precision: `BigDecimal`

> "아주 정밀한 숫자와 반올림 계산이 필요하다면 BigDecimal이라는 게 있어요."

- For situations needing very precise rounding/number handling — the instructor's example is
  **money calculations**, where rounding decisions down to the exact decimal matter a lot —
  `Math`'s `double`-based methods aren't the right tool.
- `BigDecimal` is the class designed for that — worth looking up specifically when working in
  domains like financial/settlement systems where precise decimal math is required.
- Most everyday application development never needs it — treat it as "look this up later if the
  need comes up," not something to learn right now.

---

## 2. `Random` — more flexible than `Math.random()`

> "랜덤의 경우 Math.random()을 사용해도 되지만 Random 클래스를 사용하면 더욱 다양한 랜덤값을 구할
> 수 있다. 참고로 Math.random()도 내부에서는 Random 클래스를 사용한다."

- `Math.random()` only ever gives a `double` between `0.0` and `1.0`.
- `Random` (from `java.util`, **not** `java.lang`) offers a richer set of random-value methods —
  random integers, random booleans, bounded ranges, and more.
- Interesting implementation detail: `Math.random()` itself is built on top of `Random` internally
  — it's not a separate, independent mechanism.

### 2.1 `RandomMain`

```java
package lang.math;

import java.util.Random;

public class RandomMain {

    public static void main(String[] args) {
        Random random = new Random();
//        Random random = new Random(1); //seed가 같으면 Random의 결과가 같다.

        int randomInt = random.nextInt();
        System.out.println("randomInt: " + randomInt);

        double randomDouble = random.nextDouble();//0.0d ~ 1.0d
        System.out.println("randomDouble: " + randomDouble);

        boolean randomBoolean = random.nextBoolean();
        System.out.println("randomBoolean: " + randomBoolean);

        // 범위 조회
        int randomRange1 = random.nextInt(10);//0 ~ 9까지 출력
        System.out.println("0 ~ 9: " + randomRange1);

        int randomRange2 = random.nextInt(10) + 1;// 1 ~ 10까지 출력
        System.out.println("1 ~ 10: " + randomRange2);
    }
}
```

**Output (changes every run):**

```
randomInt: -1316070581
randomDouble: 0.377353421935771 15
randomBoolean: false
0 ~ 9: 5
1 ~ 10: 7
```

### 2.2 The methods, one at a time

- **`nextInt()`** — returns a random `int` value, with no bound — meaning it can come back as any
  valid `int`, positive or negative (hence the large negative number in the sample output).
- **`nextDouble()`** — returns a random `double` between `0.0` and `1.0` (same range as
  `Math.random()`).
- **`nextBoolean()`** — returns `true` or `false`, roughly 50/50 — described as "a two-sided coin
  flip."
- **`nextInt(bound)`** — returns a random number from `0` up to (but **not including**) `bound`.
  E.g. `nextInt(10)` → anywhere from `0` to `9`. E.g. `nextInt(3)` → `0`, `1`, or `2`.
- **Getting a 1-based range** — a very common real need is "give me a number from 1 to N," not
  "0 to N-1." The trick: `random.nextInt(N) + 1`.
  - `nextInt(10)` → `0..9`
  - `nextInt(10) + 1` → `1..10`
  - General pattern for a 6-sided die: `random.nextInt(6) + 1` → `1..6`.

---

## 3. Seed — making "random" reproducible

> "랜덤은 내부에서 씨드(Seed) 값을 사용해서 랜덤 값을 구한다. 그런데 이 씨드 값이 같으면 항상 같은
> 결과가 출력된다."

This is presented as the most interesting idea in the lecture.

### 3.1 The mechanism

- Internally, `Random` computes each "random" value from a running **seed** value.
- Conceptually: the seed goes through some calculation to produce the next value — and that
  *result* becomes the seed used for the calculation after it, and so on.
- If the **starting** seed is the same, every step of that chain of calculations is identical —
  so the entire sequence of "random" values produced is identical, every single time.

### 3.2 Demonstrating it

```java
Random random = new Random(1); //seed가 같으면 Random의 결과가 같다.
```

**Output (reproducible — identical on every run with seed `1`):**

```
randomInt: -1155869325
randomDouble: 0.10047321632624884
randomBoolean: false
0 ~ 9: 4
1 ~ 10: 5
```

- Passing `1` as the constructor argument fixes the seed to `1`.
- Running the program over and over with this seed produces the **exact same sequence of values**
  every time.
- Changing the seed (e.g. to `2`, or `100`) changes the sequence — but that new sequence is itself
  also fully reproducible, for that seed.

### 3.3 Two constructors, two behaviors

- **`new Random()`** (no seed argument):
  - Internally generates a seed by mixing `System.nanoTime()` with other inputs through a fairly
    involved algorithm.
  - Because the seed is effectively different every time (down to the nanosecond), results differ
    on every run — this is the "normal" unpredictable randomness most code wants.
- **`new Random(int seed)`** (seed provided):
  - Uses exactly the seed you pass in.
  - Produces **identical results** every time it's run with that same seed.
  - This means it's not useful for generating genuinely unpredictable values — but that's exactly
    the point of the next section.

### 3.4 Why a reproducible "random" is actually useful

- **Testing.** Since results aren't random when a seed is fixed, test code can assert "given seed
  X, this exact value should come out" — letting you verify random-dependent logic deterministically.
- **Minecraft world generation** (the instructor's example, from playing with his kids) — world
  terrain is generated "randomly" at world creation, but setting the **same seed** regenerates the
  exact same map every time. This is the identical seed-based mechanism at work, just applied to
  procedural world generation instead of a Java program's console output.

---

## Summary

- `Math` provides a very large set of mathematical methods (basic ops, exponents/logs, rounding,
  trig, sqrt, `random()`) — don't try to memorize them all; know the categories exist and look up
  specifics as needed.
- For precise decimal math (e.g. money), reach for `BigDecimal` instead of `Math`'s `double`-based
  methods — not covered in depth now, just a pointer for later.
- `Random` (from `java.util`) offers richer randomness than `Math.random()` — and `Math.random()`
  is actually implemented using `Random` internally.
- Key `Random` methods: `nextInt()`, `nextDouble()` (0.0–1.0), `nextBoolean()`, and
  `nextInt(bound)` (0 to bound-1) — add `+ 1` to shift a bounded range to start at 1 instead of 0.
- `Random` is driven by an internal **seed**: the same starting seed always produces the same
  sequence of "random" values.
  - `new Random()` seeds itself unpredictably (via nanosecond time + other inputs) → different
    results every run.
  - `new Random(seed)` is fully reproducible → same results every run, useful for deterministic
    tests (and, as a fun aside, for regenerating identical procedurally-generated game worlds, as
    in Minecraft).
- This closes out new material for Section 5 — next up are Lectures 41–42, **practice problems and
  solutions** applying everything from wrapper classes through `Random`.
