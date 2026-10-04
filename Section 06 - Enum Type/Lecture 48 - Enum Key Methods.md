# Lecture 48 — Enum Key Methods

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 6. 열거형 - ENUM (5/11)

Previous: [Lecture 47 - Enum Type](./Lecture%2047%20-%20Enum%20Type.md)

> "모든 열거형은 java.lang.Enum이라는 클래스를 자동으로 상속받아요. 따라서 해당 클래스가 제공하는
> 기능들을 사용할 수가 있습니다."

Every `enum` implicitly extends `java.lang.Enum` — you never write `extends Enum` yourself, but
the inheritance is real, and it's what gives every enum a shared set of built-in methods. This
lecture walks through them.

---

## 1. `values()` — getting every constant at once

```java
package enumeration.ex3;

import java.util.Arrays;

public class EnumMethodMain {

    public static void main(String[] args) {

        //모든 ENUM 반환
        Grade[] values = Grade.values();
        System.out.println("values = " + Arrays.toString(values));
        for (Grade value : values) {
            System.out.println("name=" + value.name() + ", ordinal=" + value.ordinal());
        }

        //String -> ENUM 변환, 잘못된 문자면 IllegalArgumentException 발생
        String input = "GOLD";
        Grade gold = Grade.valueOf(input);
        System.out.println("gold = " + gold); //toString() 오버라이딩 가능
    }
}
```

**Output:**

```
values = [BASIC, GOLD, DIAMOND]
name=BASIC, ordinal=0
name=GOLD, ordinal=1
name=DIAMOND, ordinal=2
gold = GOLD
```

- `Grade.values()` returns **every constant** of the enum, as an array (`Grade[]`).
- Printing the array directly wouldn't show its contents nicely — as with any array (see Lecture
  39's `System.arraycopy` note), `Arrays.toString(values)` is what formats it as
  `[BASIC, GOLD, DIAMOND]` instead of a raw array reference.
- This also means a plain `for` loop over `values` works naturally to visit every constant, which
  is exactly what's done next to demonstrate `name()` and `ordinal()`.

---

## 2. `name()` — the constant's own name

- `value.name()` returns the constant's name exactly as declared — `"BASIC"`, `"GOLD"`,
  `"DIAMOND"`.
- Straightforward: it's just giving back the identifier used when the enum was defined.

---

## 3. `ordinal()` — the constant's declared position

- `value.ordinal()` returns the constant's position in the declaration order, as an `int`,
  starting from **0**.
- For `Grade { BASIC, GOLD, DIAMOND }`: `BASIC` → `0`, `GOLD` → `1`, `DIAMOND` → `2`.
- Demonstrated live by reordering the declaration (moving `BASIC` to the end) and rerunning:
  the ordinals shift immediately — `GOLD` becomes `0`, `DIAMOND` becomes `1`, `BASIC` becomes `2`.
  Reverting the order restores the original `0`/`1`/`2` mapping.

### 3.1 ⚠️ Why `ordinal()` is dangerous to actually rely on

> "오디널의 값은 가급적 사용하지 않는 것이 좋아요. 여러분, 이거 전설적인 장애를 한 번씩 내는 애기
> 때문에."

This is flagged as a serious, real-world warning — not a minor style note.

- **The problem:** `ordinal()` depends entirely on *declaration order*. If a new constant is
  inserted in the middle of the list later, every constant declared after it silently shifts to a
  new ordinal value.
- **Worked example:** starting from `BASIC=0, GOLD=1, DIAMOND=2`, inserting a new `SILVER` right
  after `BASIC` produces `BASIC=0, SILVER=1, GOLD=2, DIAMOND=3` — `GOLD` and `DIAMOND` both shifted
  up by one.
- **Why this becomes catastrophic:** imagine `GOLD`'s `ordinal()` value (`1`) was being saved into
  a database or file somewhere — e.g. Member A stored as grade `1`, Member B stored as grade `2`
  (meaning `GOLD` and `DIAMOND` respectively, at the time they were saved).
- After adding `SILVER` and redeploying, the **stored values never change** (still `1` and `2` in
  the database) — but what those numbers now *mean* has changed: `1` now means `SILVER`, and `2`
  now means `GOLD`.
- **Result:** every existing `GOLD` member silently becomes a `SILVER` member overnight, and every
  `DIAMOND` member silently becomes `GOLD` — purely because of where a brand-new constant happened
  to be inserted in the source file, with zero code changes to any business logic.

