# Lecture 99 — Exception Handling 2 - Practice: Practical Exception Handling Plan - Explanation

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (8/11)

Previous: [Lecture 98 - Exception Hierarchy - Usage](Lecture%2098%20-%20Exception%20Hierarchy%20-%20Usage.md)

## What's the Modern Best Practice?

With checked exceptions, unchecked exceptions, and exception hierarchies all covered, the natural question: **which should real-world applications actually use?**

This lecture is purely conceptual — no new code yet (that starts next lecture). It lays out the reasoning that the Java ecosystem converged on over time, building toward a concrete recommendation.

---

## Problem 1: Most Exceptions Can't Actually Be Recovered From

Think about what a typical exception looks like in a real system: the remote server you're talking to is down, or the database server is unreachable.

- These are **system-level failures** — not something application code can fix by catching and retrying.
- Concretely: if a bank-transfer flow can't reach the destination server, retrying the same call just produces the same failure again. The network cable wasn't magically repaired by catching the exception.
- **Recovering** from an exception means catching it and doing something that restores normal operation. For this entire category of failure — remote systems being down, infrastructure problems — there is nothing the catching code can actually *do*.

> "실무에서 일을 해보면 거의 놀랍게도 거의 대부분 내가 해결할 수 없는 예외들이에요."

The only reasonable response to this category of failure:

- Show the user something like *"We're experiencing a system issue — sorry for the inconvenience, we're working on it"* (an error page, for a web app).
- Log the failure somewhere so developers can notice and investigate quickly.

That's it. There's no meaningful "recovery" logic to write for most of these.

---

## Problem 2: Checked Exceptions Become a Growing Burden

Checked exceptions exist specifically so the compiler catches a developer forgetting to handle something important — historically a genuinely good safety net.

But as applications grow — more libraries, more external systems connected — this becomes expensive:

### The checked-exception scenario, walked through

Imagine `Main → Service → {NetworkClient, DatabaseClient, ...}`, where every library used makes its own checked exception type (`NetworkException`, `DatabaseException`, `XxxException`, ...).

- Every one of these exceptions propagates up to `Service`. Since they're all checked, `Service` must either catch each one or declare `throws` for each one.
- But as established above, `Service` usually **can't actually recover** from a remote server being down or a database being unreachable — so throwing it further out is often the better choice.
- Throwing it further out means listing every single one of these exception types in `throws` — ten exceptions means ten names cluttering the method signature.
- **It doesn't stop there.** If there's a `Facade` class between `Main` and `Service`, it faces the exact same dilemma — it can't recover from these either, so it also has to `throws` every one of them, copy-pasted down the call chain. Every additional layer repeats this boilerplate.

> "개발자는 본인이 다룰 수 없는 수많은 체크 예외의 지옥에 빠지게 됩니다." — "예외 처리 지옥" (exception-handling hell), as labeled in the slides.

### The tempting "solution" that's actually the worst option

A clever-looking shortcut: since `Exception` is the parent of everything, just declare `throws Exception` everywhere instead of listing types individually.

```java
class Facade {
    void send() throws Exception { ... }
}

class Service {
    void sendMessage(String data) throws Exception { ... }
}
```

- This compiles, covers every current exception, and never needs updating when new exception types are added later. Looks like a clean win.
- **It's actually one of the worst things you can do.** `throws Exception` throws away the entire point of checked exceptions: the compiler can no longer distinguish *which* specific checked exceptions are involved.
- If a genuinely important new checked exception is introduced somewhere deep in the call chain, the compiler sees `throws Exception` already covers it and raises **no compile error** — exactly the oversight-catching mechanism checked exceptions exist for is silently disabled.
- The instructor is explicit: static analysis tools will flag `throws Exception` as a problem, and it should only be used when truly unavoidable, never as a general convenience.

---

## Summary of the Two Problems

