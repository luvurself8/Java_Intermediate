# Lecture 81 — Nested Classes, Inner Classes 2: Anonymous Class Usage 3

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 09. 중첩 클래스, 내부 클래스2 (8/11)

Previous: [Lecture 80 - Anonymous Class Usage 2](Lecture%2080%20-%20Anonymous%20Class%20Usage%202.md)

Lecture 80 solved the `Process`/`Dice`/`Sum` problem using static nested classes. This lecture
progressively simplifies that same solution — first to a local class, then to an anonymous class,
then to passing the anonymous instance directly as an argument — and ends with a preview of where
Java eventually took this idea: **lambdas**.

## Step 1: Local Class Instead of Static Nested Class

```java
package nested.anonymous.ex;

import java.util.Random;

public class Ex1RefMainV2 {

    public static void hello(Process process) {
        System.out.println("프로그램 시작");

        //코드 조각 시작
        process.run();
        //코드 조각 종료

        System.out.println("프로그램 종료");
    }

    public static void main(String[] args) {

        class Dice implements Process {

            @Override
            public void run() {
                int randomValue = new Random().nextInt(6) + 1;
                System.out.println("주시위 = " + randomValue);
            }
        }

        class Sum implements Process {

            @Override
            public void run() {
                for (int i = 0; i < 3; i++) {
                    System.out.println("i = " + i);
                }
            }
        }

        Process dice = new Dice();
        Process sum = new Sum();

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

- `Dice` and `Sum` only matter inside this one `main()` method, so they're moved from static nested
  classes into **local classes** declared right where they're used.
- Nothing else changes — `hello()`'s own logic is untouched, and the output is identical to
  Lecture 80's version.
- The instructor notes you could also type the local variables as `Process dice = ...` instead of
  the concrete type, since a parent reference can hold a child instance — either works.

---

## Step 2: Anonymous Class — Declaration and Creation Fused

> 자, 이 지역 클래스 지금 한 번만 생성하죠. 이것도 딱 한 번만 생성해서 간단하게 사용하죠. 자, 이럴
> 경우에 뭘 할 수 있다? 익명 클래스로 바꿀 수가 있습니다. (This local class is only ever created
> once, and used simply. In this case, what can you do? You can turn it into an anonymous class.)

```java
package nested.anonymous.ex;

import java.util.Random;

public class Ex1RefMainV3 {

    public static void hello(Process process) {
        System.out.println("프로그램 시작");

        //코드 조각 시작
        process.run();
        //코드 조각 종료

        System.out.println("프로그램 종료");
    }

    public static void main(String[] args) {

        Process dice = new Process() {
            @Override
            public void run() {
                int randomValue = new Random().nextInt(6) + 1;
                System.out.println("주시위 = " + randomValue);
            }
        };

        Process sum = new Process() {

            @Override
            public void run() {
                for (int i = 0; i < 3; i++) {
                    System.out.println("i = " + i);
                }
            }
        };

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

- `class Dice implements Process { ... }` followed by `new Dice()` collapses into
  `new Process() { ... }` directly, matching exactly what Lecture 78 described — no separate name
  anywhere.
- This is the textbook case for reaching for an anonymous class, per the rule from Lecture 78: the
  local class was only ever instantiated once, which is exactly when an anonymous class fits.

---

## Step 3: Skip the Variable Entirely — Pass the Reference Directly

> 이 경우 익명 클래스의 참조값을 변수에 담아둘 필요 없이, 인수로 바로 전달할 수 있다. (In this case,
> there's no need to store the anonymous class's reference in a variable — it can be passed
> directly as an argument.)

```java
package nested.anonymous.ex;

import java.util.Random;

public class Ex1RefMainV4 {

    public static void hello(Process process) {
        System.out.println("프로그램 시작");

        //코드 조각 시작
        process.run();
        //코드 조각 종료

        System.out.println("프로그램 종료");
    }

    public static void main(String[] args) {

        System.out.println("Hello 실행");
        hello(new Process() {
            public void run() {
                int randomValue = new Random().nextInt(6) + 1;
                System.out.println("주시위 = " + randomValue);
            }
        });

        hello(new Process() {
            @Override
            public void run() {
                for (int i = 0; i < 3; i++) {
                    System.out.println("i = " + i);
                }
            }
        });
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

- Instead of `Process dice = new Process() {...}; hello(dice);`, the anonymous instance is
  constructed directly inside the call: `hello(new Process() {...});`.
- One syntax note from the recording: when passing a `new` expression directly as an argument like
  this, there is **no trailing semicolon** after the closing `}` of the anonymous class body — the
  semicolon only appears when you were assigning the expression to a variable.
- This is the same code, just with one fewer intermediate variable — not dramatically "better,"
  as the instructor admits, but noticeably more compact.

---

## The Developer's Next Ask — and Why It Can't Be Solved Yet

> 사실 지금 개발자인 내가 하고 싶은 건 이 인스턴스를 넘기고 싶은 게 아니라 사실 이 코드 조각을 넘기고
> 싶은 거죠. (What I, the developer, actually want is not to pass this instance — it's to pass this
> code fragment itself.)

- Up through Java 7, a method argument could only ever be one of two things:

| Argument kind | Examples |
|---|---|
| Primitive data type | `int`, `double`, etc. |
| Reference type (instance) | `Process`, `Member`, etc. |

- There was no way to pass a method — or more precisely, a function — directly as an argument.
  Building an instance just to carry one method around (everything done in this lecture so far)
  was the only option available.

---

## A Glimpse Ahead: Lambdas

> 자바 8에 들어서면서 큰 변화가 있었는데 바로 메서드(더 정확히는 함수)를 인수로 전달할 수 있게
> 되었다. 이것을 간단히 람다(Lambda)라 한다. (A major change came with Java 8: a method — more
> precisely, a function — could finally be passed directly as an argument. This is called, simply,
> a lambda.)

```java
package nested.anonymous.ex;

import java.util.Random;

public class Ex1RefMainV5 {