> "쉽게 얘기해서 오디널의 값을 사용하면 기존 골드 회원이 갑자기 실버가 되는 무시무시한 버그가
> 발생할 수 있습니다."

**Takeaway:** `ordinal()` is fine to use transiently (e.g. for display, or logic entirely within a
single running program), but it should essentially never be persisted (database, file, external
API) as a stable identifier for an enum value — any future reordering/insertion silently corrupts
the meaning of previously-stored data.

---

## 4. `valueOf(String)` — converting a string back into an enum constant

```java
String input = "GOLD";
Grade gold = Grade.valueOf(input);
System.out.println("gold = " + gold); //toString() 오버라이딩 가능
```

**Output:**

```
gold = GOLD
```

- `Grade.valueOf("GOLD")` looks up the enum constant whose name **exactly matches** the given
  string, and returns it.
- This is the clean, safe on-ramp from "a string came in from the outside world" (user input, a
  config file, a database column, etc.) to an actual, type-safe `Grade` constant.
- Printing `gold` shows `GOLD` because of the automatic `toString()` override from Lecture 47 —
  the instructor notes this can be overridden further if a different display format is ever
  needed.

### 4.1 What happens with an invalid string

> "참고로 잘못된 문자가 들어가면... IllegalArgumentException 문자면 이거 예외가 발생합니다."

- If the string doesn't exactly match any declared constant name (wrong case, typo, nonexistent
  value — exactly the failure modes from Lectures 44–45), `valueOf(...)` throws an
  **`IllegalArgumentException`** at that point, immediately.
- This is actually a meaningful safety property: unlike the original `String`-based approach that
  silently fell through to "no discount," an invalid input here causes a loud, immediate failure —
  much easier to detect and debug than data quietly computing to zero.

---

## 5. The enum method reference table

| Method | What it returns |
|---|---|
| `values()`          | an array containing every enum constant |
| `valueOf(String name)` | the enum constant whose name exactly matches `name` (throws `IllegalArgumentException` if none matches) |
| `name()`             | the constant's name, as declared, as a `String` |
| `ordinal()`          | the constant's declaration position, starting from `0` — **avoid relying on this for anything persisted** |
| `toString()`         | the constant's name as a `String` — similar to `name()`, but **can be overridden** (unlike `name()`, which cannot) |

- `name()` vs `toString()`: both return the constant's name by default, but only `toString()` can
  be customized with your own override. `name()` is fixed/final — it always returns exactly the
  declared identifier.

---

## 6. A few extra facts about enums (recap + new)

> "열거형은 java.lang.Enum을 자동(강제)으로 상속 받는다."

- **Enums can't extend anything else.** Since Java doesn't support multiple inheritance of
  classes, and an enum already (implicitly) extends `java.lang.Enum`, it cannot `extends` any
  other class.
- **Enums *can* implement interfaces.** No conflict there — implementing an interface is
  independent of the single-inheritance restriction above.
- **Enums can declare and implement abstract methods**, with each constant providing its own
  implementation — this uses a mechanism similar to **anonymous classes**. The instructor
  explicitly defers this: anonymous classes haven't been covered yet in the course, so this is
  flagged as "this is possible" without demonstrating it here — it'll make more sense once
  anonymous classes are covered later.

---

## Summary

- Every enum automatically extends `java.lang.Enum`, inheriting a shared set of useful methods.
- `values()` returns every constant as an array; combine with `Arrays.toString(...)` (or a
  `for` loop) to actually see/use the contents.
- `name()` returns the constant's declared name; `ordinal()` returns its declaration-order position
  starting at `0`.
- **`ordinal()` is dangerous to persist anywhere** (database, file, external system) — inserting a
  new constant anywhere but the very end silently shifts every later constant's ordinal, which can
  silently corrupt previously-stored data (the "GOLD members become SILVER overnight" scenario).
- `valueOf(String)` converts a matching string into its enum constant, and throws
  `IllegalArgumentException` immediately for anything that doesn't match exactly — a loud failure,
  which is actually safer than the silent-failure behavior of the original `String`-based approach.
- `toString()` behaves like `name()` by default but, unlike `name()`, can be overridden.
- Enums can't extend another class (since they already extend `Enum`), but they can implement
  interfaces, and can even declare/implement abstract methods per-constant (via a mechanism similar
  to anonymous classes — covered properly later in the course).
- This closes out the core mechanics of `enum`. Next lectures move into **refactoring** the
  existing discount example to actually take advantage of enums properly.
