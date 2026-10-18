# Lecture 78 — Nested Classes, Inner Classes 2: Anonymous Class - Getting Started

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 09. 중첩 클래스, 내부 클래스2 (5/11)

Previous: [Lecture 77 - Local Class - Local Variable Capture 3](Lecture%2077%20-%20Local%20Class%20-%20Local%20Variable%20Capture%203.md)

With local classes fully covered, this lecture introduces the fourth and final kind of nested
class from Lecture 68: the **anonymous class**. As foreshadowed back in Lecture 74 ("there's some
foreshadowing here for later"), this is where that setup pays off.

## What an Anonymous Class Is

> 익명 클래스(anonymous class)는 지역 클래스의 특별한 종류의 하나이다. 익명 클래스는 지역 클래스인데,
> 클래스의 이름이 없다는 특징이 있다. (An anonymous class is a special kind of local class. An
> anonymous class is a local class, with the distinguishing feature that it has no name.)

- Everything learned about local classes still applies — including local variable access and
  capture rules — an anonymous class is simply a local class with its name removed.

### Revisiting `LocalOuterV2`: Declaration and Creation Are Two Separate Steps

```java
package nested.local;

public class LocalOuterV2 {

    private int outInstanceVar = 3;

    public void process(int paramVar) {
        int localVar = 1;

        class LocalPrinter implements Printer {

            int value = 0;

            @Override
            public void print() {
                System.out.println("value=" + value);
                System.out.println("localVar=" + localVar);
                System.out.println("paramVar=" + paramVar);
                System.out.println("outInstanceVar=" + outInstanceVar);
            }
        }

        Printer printer = new LocalPrinter();
        printer.print();
    }

    public static void main(String[] args) {
        LocalOuterV2 localOuter = new LocalOuterV2();
        localOuter.process(2);
    }
}
```

Using a local class, as in Lecture 74, always takes **two distinct steps**:

```java
//선언
class LocalPrinter implements Printer {
    //body
}

//생성
Printer printer = new LocalPrinter();
```

1. **Declaration**: define the local class, giving it a name (`LocalPrinter`) and having it
   implement `Printer`.
2. **Creation**: separately call `new LocalPrinter()` to instantiate it.

> 익명 클래스를 사용하면 클래스의 이름을 생략하고, 클래스의 선언과 생성을 한번에 처리할 수 있다. (With
> an anonymous class, you can skip the class name and handle declaration and creation in a single
> step.)

---

## Rewriting `LocalOuterV2` With an Anonymous Class

```java
package nested.anonymous;

import nested.local.Printer;

public class AnonymousOuter {

    private int outInstanceVar = 3;

    public void process(int paramVar) {

        int localVar = 1;

        Printer printer = new Printer() {
            int value = 0;

            @Override
            public void print() {
                System.out.println("value=" + value);
                System.out.println("localVar=" + localVar);
                System.out.println("paramVar=" + paramVar);
                System.out.println("outInstanceVar=" + outInstanceVar);
            }
        };

        printer.print();
        System.out.println("printer.class=" + printer.getClass());
    }

    public static void main(String[] args) {
        AnonymousOuter main = new AnonymousOuter();
        main.process(2);
    }
}
```

```
value=0
localVar=1
paramVar=2
outInstanceVar=3
printer.class=class nested.anonymous.AnonymousOuter$1
```

> 이 코드는 앞서 설명한 LocalOuterV2와 완전히 같은 코드이다. 여기서는 익명 클래스를 사용했다. (This
> code is functionally identical to `LocalOuterV2` — it just uses an anonymous class here instead.)

- The declaration step and the creation step from `LocalOuterV2` are fused into a single
  expression: `new Printer() { ...body... };`.
- There is no `LocalPrinter` name anywhere — the class is defined and instantiated in the same
  breath.

---

## Parsing `new Printer() { ...body... }`

This syntax looks unusual at first, so the instructor walks through exactly what it means.

> 여기서 Printer는 Printer 객체를 생성한다는 뜻이 아니라, 이 Printer를 상속받거나 구현하겠다는
> 의미예요. (Here, "Printer" does not mean "create a Printer object" — it means "I'm going to
> extend or implement Printer.")

- `new` is immediately followed by the parent type you're extending or implementing — here,
  `Printer`.
- **Important clarification**: this is *not* instantiating the `Printer` interface — that's
  impossible in Java; interfaces can never be instantiated directly.
- What it actually does: creates an anonymous class **that implements `Printer`**, and
  simultaneously instantiates that new class. `Printer` here names the type being
  implemented/extended, not the type being constructed.
- The `{ ... }` block is the body of this unnamed class — it's exactly where the implementation of
  `Printer`'s methods goes, the same content that would otherwise sit inside a named
  `class LocalPrinter implements Printer { ... }`.

> 쉽게 이야기해서 Printer를 상속(구현)하면서 바로 생성하는 것이다. (Simply put: it implements/extends
> `Printer` while creating the instance at the same time.)

---

## Characteristics of an Anonymous Class

- It declares an unnamed local class **and** instantiates it, in one combined step.
- It **must** extend a parent class or implement an interface — because `new` always needs
  *something* written right after it to say what's being extended/implemented. There's no
  syntax for an anonymous class with neither a superclass nor an interface.
- Having no name means an anonymous class **cannot have a constructor** — only the default
  constructor is ever used, since there's no name to declare a constructor under.
- Internally, Java names it using the pattern `OuterClassName$N` — here, `AnonymousOuter$1`. If a
  class contains multiple anonymous classes, they're numbered `$1`, `$2`, `$3`, and so on, in the
  order the compiler encounters them.

---

## Why Use an Anonymous Class? (And When Not To)

### The Advantage

> 익명 클래스를 사용하면 클래스를 별도로 정의하지 않고도 인터페이스나 추상 클래스를 즉석에서 구현할 수
> 있어 코드가 더 간결해진다. (Using an anonymous class lets you implement an interface or abstract
> class on the spot, without a separate class definition — making the code more concise.)

- The instructor is upfront that an anonymous class isn't strictly necessary: a plain local class
  would work just as well for anything an anonymous class can do.
- The actual benefit is purely convenience — skipping the ceremony of a separate name and separate
  declaration when you only need the implementation once.

> 하지만 복잡하거나 재사용이 필요한 경우에는 별도의 클래스를 정의하는 것이 좋다. (But for complex or
> reusable logic, it's better to define a separate, named class.)

### When It Can't Be Used: More Than One Instance

> 익명 클래스는 단 한 번만 인스턴스를 생성할 수 있다. (An anonymous class can only ever have one
> instance created.)

```java
Printer printer1 = new LocalPrinter();
printer1.print();

Printer printer2 = new LocalPrinter();
printer2.print();
```

- An anonymous class is defined *while* being instantiated — there's no separate name left to
  reuse for a second `new` call.
- If you need the same class body instantiated multiple times (as shown above with a named
  `LocalPrinter`), you must fall back to a regular, named local class — anonymous classes only fit
  the single-use case.

---

## Summary

- An anonymous class is a local class with no name — it inherits every local-class behavior
  covered so far (instance access, local variable capture, the `final`/effectively-final rule).
- Normally a local class requires two steps — declare the class, then separately `new` it. An
  anonymous class fuses both into one expression: `new ParentTypeOrInterface() { body };`.
- The type right after `new` is what the anonymous class implements or extends — it is never the
  type being directly instantiated (you can never `new` an interface itself).
- An anonymous class must always extend something or implement something, because `new` requires
  a type right after it.
- Having no name means no constructor can be declared — only the default constructor is used.
- Internally, Java names anonymous classes `OuterClassName$1`, `$2`, `$3`, etc., in declaration
  order.
- The upside is conciseness for one-off implementations; the downside is you can only ever create
  a single instance of a given anonymous class — for anything reused or complex, use a named local
  class (or a regular top-level class) instead.