    public static void hello(Process process) {
        System.out.println("프로그램 시작");

        //코드 조각 시작
        process.run();
        //코드 조각 종료

        System.out.println("프로그램 종료");
    }

    public static void main(String[] args) {

        System.out.println("Hello 실행");
        hello(() -> {
            int randomValue = new Random().nextInt(6) + 1;
            System.out.println("주시위 = " + randomValue);
        });

        hello(() -> {
            for (int i = 0; i < 3; i++) {
                System.out.println("i = " + i);
            }
        });
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

- No class, no interface name, no instance keyword anywhere — just the code block itself,
  `() -> { ... }`, passed straight into `hello()`.
- The instructor is explicit this is only a teaser: lambdas are a large enough topic to warrant
  their own dedicated treatment later in the learning path, and the only goal here is recognizing
  that this capability exists and roughly what problem it solves.

### Lambdas Don't Make Anonymous Classes Obsolete

> 익명 클래스는 여기 보시면 여기다가 이렇게 지역변수가 아니고 멤버변수도 선언할 수가 있어요. 그렇기
> 때문에 익명 클래스도 필요합니다. 람다는 멤버변수 같은 걸 선언할 수가 없어요. 클래스가 아니기
> 때문에. (An anonymous class can declare member variables, not just local variables — so it's
> still needed. A lambda can't declare member variables, because it isn't a class.)

- An anonymous class is a real class — it can have its own fields (member variables), as seen with
  `int value = 0;` back in Lecture 78's example.
- A lambda is not a class at all — it has no place to hold its own member variables.
- Practical takeaway: lambdas are more concise for simple single-method cases, but anonymous
  classes remain necessary whenever you need actual state (fields) alongside the behavior.

---

## Summary

- The `Process`/`Dice`/`Sum` example was progressively simplified: static nested classes (Lecture
  80) → local classes → anonymous classes → an anonymous instance passed directly as a method
  argument (no intermediate variable).
- Passing a `new` expression directly as an argument drops the trailing semicolon that would
  otherwise follow a variable assignment.
- Anonymous classes exist because, historically, Java could only pass primitive values or object
  references as method arguments — there was no way to pass "just a method" directly.
- Java 8 introduced **lambdas**, which finally allow a function/code block to be passed directly
  as an argument, without defining any class or instance at all — a deep dive on lambdas is saved
  for a later, dedicated part of the course.
- Lambdas don't replace anonymous classes entirely: a lambda isn't a class and can't declare member
  variables, so anonymous classes are still the right tool whenever state is needed alongside
  behavior.
- Next up: a problem-and-solution lecture exercising everything covered so far — static nested
  classes, inner classes, local classes, and anonymous classes.
