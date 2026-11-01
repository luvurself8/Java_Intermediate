# Lecture 88 — Exception Handling 1 - Theory: Java Exception Handling - Exception Hierarchy

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 10. 예외 처리1 - 이론 (4/7)

Previous: [Lecture 87 - Need for Exception Handling - Handling Exceptions with Return Values](Lecture%2087%20-%20Need%20for%20Exception%20Handling%20-%20Handling%20Exceptions%20with%20Return%20Values.md)

## Starting Java's Real Exception Mechanism

Java provides a dedicated mechanism for handling unexpected situations at runtime — exceptions — built on five keywords and a set of exception objects:

- Keywords: `try`, `catch`, `finally`, `throw`, `throws`.
- This mechanism is what finally lets normal flow and error-handling flow be cleanly separated, the problem Lecture 87 ended on.

This lecture covers just the first piece: **the class hierarchy that Java's exceptions live in.**

---

## The Exception Hierarchy

```
                Object
                  |
              Throwable
               /      \
         Exception    Error
          /    \          \
  SQLException IOException  OutOfMemoryError
          \    /
     RuntimeException
       /         \
NullPointerException  IllegalArgumentException
```

- **`Object`**: in Java, everything except primitive types is an object — and that includes exceptions. Since `Object` is the root parent of all objects, it's also the root parent of all exceptions.
- **`Throwable`**: the actual top-level exception type — its name reflects that it's "something that can be thrown." Has two direct children: `Exception` and `Error`.
- **`Error`**: represents problems like running out of memory or serious system-level failures — things an application **cannot recover from**.
  - Application developers should never try to catch `Error`. There's nothing useful to do about it anyway.
- **`Exception`**: the practical, real top-level type that application logic actually uses.
  - `Exception` and everything under it are **checked exceptions** — with one specific carve-out: `RuntimeException`.
- **`RuntimeException`**: also called an **unchecked exception**, or (from its own name) a **runtime exception**.
  - The compiler does **not** check these.
  - `RuntimeException` and everything under it are all unchecked exceptions.
  - The instructor uses "unchecked exception" and "runtime exception" interchangeably from here on, following the class's own name.

### Checked vs. Unchecked (quick preview — detailed next lecture)

| | Checked exception | Unchecked (runtime) exception |
|---|---|---|
| Where it sits | `Exception` and its subclasses, **except** `RuntimeException` | `RuntimeException` and its subclasses |
| Compiler behavior | **Checks** it — must be explicitly handled or declared | Does **not** check it |
| Developer obligation | Must explicitly catch it or declare it being thrown, or the code won't compile | Not required to explicitly handle it |

> The names are the mnemonic: checked exceptions are checked by the compiler and so must be explicitly dealt with; unchecked exceptions are not checked by the compiler and so don't have to be.

Full detail on this distinction is the subject of upcoming lectures — for now, just understanding where each sits in the hierarchy is the goal.

---

## Catching a Parent Type Also Catches Its Children

This reuses a familiar inheritance idea: a variable of a parent type can hold a child-type instance. The same applies when **catching** exceptions.

- Catching a higher (parent) exception type with `catch` also catches everything beneath it in the hierarchy.
  - `catch (Exception e)` → also catches everything under `Exception` (including all checked exceptions and `RuntimeException`'s whole subtree).
  - `catch (RuntimeException e)` → also catches every unchecked exception beneath it.
  - `catch (Error e)` → also catches every subtype of `Error`.

### Why application code should never catch `Throwable`

- `Throwable` sits above **both** `Exception` and `Error`.
- Catching `Throwable` would therefore also catch `Error` — the exact category that was just established as unrecoverable and not meant to be caught.
- **Application logic should treat `Exception` as its practical top-level type to catch and throw** — never reach above it to `Throwable`.

---

## Summary

- Java's exception mechanism is built on five keywords (`try`, `catch`, `finally`, `throw`, `throws`) plus a hierarchy of exception objects.
- Exceptions are objects, so like everything else non-primitive in Java, their ultimate root is `Object`.
- `Throwable` is the actual top-level exception type, splitting into `Exception` (practical, application-level exceptions) and `Error` (unrecoverable system-level failures that application code must never try to catch).
- `Exception` and its subclasses are **checked exceptions**, compiler-enforced to be handled or declared — except for `RuntimeException`, which (along with its own subclasses) is an **unchecked exception** (a.k.a. **runtime exception**), not enforced by the compiler.
- Catching a parent exception type also catches every exception type beneath it — which is exactly why application code should catch starting from `Exception`, never from `Throwable` (catching `Throwable` would also silently catch unrecoverable `Error`s).
