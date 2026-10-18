# Lecture 70 — Nested Classes, Inner Classes: Static Nested Class in Practice

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 08. 중첩 클래스, 내부 클래스1 (3/6)

Previous: [Lecture 69 - Nested Classes, Inner Classes - Static Nested Class](Lecture%2069%20-%20Nested%20Classes%2C%20Inner%20Classes%20-%20Static%20Nested%20Class.md)

This lecture refactors a small, realistic example to show the actual payoff of a static nested
class: hiding an implementation-detail helper class from anyone browsing the package.

## Before: Two Top-Level Classes (`ex1`)

```java
package nested.nested.ex1;

// Network 객체 안에서만 사용
public class NetworkMessage {

    private String content;

    public NetworkMessage(String content) {
        this.content = content;
    }

    public void print() {
        System.out.println(content);
    }
}
```

```java
package nested.nested.ex1;

public class Network {

    public void sendMessage(String text) {
        NetworkMessage networkMessage = new NetworkMessage(text);
        networkMessage.print();
    }
}
```

```java
package nested.nested.ex1;

public class NetworkMain {

    public static void main(String[] args) {
        Network network = new Network();
        network.sendMessage("hello java");
    }
}
```

```
hello java
```

- `NetworkMessage` just wraps a `content` string and prints it. That's its only job.
- `Network.sendMessage()` is the only code anywhere that ever touches `NetworkMessage` — it's a
  pure implementation detail of `Network`.
- `NetworkMain` only calls `Network`; it never references `NetworkMessage` directly.

### The Problem This Creates

The instructor frames this as a thought experiment: imagine switching roles to "a different
developer" encountering this package for the first time, without having written it.

- Someone opening the `ex1` package (ignoring `NetworkMain`) sees **two classes**: `Network` and
  `NetworkMessage`.
- Seeing two classes side by side, a new reader will naturally assume both are meant to be used,
  and will wonder things like:
  - "Do I need to use both `Network` and `NetworkMessage`?"
  - "Do I build a `NetworkMessage` myself and hand it to `Network`?"
  - Or maybe: "Is `NetworkMessage` a general-purpose class used all over the place by other
    classes too?"
- The only way to resolve that confusion is to actually open and read **both** classes' source
  code — only then does it become clear that `NetworkMessage` is purely internal to `Network` and
  was never meant to be touched directly.

> 두 클래스의 코드를 모두 확인하고 나서야 아~ Network 클래스만 사용하면 되는구나, NetworkMessage 는
> 단순히 Network 안에서만 사용되는구나 라고 이해할 수 있다. (Only after reading both classes' code
> do you realize: ah, you only need `Network` — `NetworkMessage` is purely internal to it.)

---

## After: Static Nested Class (`ex2`)

```java
package nested.nested.ex2;

public class Network {

    public void sendMessage(String text) {
        NetworkMessage networkMessage = new NetworkMessage(text);
        networkMessage.print();
    }

    private static class NetworkMessage {

        private String content;

        public NetworkMessage(String content) {
            this.content = content;
        }

        public void print() {
            System.out.println(content);
        }
    }
}
```

```java
package nested.nested.ex2;

public class NetworkMain {

    public static void main(String[] args) {
        Network network = new Network();
        network.sendMessage("hello java");
    }
}
```

```
hello java
```

Two changes were made while building this refactor in the recording:

1. `NetworkMessage` was moved **inside** `Network`'s body and marked `static class NetworkMessage`.
   - Reminder: copying a class like this in an IDE can drag its old `import` statements along —
     the instructor calls this out explicitly as something to double-check and clean up afterward.
2. `NetworkMessage`'s access modifier was additionally tightened to `private`.
   - `private static class NetworkMessage` means it is **completely unreachable from outside
     `Network`** — not just conventionally-internal, but compiler-enforced internal.
   - You literally cannot write `new Network.NetworkMessage(...)` or reference the type
     `Network.NetworkMessage` from any other class — it won't compile.
   - This sends "a strong signal of intent": *this class exists purely to be used inside me, and
     nowhere else.*

### The Payoff

