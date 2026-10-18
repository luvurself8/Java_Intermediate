# Lecture 74 — Nested Classes, Inner Classes 2: Local Class - Getting Started

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 09. 중첩 클래스, 내부 클래스2 (1/11)

This section picks up where Section 08 left off: with the static nested class and the inner class
already covered, this lecture starts on the third of the four kinds of nested classes from
Lecture 68 — the **local class**.

## What a Local Class Is

> 지역 클래스(Local class)는 내부 클래스의 특별한 종류의 하나이다. 따라서 내부 클래스의 특징을 그대로
> 가진다. (A local class is a special kind of inner class. Therefore it keeps all the
> characteristics of an inner class.)

- A local class **is** an inner class — so everything from Lecture 71 still applies: it belongs to
  an instance of the outer class, and it can access the outer class's instance members (even
  `private` ones).
- What makes it "local" specifically: it's declared **inside a code block**, at exactly the same
  position a local variable would be declared — not directly in the outer class's body like a
  regular inner class.

```java
class Outer {

    public void process() {
        //지역 변수
        int localVar = 0;

        //지역 클래스
        class Local {...}

        Local local = new Local();
    }
}
```

- `Local` is declared inside `process()`'s body, right alongside `localVar` — this mirrors the
  "position maps to variable-declaration position" framing introduced back in Lecture 68.

### The Local Class's Defining Features

- It declares a class inside a code block, the same way a local variable is declared.
- It can access local variables of the enclosing block — this is the one capability a regular
  inner class doesn't have, and it's the focus of upcoming lectures on "local variable capture."

---

## Example 1: A Local Class Accessing Everything Around It

```java
package nested.local;

public class LocalOuterV1 {

    private int outInstanceVar = 3;

    public void process(int paramVar) {

        int localVar = 1;

        class LocalPrinter {
            int value = 0;

            public void printData() {
                System.out.println("value=" + value);
                System.out.println("localVar=" + localVar);
                System.out.println("paramVar=" + paramVar);
                System.out.println("outInstanceVar=" + outInstanceVar);
            }
        }

        LocalPrinter printer = new LocalPrinter();
        printer.printData();
    }

    public static void main(String[] args) {
        LocalOuterV1 localOuter = new LocalOuterV1();
        localOuter.process(2);
    }
}
```

```
value=1
localVar=1
paramVar=2
outInstanceVar=3
```

`LocalPrinter.printData()` reaches four different things, each demonstrating a different access
rule:

### 1. Its own instance field — `value`

- Not surprising — any class can access its own members.

### 2. The local variable of the enclosing block — `localVar`

- `localVar` is declared in `process()`, the method body `LocalPrinter` is nested inside.
- This access is the hallmark feature that separates a local class from a plain inner class.

### 3. The method parameter — `paramVar`

> 매개변수도 지역 변수의 한 종류예요. (A method parameter is also a kind of local variable.)

- Since a parameter is just a specific kind of local variable, this is really the same rule as #2
  — the local class can reach it the same way.

### 4. The outer class's instance member — `outInstanceVar`

- This works **because a local class is still an inner class** — it inherits the inner class's
  ability to reach the outer class's instance members (even `private` ones), on top of its own
  local-variable access.

| Local class can access... | Why |
|---|---|
| Its own members | Any class can |
| Local variables of its enclosing block | Defining feature of a local class |
| Method parameters of its enclosing block | Parameters are a kind of local variable |
| Outer class's instance members (even `private`) | Inherited from being an inner class |

### Scope: `LocalPrinter` Only Exists Inside `process()`

- The class name `LocalPrinter` is only usable within the code block it's declared in — you cannot
  reference `LocalPrinter` from anywhere outside `process()`'s body, the same way `localVar`
  itself is invisible outside that method.

### One Restriction: No Access Modifiers

> 지역 클래스는 지역 변수처럼 접근 제어자를 사용할 수는 없어요. (A local class cannot use access
> modifiers, just like a local variable can't.)

- You cannot write `private class LocalPrinter { ... }` or `public class LocalPrinter { ... }`
  inside a method — `private`/`public` are only valid on members, and a local class isn't a member
  of the outer class; it behaves like a local variable instead, and local variables never take
  access modifiers either.

---

## Example 2: A Local Class Implementing an Interface

> 당연한 이야기지만 내부 클래스를 포함한 중첩 클래스들도 일반 클래스처럼 인터페이스를 구현하거나, 부모
> 클래스를 상속할 수 있다. (Naturally, nested classes — including inner classes — can implement
> interfaces or extend a parent class just like a regular class can.)

```java
package nested.local;

public interface Printer {
    void print();
}
```

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

        LocalPrinter printer = new LocalPrinter();
        printer.print();
    }

    public static void main(String[] args) {
        LocalOuterV2 localOuter = new LocalOuterV2();
        localOuter.process(2);
    }
}
```

```
value=1
localVar=1
paramVar=2
outInstanceVar=3
```

- The only change from Example 1: `LocalPrinter` now `implements Printer`, and the method is
  renamed to `print()` with an `@Override` annotation to satisfy the interface contract.
- Behaviorally nothing else changes — same outputs, same access rules.
- The instructor is explicit that this example exists purely to establish a fact that will matter
  later: **a local class is still just a class**, so it can implement interfaces or extend parent
  classes exactly like a top-level class would.

> 사실은 뒤에 복선이 하나 있는데... 아무튼 지금은 뭐 이런 것들도 다 할 수 있다 라고 보시면 됩니다.
> (There's actually some foreshadowing here for later... but for now, just understand that a local
> class can do all of this too.)

---

## Summary

- A local class is a special kind of inner class — it keeps every inner-class capability (access
  to the outer class's instance members, even `private` ones) and adds one more: access to the
  local variables (and parameters) of the code block it's declared in.
- It's declared at the same position a local variable would be — inside a method body — and its
  name is only usable within that enclosing block.
- A local class **cannot** carry an access modifier (`private`/`public`/etc.), just like a local
  variable can't.
- A local class can implement interfaces and extend classes exactly like any other class — this
  fact is set up here because it becomes important in a later lecture (anonymous classes).
- Next up: a deeper look at exactly how local classes "capture" the local variables they access —
  and why, as a rule of thumb, you should treat a captured local variable's value as something you
  must not change.
