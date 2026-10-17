# Lecture 68 — Nested Classes, Inner Classes: What Are They?

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 08. 중첩 클래스, 내부 클래스1 (1/6)

This lecture is purely conceptual — no runnable example project yet. It sets up the vocabulary
(nested vs. inner) that every following lecture in this section builds on, so it's worth getting
precise about it here before diving into code.

## What Is a Nested Class?

Just like a `for` loop can contain another `for` loop (a "nested for-loop"), a class can contain
another class definition inside it — this is called a **nested class**.

```java
// nested for-loop
for (...) {
    for (...) {
    }
}
```

```java
class Outer {
    ...
    // nested class
    class Nested {
        ...
    }
}
```

- A nested class is simply "a class declared inside another class" — the term says nothing yet
  about *how* it relates to the outer class.
- Java further classifies nested classes based on **where exactly** they're declared, because
  that position determines their behavior.

---

## The Four Kinds of Nested Classes

There are **4 kinds total**, which split into **2 broad categories**:

- **Static nested class** (정적 중첩 클래스)
- **Inner class family** (내부 클래스 종류)
  - Inner class (내부 클래스)
  - Local class (지역 클래스)
  - Anonymous class (익명 클래스)

The instructor's key teaching trick here: **the position you declare a nested class in maps
exactly to the position you'd declare a variable in.**

| Variable declaration position | Equivalent nested class |
|---|---|
| Static variable (class variable) | Static nested class |
| Instance variable | Inner class |
| Local variable (inside a code block) | Local class |

```java
class Outer {
    ...
    // static nested class — same position as a static variable
    static class StaticNested {
        ...
    }

    // inner class — same position as an instance variable
    class Inner {
        ...
    }
}
```

- A **static nested class** has `static` in front, exactly like a static variable does.
- An **inner class** has no `static`, exactly like an instance variable.

```java
class Outer {
    public void process() {
        // local variable
        int localVar = 0;

        // local class — declared inside a code block, just like a local variable
        class Local {...}

        Local local = new Local();
    }
}
```

- A **local class** is declared inside a method body (a code block), the same place a local
  variable would be declared.
- The **anonymous class** isn't a separate "position" — it's a special variant of the local class
  that additionally has no name. It's covered in a later lecture.

---

## "Nested" vs. "Inner": What's the Actual Difference?

This is the conceptual core of the whole lecture, and the instructor spends most of the time on
it because the two words sound almost interchangeable in casual use but mean something precise
here.

- **Nested (중첩)**: a structural relationship where one thing is simply positioned inside
  another. Being "nested" says nothing about whether the inner thing belongs to or is part of the
  outer thing — it might be a completely unrelated class that just happens to live inside.
- **Inner (내부)**: something that is inside *and* is a constituent part of what contains it —
  i.e., it belongs to and makes up the outer thing.

> A big wooden box with a completely different, unrelated small wooden box placed inside it — that's **nested**.
> My heart, which is inside me and is a component that makes me function — that's **inner**.

Applying that distinction to classes:

- A **static nested class** is inside the outer class, but has **no relationship** to the outer
  class — it's effectively a totally separate class that just happens to be declared there for
  organizational convenience.
- An **inner class** is inside the outer class **and** is a component of it, closely tied to the
  outer class's behavior.

### The Precise Distinguishing Test: Instance Membership

The instructor states the real technical criterion for the "nested vs. inner" split explicitly:

> Does the class inside belong to an *instance* of the outer class, or not? That's the whole
> distinction.

- **Static nested class**: an entirely different class from the outer class. It does **not**
  belong to any instance of the outer class.
- **Inner class** (and its sub-kinds — inner, local, anonymous): a component of the outer class.
  It **does** belong to an instance of the outer class.

| | `static` keyword? | Belongs to outer instance? |
|---|---|---|
| Static nested class | Yes | No |
| Inner class | No | Yes |
| Local class | No | Yes |
| Anonymous class | No | Yes |

### The Inner Class Family, One More Level of Detail

- **Inner class** (내부 클래스, "inner" in the narrow sense): can access the outer class's
  instance members.