1. **Unrecoverable exceptions outnumber recoverable ones.** Most real failures (remote systems down, infra problems) can't be fixed by catching and retrying — they just need to propagate out to somewhere that can show an error and log it.
2. **Checked exceptions impose a growing tax.** Every checked exception that can't be handled locally must be explicitly declared via `throws` at every layer that doesn't handle it, and this compounds as more libraries and external systems get added. (A few other related problems exist too, covered in more depth in the instructor's separate Spring Data Access course — not needed at this stage.)

The practical conclusion: developers generally don't want to be forced to deal with every single exception pouring out of every library they use — especially ones they have no way to fix anyway. **Catching what you can actually act on, and otherwise not being forced to think about the rest, is often the better default.**

---

## The Alternative: an Unchecked Exception Scenario

Same call shape (`Main → Service → {NetworkClient, DatabaseClient, XxxClient}`), but this time every client throws **unchecked (runtime)** exceptions instead.

- `Service` calls all three clients without any `throws` declaration at all — unchecked exceptions don't require it.
- If `Service` can't do anything about a given exception (the usual case for these kinds of system failures), it simply doesn't catch it — the exception automatically propagates upward, no boilerplate needed.
- If `Service` *can* meaningfully handle a specific exception, nothing stops it from catching just that one — unchecked exceptions can still be caught exactly like checked ones when that's useful.
- Adding more libraries over time means more unchecked exceptions might propagate through — but `Service`'s method signature and `throws` list **never need to change** to accommodate them. Only the exceptions actually worth handling get explicit `catch` blocks; everything else is simply left alone.

> "서비스에서는 어차피 본인이 처리할 수 없는 예외들이기 때문에 밖으로 던지는 것이 더 나은 결정이다. 언체크 예외는 잡지 않으면 throws 선언이 없어도 자동으로 밖으로 던진다."

---

## Where Do All Those Unhandled Exceptions Go? Common Exception Handling

Exceptions that no intermediate layer (`Service`, `Facade`, etc.) can resolve shouldn't be handled piecemeal in scattered places — instead, build **one common place** where they're all funneled and dealt with uniformly.

- Picture `Main` as the point closest to the user (e.g. where keyboard input is read, or — in a web app — the outermost request-handling layer).
- When an exception finally reaches that point unhandled, that's where the common handling logic lives:
  - Show the user a generic, safe message — *"Sorry, there's currently a system issue"* — never the raw exception detail.
  - Log the real exception details somewhere developers can see quickly — a log file, and/or alerting via email, Slack, etc., so the team notices fast.

> "이런 것들은 뭐냐, 고객님 죄송합니다 하고 로그를 남기는 정도로 해서 우리 내부 개발자가 로그를 남기고 빨리 인지할 수 있게 하는 정도로 공통으로 처리하면 됩니다."

This is the shape the next lecture implements directly: converting the running example's exceptions to unchecked, and adding a single common-handling point for whatever can't be resolved along the way.

---

## Summary

- Most exceptions in real systems are genuinely unrecoverable (remote servers down, infrastructure failures) — catching and retrying just reproduces the same failure, so the only sane response is showing the user a generic error and logging the details for developers.
- Checked exceptions compound this problem: every checked exception a class can't handle must be explicitly declared via `throws`, repeated at every layer in the call chain that also can't handle it — producing "checked exception hell" as applications and their library dependencies grow.
- `throws Exception` as a blanket fix is actively harmful, not just lazy: it disables the compiler's ability to flag missing handling for any *new*, genuinely important checked exception introduced later, defeating the entire purpose of checked exceptions.
- Unchecked exceptions avoid this tax entirely: methods never need `throws` declarations for exceptions they can't handle, new exception types from new libraries don't require signature changes anywhere, and specific exceptions worth handling can still be caught exactly as before.
- The practical pattern: let unrecoverable exceptions propagate automatically (no `throws` boilerplate), catch only what's genuinely actionable, and route everything else to **one common exception-handling point** near the application's boundary — show the user a safe generic message, log the real details for developers. The next lecture implements exactly this.
