# Lecture 89 — Exception Handling 1 - Theory: Java Exception Handling - Basic Exception Rules

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 10. 예외 처리1 - 이론 (5/7)

Previous: [Lecture 88 - Java Exception Handling - Exception Hierarchy](Lecture%2088%20-%20Java%20Exception%20Handling%20-%20Exception%20Hierarchy.md)

## Exceptions Are Like Passing a Hot Potato

> "예외는 폭탄 돌리기와 같다. 예외가 발생하면 잡아서 처리하거나, 처리할 수 없으면 밖으로 던져야 한다."

Every method that receives a propagating exception has exactly one choice to make: **handle it, or pass it along.** There's no third option — ignoring it isn't possible, because an unhandled exception keeps moving by itself.

---

## Case 1: The Exception Gets Caught and Handled

```
1: Main → Service
2: Service → Client
3: Client: exception occurs
4: Client → Service: exception thrown (propagates up)
5: Service: exception processed (caught)
6: Service → Main: normal flow resumes
```

- `Main` calls `Service`, `Service` calls `Client` — a normal call chain.
- An exception occurs inside `Client`.
- `Client` can't handle it, so it throws the exception **back out to whoever called it** — in this case, `Service`.
  - "밖으로 던진다" here specifically means: back to the caller. `Client`'s "outside" is `Service`, because `Service` is what invoked `Client`.
- `Service` receives the exception. This time, `Service` **does** catch it with `catch` and handles it.
- Once an exception is caught and handled, **the application resumes normal flow** from that point on — `Service` can do whatever cleanup/logic it needs and then return normally to `Main`.

## Case 2: Nobody Handles It — It Keeps Propagating

```
1: Main → Service
2: Service → Client
3: Client: exception occurs
4: Client → Service: exception thrown
5: Service → Main: exception thrown (Service couldn't handle it either)
```

- Same call chain, same exception in `Client`.
- `Client` throws it out to `Service`, same as before.
- But this time `Service` **also** can't handle it — so `Service` makes the same choice `Client` did: throw it further out, to whoever called `Service`, which is `Main`.
- The exception just keeps getting re-thrown outward, one caller at a time, until someone catches it — or until it runs out of callers.

---

## The Two Fundamental Rules

Only two rules to remember for exceptions:

1. **An exception must either be caught and handled, or thrown outward.** There is no other option — you either resolve it yourself or hand it to your caller.
2. **Catching or throwing a type also covers all of its child types**, because exceptions are objects and inheritance applies the same way it does everywhere else in Java (a parent-typed reference can hold a child instance).
   - `catch (Exception e)` catches `Exception` itself **and** every exception type beneath it in the hierarchy.
   - `throws Exception` (or re-throwing an `Exception`-typed variable) propagates `Exception` itself **and** every subtype of it.
   - Concretely: catching `RuntimeException` also catches all of its children; throwing `RuntimeException` outward also throws all of its children outward.

These two rules are literally everything to remember about exception semantics — the catch/throw keyword mechanics covered in upcoming lectures build on exactly this.

---

## What Happens If Nothing Ever Catches It?

If an exception keeps getting thrown outward and is never caught anywhere in the call chain, it eventually reaches the very outermost caller — `main()`.

- When an exception propagates out of `main()` itself, Java prints the exception's stack trace log to the console and **terminates the program.**
- This is the familiar "red text in the console, program just stops" experience every Java developer has hit before — that's exactly this mechanism: an unhandled exception reaching `main()`'s outer boundary and ending the JVM process.

---

## Summary

- An exception that occurs inside a method must be either caught and handled there, or thrown out to the method's caller — there's no third option.
- Once caught and handled, the application resumes normal flow from that point forward.
- If not handled, the exception propagates outward one caller at a time, repeating the same catch-or-throw decision at every level, until something catches it or it runs out of callers.
- Two rules cover all of exception semantics: (1) catch it or throw it outward — nothing else is possible, and (2) catching/throwing a given exception type also catches/throws every one of its subtypes, by ordinary inheritance.
- If an exception is never caught anywhere and propagates all the way out of `main()`, Java prints the exception log and terminates the program — the classic "red error, program stopped" scenario.
- With the theory in place (hierarchy + these two rules), the next lectures move into actual code: checked exceptions, then unchecked exceptions.