- **Local class** (지역 클래스): has all the characteristics of an inner class, **plus** it can
  also access local variables of the method it's declared in.
- **Anonymous class** (익명 클래스): has all the characteristics of a local class, **plus** it has
  no class name — "anonymous" literally means nameless.

---

## Terminology Clarification (Important for Reading Real Code/Docs)

- **Nested class** (중첩 클래스) — the umbrella term. Includes *both* the static nested class
  *and* every kind of inner class. Anything that is "a class inside a class" is a nested class.
- **Static nested class** (정적 중첩 클래스) — refers specifically to the `static` variant.
- **Inner class** (내부 클래스), used broadly — refers to inner class + local class + anonymous
  class together.

> Strictly speaking, a static nested class should never be called an "inner class" — it doesn't
> belong to an instance, so it doesn't fit that category.

### A Note on How the Terms Are Actually Used in the Field

The instructor flags this explicitly because it trips people up when reading other developers'
code or blog posts:

- In practice, most developers **don't** carefully distinguish "nested" from "inner" — they just
  say "nested class" or "inner class" loosely to mean "a class declared inside another class."
- This happens because, loosely speaking, an inner class *is* a kind of nested class too, so the
  sloppy usage isn't *wrong*, just imprecise.
- Strictly/technically, a `static`-prefixed static nested class should **not** be called an inner
  class.
- Since most developers don't bother with this precision, when you read "inner class" or "nested
  class" out in the wild, you should **infer the exact meaning from context** rather than assuming
  strict terminology is being used.

---

## When Should You Use a Nested Class?

A direct rule of thumb from the instructor:

> Every nested class — including all inner-class variants — should only be used when a specific
> class is used exclusively inside one other class, or when the two classes are extremely tightly
> coupled. If multiple outside classes need to use that class, it must **not** be a nested class
> — pull it out into its own top-level class instead.

## Why Use a Nested Class At All? (Two Motivating Reasons)

These are introduced conceptually here; the instructor notes they'll make a lot more sense once
concrete examples show up in the next lectures.

1. **Logical grouping.** If a class is only ever used inside one other specific class, grouping it
   inside that class is more logically organized than leaving it as a top-level class in the
   package.
   - Practical benefit: when someone opens the package, they don't see a bunch of classes that are
     irrelevant outside of one narrow use case — those helper classes stay hidden inside the class
     that actually uses them, instead of being exposed at the package level where they look like
     they might be reusable.
2. **Encapsulation.** A nested class can access the outer class's `private` members.
   - This lets the outer and nested class be tightly coupled while still keeping fields `private`.
   - It can also eliminate the need for otherwise-unnecessary `public` methods that would exist
     purely to expose those private members to an external helper class.
   - The instructor explicitly says this one is hard to grasp from words alone and promises a
     worked example in an upcoming lecture.

---

## Summary

- A nested class is any class declared inside another class; there are 4 kinds, in 2 categories.
- **Static nested class**: `static` keyword, declared like a static variable, does **not** belong
  to an instance of the outer class — effectively an unrelated class that's just co-located.
- **Inner class**: no `static`, declared like an instance variable, **does** belong to an instance
  of the outer class, and can access the outer instance's members.
- **Local class**: everything an inner class can do, plus access to the enclosing method's local
  variables; declared inside a code block like a local variable.
- **Anonymous class**: everything a local class can do, plus it has no name — a later lecture.
- The precise distinguishing test between "nested" and "inner" is **instance membership**: does
  the inner class belong to an instance of the outer class, or not?
- Strictly, only inner/local/anonymous classes are "inner classes"; a static nested class is not,
  even though real-world code often uses the terms loosely and interchangeably — read by context.
- Use a nested class only when it's used exclusively by, or extremely tightly coupled with, one
  outer class. If other classes need it too, make it a top-level class instead.
- Two reasons to nest: **logical grouping** (hide implementation-detail helper classes from the
  rest of the package) and **encapsulation** (access the outer class's `private` members directly,
  avoiding unnecessary `public` exposure).
