# Lecture 79 — Nested Classes, Inner Classes 2: Anonymous Class Usage 1

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 09. 중첩 클래스, 내부 클래스2 (6/11)

Previous: [Lecture 78 - Anonymous Class - Getting Started](Lecture%2078%20-%20Anonymous%20Class%20-%20Getting%20Started.md)

Before showing what anonymous classes are actually useful for, this lecture steps away from
nested classes entirely to work through a deliberately simple refactoring exercise. The instructor
poses it as a hands-on challenge — pause and solve it yourself first — because the principle it
teaches (separating what varies from what doesn't) is exactly what makes anonymous classes
valuable in the next lecture.

## Before: Two Nearly-Identical Methods

```java
package nested.anonymous.ex;

public class Ex0Main {

    public static void helloJava() {
        System.out.println("프로그램 시작");
        System.out.println("Hello Java");
        System.out.println("프로그램 종료");
    }

    public static void helloSpring() {
        System.out.println("프로그램 시작");
        System.out.println("Hello Spring");
        System.out.println("프로그램 종료");
    }

    public static void main(String[] args) {
        helloJava();
        helloSpring();
    }
}
```

```
프로그램 시작
Hello Java
프로그램 종료
프로그램 시작
Hello Spring
프로그램 종료
```

- `helloJava()` and `helloSpring()` are identical except for one line in the middle — the string
  that gets printed.
- The instructor's framing: pause here and try to refactor this yourself, merging the two methods
  into one, before continuing.

---

## Finding the Dividing Line: What Varies, What Doesn't

Looking at the two methods side by side exposes exactly one line that differs:

```java
public static void helloJava() {
    System.out.println("프로그램 시작"); //변하지 않는 부분
    System.out.println("Hello Java");   //변하는 부분
    System.out.println("프로그램 종료"); //변하지 않는 부분
}

public static void helloSpring() {
    System.out.println("프로그램 시작"); //변하지 않는 부분
    System.out.println("Hello Spring"); //변하는 부분
    System.out.println("프로그램 종료"); //변하지 않는 부분
}
```

> 여기서 핵심은 변하는 부분과 변하지 않는 부분을 분리하는 것이다. 변하는 부분은 그대로 유지하고 변하지
> 않는 부분을 어떻게 해결할 것인가에 집중하면 된다. (The key here is separating what varies from
> what doesn't. Keep the varying part as-is, and focus on how to solve the non-varying part.)

- The "start" and "end" print statements never change between the two methods — they're the
  **unchanging** part.
- Only the middle string (`"Hello Java"` vs. `"Hello Spring"`) changes — it's the **varying**
  part.

---

## After: One Method, Parameterized on What Varies

```java
package nested.anonymous.ex;

public class Ex0RefMain {

    public static void hello(String str) {
        System.out.println("프로그램 시작"); //변하지 않는 부분
        System.out.println(str); //str: 변하는 부분
        System.out.println("프로그램 종료"); //변하지 않는 부분
    }

    public static void main(String[] args) {
        hello("Hello Java");
        hello("Hello Spring");
    }
}
```

```
프로그램 시작
Hello Java
프로그램 종료
프로그램 시작
Hello Spring
프로그램 종료
```

- `hello(String str)` keeps both unchanging `System.out.println` calls exactly as they were.
- The one thing that varied — the middle string — is pulled out as a method **parameter**, `str`,
  and supplied by the caller instead of being hardcoded inside the method body.
- The result behaves identically to calling `helloJava()`/`helloSpring()`, but is now a single,
  reusable method.

---

## Why This Matters: The Root Cause of the Duplication

> 기존에는 이거를 재사용할 수 없었던 이유는 뭘까요? 얘들은 지금 이 변하는 부분이 안에서 들고 있잖아요.
> (Why couldn't the original methods be reused? Because they each held the varying part *inside*
> themselves.)

- `helloJava()` and `helloSpring()` couldn't be reused because each one **hardcoded** its own
  varying piece internally — calling `helloJava()` would always print `"Hello Java"`, no matter
  what. There was no way to ask it for something different.
- Moving the varying part to a parameter, supplied from the **outside** at call time, is what
  actually unlocks reuse — the method itself no longer decides what string to print; the caller
  does.

> 이렇게 변하는 부분과 변하지 않는 부분을 분리하고, 변하는 부분을 외부에서 전달받으면, 메서드(함수)의
> 재사용성을 높일 수 있다. (By separating what varies from what doesn't, and having the varying
> part supplied from outside, you increase the reusability of the method.)

- This is presented as a general principle, not just a trick for this specific example: **removing
  duplication and writing good code both come down to identifying what varies and what doesn't**,
  then making sure the varying part is handed in from outside rather than baked into the method.

---

## Setting Up the Next Lecture

The instructor is explicit that this example was deliberately simple — here, the "varying part"
was just a `String` value, something trivially passed as a parameter.

> 여기는 좀 쉬운 예제였고요, 이제 이번에는 조금 어려운 예제로 다음 시간에는 넘어가보겠습니다. (This
> was an easy example — next time, we'll move to a slightly harder one.)

- The next lecture pushes the same "separate what varies from what doesn't" idea further: instead
  of a varying *value* (a string), the varying part becomes a varying *block of code* — which is
  exactly the kind of problem anonymous classes are built to solve.

## Summary

- Two near-duplicate methods (`helloJava()`, `helloSpring()`) differ by exactly one line — the
  string being printed — while the surrounding "start"/"end" logic is identical.
- The fix: extract the **varying** part (the string) into a method parameter, and keep the
  **unchanging** part inside the method body unchanged.
- The root reason the original methods couldn't be reused is that each one hardcoded its own
  varying piece internally; parameterizing it lets the caller supply that piece from outside.
- This "separate what varies from what doesn't, then pass the varying part in from outside" idea
  is the general principle behind reducing duplication — and it's also exactly the setup for why
  anonymous classes are useful, which the next lecture builds on by making the varying part a
  **block of code** rather than a simple value.
