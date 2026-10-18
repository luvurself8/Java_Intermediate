# Lecture 80 — Nested Classes, Inner Classes 2: Anonymous Class Usage 2

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 09. 중첩 클래스, 내부 클래스2 (7/11)

Previous: [Lecture 79 - Anonymous Class Usage 1](Lecture%2079%20-%20Anonymous%20Class%20Usage%201.md)

Lecture 79 solved duplication by extracting a varying **value** (a string) into a parameter. This
lecture poses a harder version of the same problem — where the varying part is a **block of
code**, not a simple value — which is exactly the kind of problem that pushes toward interfaces,
polymorphism, and eventually anonymous classes.

## Before: Two Methods That Share Structure but Not Logic

```java
package nested.anonymous.ex;

import java.util.Random;

public class Ex1Main {

    public static void helloDice() {
        System.out.println("프로그램 시작");

        //코드 조각 시작
        int randomValue = new Random().nextInt(6) + 1;
        System.out.println("주시위 = " + randomValue);
        //코드 조각 종료

        System.out.println("프로그램 종료");
    }

    public static void helloSum() {
        System.out.println("프로그램 시작");

        //코드 조각 시작
        for (int i = 0; i < 3; i++) {
            System.out.println("i = " + i);
        }
        //코드 조각 종료

        System.out.println("프로그램 종료");
    }

    public static void main(String[] args) {
        helloDice();
        helloSum();
    }
}
```

```
프로그램 시작
주시위 = 5 //랜덤
프로그램 종료
프로그램 시작
i = 0
i = 1
i = 2
프로그램 종료
```

- `helloDice()` rolls a random 1–6 value; `helloSum()` runs a loop printing `i`. The "start"/"end"
  print statements are identical, but the middle **code block itself is structurally different**
  between the two methods — not just a different value.

> 참고로 앞의 문제는 변하는 문자열을 외부에서 전달하면 되었다. 이번에는 문자열 같은 데이터가 아니라 코드
> 조각을 전달해야 한다. (Note: the previous problem just needed a varying string passed in from
> outside. This time, instead of string-like data, you need to pass in a code fragment itself.)

The instructor's challenge: spend 10–20 minutes trying to merge these into one reusable method
before continuing.

---

## Why This Is a Fundamentally Harder Problem

> 코드 조각은 보통 메서드(함수)에 정의한다. 따라서 코드 조각을 전달하기 위해서는 메서드가 필요하다.
> 그런데 지금까지 학습한 내용으로는 메서드를 전달할 수 있는 방법이 없다. (A code fragment is normally
> defined inside a method. So passing a code fragment around would require passing a method itself
> — but nothing learned so far gives a way to pass a method directly.)

- Lecture 79's fix worked because `String` is ordinary data — trivially passable as a parameter.
- Here, the varying part is *behavior*, not data — Java has no way to hand a bare method around as
  a value (this course predates lambdas, which are covered in a later course).
- **The workaround**: you can't pass a method, but you *can* pass an **instance** — and then call
  a method on that instance.

> 대신에 메서드를 가지고 있는 인스턴스를 던지고, 그 인스턴스에 있는 메서드를 호출하는 식으로 되는
> 거죠. (Instead, you hand over an instance that carries the method, and call that method on the
> instance.)

---

## The Refactor: Interface + Implementations + Polymorphism

### 1. Define an interface with one method that represents "the code fragment"

```java
package nested.anonymous.ex;

public interface Process {
    void run();
}
```

### 2. Give each varying code fragment its own implementing class

```java
package nested.anonymous.ex;

import java.util.Random;

public class Ex1RefMainV1 {

    public static void hello(Process process) {
        System.out.println("프로그램 시작");

        //코드 조각 시작
        process.run();
        //코드 조각 종료

        System.out.println("프로그램 종료");
    }

    static class Dice implements Process {

        @Override
        public void run() {
            int randomValue = new Random().nextInt(6) + 1;
            System.out.println("주시위 = " + randomValue);
        }
    }

    static class Sum implements Process {

        @Override
        public void run() {
            for (int i = 0; i < 3; i++) {
                System.out.println("i = " + i);
            }
        }
    }

    public static void main(String[] args) {
        Dice dice = new Dice();
        Sum sum = new Sum();

        System.out.println("Hello 실행");
        hello(dice);
        hello(sum);
    }
}
```

```
Hello 실행
프로그램 시작
주시위 = 5 //랜덤
프로그램 종료
프로그램 시작
i = 0
i = 1
i = 2
프로그램 종료
```

- `Dice` and `Sum` are static nested classes (from Lecture 69) — the instructor notes this choice
  is not required; a regular top-level class would work exactly the same way here. Static nested
  classes are used purely because `Dice`/`Sum` are only meaningful alongside this example.
- Each implements `Process` and packs its own code fragment into `run()`.
- `hello(Process process)` no longer hardcodes *any* logic for what happens between "start" and
  "end" — it just calls `process.run()` and lets whatever was passed in decide what that means.

### 3. Polymorphism does the rest

> 이때 다형성을 활용해서 외부에서 전달되는 인스턴스에 따라 각각 다른 코드 조각이 실행된다. (Using
> polymorphism here, whichever instance is passed in from outside determines which code fragment
> actually runs.)

- `hello(dice)` runs `Dice.run()` — the dice-rolling logic.
- `hello(sum)` runs `Sum.run()` — the looping logic.
- The same `hello()` method body executes a completely different fragment depending solely on
  which object was handed to it — this is ordinary polymorphism, applied to solve the "pass a code
  fragment" problem.

---

## The General Principle

> 문자열 같은 데이터를 메서드에 전달할 때는 String, int와 같은 각 데이터에 맞는 타입을 전달하면 된다.
> 코드 조각을 메서드에 전달할 때는 인스턴스를 전달하고 해당 인스턴스에 있는 메서드를 호출하면 된다.
> (When passing data like a string to a method, just pass the matching data type — `String`,
> `int`, and so on. When passing a code fragment to a method, pass an instance, and call a method
> on that instance.)

| What you're passing | How to pass it |
|---|---|
| A value (string, number, etc.) | Pass it directly as a parameter of the matching type |
| A block of behavior / code fragment | Define an interface, implement it, pass an instance of the implementation, and call its method |

- `hello()` is now fully reusable: the unchanging "start"/"end" logic stays put in one place, and
  the varying logic is supplied entirely from outside by whichever `Process` implementation gets
  passed in.
- This is the exact same "separate what varies from what doesn't" principle from Lecture 79 — just
  applied one level deeper, where the varying part is executable logic rather than a plain value.

The instructor closes by noting this solution used a static nested class for `Dice`/`Sum` purely
for convenience — the next lecture revisits the same problem using a local class instead, which
sets up the payoff for anonymous classes.

## Summary

- When the varying part of near-duplicate code is a **value**, pass it as a parameter (Lecture
  79). When the varying part is a **block of code**, that approach doesn't work — Java has no way
  to pass a bare method as a value.
- The solution: define an interface with a single method representing "the code fragment," write
  one implementing class per variant (`Dice`, `Sum`), and pass an *instance* of the right
  implementation into the reusable method.
- The reusable method (`hello(Process process)`) calls `process.run()` without knowing or caring
  which implementation it received — polymorphism decides which code fragment actually executes
  based on the instance passed in.
- This generalizes the "separate what varies from what doesn't" principle: passing data uses a
  matching type directly; passing behavior uses an interface, an implementing instance, and a
  method call on that instance.
- `Dice` and `Sum` were written as static nested classes here purely for convenience — a top-level
  class would work identically; the choice of nesting doesn't affect this lecture's core lesson.