Replaying the same thought experiment — a different developer opens the `ex2` package for the
first time:

- Ignoring `NetworkMain`, there is now only **one class visible**: `Network`.
- They can immediately tell `sendMessage()` is the entry point to call.
- If they do open `Network`'s source and notice `NetworkMessage` nested inside with `private`,
  they instantly understand — no investigation required — that it's a private implementation
  detail used solely inside `Network`.
  - Being nested alone already hints "used only inside here."
  - Adding `private` on top makes that unmistakable: "this really is used exclusively inside."

> Network 관련 라이브러리를 사용하기 위해 ex2 패키지를 열어본 개발자는 해당 클래스만 확인할 것이다.
> (A developer opening the `ex2` package to use the Network-related library will only need to
> check that one class.)

This is the concrete payoff of the "logical grouping" and "encapsulation" reasons for nesting
from Lecture 68 — fewer irrelevant classes cluttering the package namespace, and an
unambiguous, compiler-enforced signal of what's actually meant to be used externally.

---

## How to Reference a Nested Class: With or Without the Outer Class Name

The slide deck highlights a subtle but important rule about qualifying a nested class's name.

**From outside the outer class** (e.g. referencing `NestedOuter.Nested` from unrelated code),
you must qualify it with the outer class name:

```java
NestedOuter.Nested nested = new NestedOuter.Nested();
```

**From inside the outer class that the nested class belongs to**, the outer-class prefix can be
dropped entirely:

```java
public class Network {
    public void sendMessage(String text) {
        NetworkMessage networkMessage = new NetworkMessage(text);
    }
    private static class NetworkMessage {...}
}
```

- Note `Network.sendMessage()` writes `new NetworkMessage(text)`, **not**
  `new Network.NetworkMessage(text)` — writing the fully-qualified form here would be unusual and
  unnecessary, since you're already inside `Network`'s own body.

---

## The General Rule: When a Nested Class Is Being Used Wrong

The instructor generalizes this into a rule that applies to every kind of nested class (not just
static nested classes):

> 중첩 클래스(내부 클래스 포함)는 그 용도가 자신이 소속된 바깥 클래스 안에서 사용되는 것이다. 따라서
> 자신이 소속된 바깥 클래스가 아닌 외부에서 생성하고 사용하고 있다면, 이미 중첩 클래스의 용도에 맞지
> 않을 수 있다. 이때는 중첩 클래스를 밖으로 빼는 것이 더 나은 선택이다.
> (A nested class — including every inner-class variant — exists to be used inside the outer class
> it belongs to. If it's being created and used from somewhere outside that outer class, it's
> probably already being misused. At that point, pulling it out into a top-level class is the
> better choice.)

- If you ever find yourself writing `OuterClass.NestedClass` access patterns scattered around in
  unrelated code elsewhere in the codebase, that's a warning sign: the class probably shouldn't be
  nested anymore.
- **Exception noted in the recording**: occasionally two classes really are extremely tightly
  coupled, and the nested class does get referenced from outside here and there even though it
  stays nested — this happens, but it's the uncommon case.
  - In the large majority of cases, if the "inner" class is being used by other outside classes,
    the right move is to pull it out to the top level.

---

## Summary

- Refactoring `NetworkMessage` from a separate top-level class into a `private static` nested
  class inside `Network` hides an implementation detail that should never have been part of the
  package's public surface.
- **Before** (two top-level classes): anyone browsing the package sees both classes and has to
  read both bodies to figure out that `NetworkMessage` is purely internal.
- **After** (static nested + `private`): browsing the package shows only `Network`; seeing
  `NetworkMessage` nested and `private` instantly communicates "internal-only, don't touch."
- `private` on a nested class is a compiler-enforced guarantee, not just a naming convention —
  external code genuinely cannot reference `Network.NetworkMessage`.
- Reference a nested class as `OuterClass.NestedClass` from the outside; from inside the owning
  outer class, you can drop the outer-class prefix.
- General rule: a nested class should only be created and used from within its owning outer class.
  If outside code keeps needing to reach into it, that's a signal to pull it out to the top level
  instead — nesting has stopped matching its intended purpose.
