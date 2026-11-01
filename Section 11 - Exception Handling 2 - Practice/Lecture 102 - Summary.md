# Lecture 102 — Exception Handling 2 - Practice: Summary

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (11/11)

Previous: [Lecture 101 - try-with-resources](Lecture%20101%20-%20try-with-resources.md)

## No Problem-and-Solution This Time

Every section so far has closed with a hands-on "문제와 풀이" (problem and solution) segment. This lecture explicitly skips that format:

> "이번 강의에는 문제와 풀이가 없는데요. 이 예외처리 강의는 하나의 예제가 큰 흐름으로 이어지는 것을 보셨을 거예요. 그래서 여러분이 예제를 처음부터 복습하면서 큰 흐름을 스스로 정리해보는 것을 문제와 풀이로 하겠습니다."

- The suggested self-study approach: go back to the original "bad" version of the code (the one using return-value error codes), and re-derive each subsequent improvement step yourself before checking it against the actual lecture material — essentially replaying Lectures 92–101 as a self-guided exercise rather than a separate exercise file.
- This lecture itself is a pure retrospective — tying together the entire arc of Section 11 (and its foundation in Section 10) into one coherent narrative, not introducing new code.

---

## The Full Arc, Lecture by Lecture

### 1. Why exception handling exists at all (Lectures 85–87, recapped here)

The starting point was a simple `Main → NetworkService → NetworkClient` program (`sendMessage(data)` → `connect()`/`send(data)`/`disconnect()`) that reported errors through **return values** instead of exceptions:

- Error situations were signaled by returning something like a status/error code from each method.
- The caller then had to branch on that returned value to figure out what happened and what to do next.

This produced a specific, recurring problem:

> "정상 흐름과 예외 흐름이 섞여 있기 때문에 코드를 한눈에 이해하기 어렵다. 쉽게 이야기해서 가장 중요한 정상 흐름이 한눈에 들어오지 않는다. 심지어 예외 흐름이 더 많은 코드 분량을 차지한다. 실무에서는 예외 처리가 훨씬 더 복잡하다."

- Normal-path logic and error-handling logic were interleaved line by line, making the "happy path" hard to follow at a glance.
- In real systems, the error-handling code often ends up being the *larger* share of the method body — making this interleaving problem worse than the toy example suggests.
- This specific problem — separating normal flow from exception flow — turns out to be **structurally unsolvable** by the return-value approach alone; it requires a fundamentally different mechanism.

### 2. Java's actual exception mechanism (Section 10, Lectures 88–91)

Java solves the separation problem directly: normal flow lives in `try`, exception flow lives in `catch`.

```
                Throwable
                 /      \
             Error      Exception
          (JVM-internal)   |
                 /----------+----------\
            (checked)              RuntimeException
       e.g. IOException         (unchecked exceptions)
```

- `Error` is the JVM's own internal problem-reporting mechanism (out-of-memory, etc.) — not something application code is meant to create or catch; application code deals with `Exception` and its descendants.
- Everything under `Exception` that does **not** extend `RuntimeException` is a **checked exception**.
- Everything that **does** extend `RuntimeException` is an **unchecked (runtime) exception**.

> "체크 예외와 언체크 예외의 차이는 예외를 처리할 수 없을 때 예외를 밖으로 던지는 부분에 있습니다. 이 부분을 필수로 선언해야 되는가, 생략해야 되는가의 차이가 있었죠."

- That's genuinely the **entire** mechanical difference: a checked exception that isn't caught *must* be declared with `throws` in the method signature, or the code fails to compile. An unchecked exception that isn't caught is automatically propagated whether or not `throws` is declared — the compiler simply doesn't check for it.
- Everything else — how they propagate up the call stack, how `catch` matches them, how `finally` behaves around them — is identical between the two categories.

### 3. The first real implementation: one exception class, an error code (Lectures 92–93)

- The first working version introduced a single custom exception class (`NetworkClientException`), distinguishing failure kinds internally via an `errorCode` string field.
- `try`/`catch` replaced the return-value branching — a direct, if somewhat unrefined ("무식하게"), first cut.
- A more deliberate second pass made the recovery logic in `catch` more meaningful — not just logging the error, but giving the caller (`Main`'s input loop) a real chance to continue operating normally afterward.

### 4. The resource-leak problem, solved by `finally` (Lectures 94–96)

- Once exceptions started interrupting the normal flow of `connect() → send(data) → disconnect()`, a new problem surfaced: if an exception was thrown partway through, `disconnect()` might never run — leaving the resource (the network connection) open.
- `finally` closed this gap cleanly: code placed in `finally` is **guaranteed to run** regardless of how the `try` block exits — normal completion, a caught exception, or an uncaught exception propagating out.

> "처리할 수 없는 예외와 finally, finally는 항상 호출돼요. 어떤 경우라도. 보장이 되기 때문에 자바 문법상."

### 5. Exception hierarchies: catching at the right level of specificity (Lectures 97–98)

- The single exception class with an `errorCode` field was replaced with a real type hierarchy: `NetworkClientExceptionV3` as parent, `ConnectExceptionV3`/`SendExceptionV3` as specific children, each carrying its own relevant data (`address`, `sendData`).
- This unlocked a layered `catch` strategy: catch a specific subtype for precise handling, catch the shared parent to sweep up every remaining sibling exception in one block, and catch `Exception` as a final fallback — ordered most-specific-first, since `catch` clauses are matched top-to-bottom and Java's compiler rejects an unreachable, overly-broad `catch` placed before a more specific one.
- `catch (TypeA | TypeB e)` was also shown as a way to handle two unrelated sibling types identically in one block — at the cost of only being able to call methods on their nearest common ancestor type.

> "예외를 계층화하고 다양하게 만들면 더 세밀한 동작들을 깔끔하게 처리할 수 있습니다. 그리고 특정 분류의 공통 예외들도 한 번에 catch로 잡아서 처리할 수가 있습니다."

### 6. The practical exception-handling plan (Lectures 99–100)

This is where the course's central real-world recommendation gets made, built on two observations:

- **Most exceptions can't actually be recovered from.** A down remote server or an unreachable database isn't something `catch`-and-retry fixes — the only sane response is a generic error shown to the user and a logged detail for developers.
- **Checked exceptions compound this into "exception-handling hell"** as applications pull in more libraries: every checked exception a class can't handle must be explicitly declared via `throws`, repeated at every intermediate layer that also can't handle it.
- The tempting shortcut, `throws Exception`, was called out as actively harmful rather than merely lazy — declaring the broadest possible checked type disables the compiler's ability to flag a genuinely important *new* checked exception introduced later, since the blanket `throws Exception` already "covers" it.

> "스로우즈 익셉션이라는 극단적인 방법도 자주 사용했습니다. 물론 이 방법을 사용하면 안 돼요. 모든 예외를 던진다고 선언한 것인데, 결과적으로 어떤 예외를 잡고 어떤 예외를 던지는지 알 수 없기 때문이에요."

The resulting implementation (`exception.ex4`, Lecture 100):

- The exception hierarchy's parent switched from `Exception` to `RuntimeException` — making the whole hierarchy unchecked in one edit, and letting every `throws` declaration in `NetworkClientV4` simply disappear.
- `NetworkServiceV4` stopped catching `ConnectExceptionV4`/`SendExceptionV4` entirely — since it can't recover from either, it lets both propagate automatically, keeping only its own `finally { client.disconnect(); }` cleanup responsibility.
- `MainV4` added exactly **one** common exception-handling point: `catch (Exception e) { exceptionHandler(e); }`, where `exceptionHandler` prints a generic, safe message for the user, logs the full stack trace for developers (`e.printStackTrace(System.out)`), and optionally layers on type-specific extra logging via `instanceof` without abandoning the shared handling path.

> "중요한 거 내가 해결할 수 있는 건 중간에 해결하고, 해결할 수 없는 거는 그냥 공통으로 다 처리가 될 거예요. 뭔가 다른 로직이 되게 많아져도 내가 신경 쓰는 것만 쓰고, 나머지는 막 로직이 늘어나더라도 어차피 공통 예외 처리가 있으니까... 이렇게 큰 틀을 잡아주시는 것을 추천드립니다."

### A clarifying note on checked exceptions: they're not banned outright

> "이렇게 한다고 하면 체크 예외를 사용하면 안 되냐? 그게 아니에요. 진짜 중요한데 가끔 체크 예외를 쓰시면 되고, 대부분의 내가 복구할 수 없는 예외는 그냥 런타임으로 두시면 됩니다."

- The recommendation isn't "never use checked exceptions" — it's that checked exceptions remain a reasonable, occasional choice specifically when a failure is both **critically important** *and* **actually recoverable** by the calling code (the instructor's example: a core business-critical operation like a money transfer, where the caller is expected to catch and definitely handle the failure).
- For the much larger, more common category — failures the calling code genuinely cannot act on — unchecked exceptions paired with common handling is the recommended default.
- The instructor adds a candid aside: even some of those rare "important and recoverable" cases might still be better off as runtime exceptions in practice — this is ultimately a judgment call ("선택"), not a hard rule.

### 7. try-with-resources (Lecture 101) — a late addition to an earlier problem

> "Try with resources, 이거는 사실 finally 다음에 설명하기 더 좋은데, 제가 그렇게 하려다 보니까 이게 흐름이 조금 깨지는 게 있더라고요. 그래서 제가 이걸 마지막에 가져왔고요."

- Conceptually, `try`-with-resources belongs right alongside `finally` (Lecture 96) — it solves the exact same resource-cleanup problem, just more cleanly. It was deliberately placed last in the sequence purely to avoid breaking the narrative flow of the running example.
- Mechanics recap: a class implementing `AutoCloseable` can be declared inside `try (...)`; its `close()` is invoked automatically the instant the `try` block exits — before any matching `catch` runs, and without needing an explicit `finally` at all.
- A pointed clarifying question from the transcript: what happens if a type that does **not** implement `AutoCloseable` is placed inside `try (...)`? Answer: **a compile error** — the `try (...)` header specifically requires an `AutoCloseable`-typed value; nothing else is accepted there.

> "auto-closable이 없으면 어떻게 되나요? ... 컴파일 오류 납니다. 이거 타입이 auto-closable 타입이 들어와야 돼 라고 되어 있죠. ... auto-closable 반드시 있어야 됩니다. 아니면 컴파일 오류 납니다."

- Advantages recapped one final time: resource-leak prevention, more concise code, a narrower (and therefore easier-to-debug) scope for the resource variable, and faster resource release than `finally` provides (since `close()` fires before `catch` even runs).

---

## The Instructor's Closing Note

> "여러분이 사실 이 정도 딱 이해하시면 여러분 실무에서도, 아니면 지금 실무를 하고 계신 분들도 '아, 이런 스타일로 예외 처리를 하면 되겠구나'라고 딱 감을 잡을 수 있을 거예요. ... 이렇게 제가 실무 트렌드까지 반영해서 최신인, 요즘엔 어떻게 하는 게 베스트 프랙티스다까지 이 전체 흐름을 한 번 딱 잡아드리고 싶었어요."

- The explicit goal of the whole section was to leave the learner with a working mental model of *why* checked vs. unchecked exceptions exist, not just the syntax — specifically so that, faced with exceptions in a real codebase, the instinct is "which category does this belong to, and where's the common-handling boundary" rather than reflexively scattering `try`/`catch` everywhere.
- The instructor frames this as intentionally going a bit beyond "just syntax" because this is an intermediate-level ("중급") course — current industry best practice (runtime exceptions + common handling), not just the language rules, was treated as in-scope material.

---

## Summary

- This lecture is a pure retrospective with no new code — it deliberately skips the usual "문제와 풀이" segment in favor of the learner replaying the whole section's example progression themselves.
- The exception hierarchy's one essential rule: everything under `Exception` that doesn't extend `RuntimeException` is checked (must be declared via `throws` if not caught, or the code won't compile); everything extending `RuntimeException` is unchecked (propagates automatically without any `throws` declaration) — that distinction is the *entire* mechanical difference between the two categories.
- The arc, recapped: return-value error codes (mixed normal/exception flow) → real `try`/`catch` with a single exception + error code → `finally` for guaranteed resource cleanup → exception hierarchies for layered, precise `catch`-ing → the practical plan (unchecked exceptions + one common exception-handling point) → `try`-with-resources as a cleaner, faster alternative to `finally`.
- The practical recommendation isn't "never use checked exceptions" — it's that checked exceptions remain reasonable for failures that are both critically important *and* genuinely recoverable by the caller; everything else (the large majority, in practice) is better left as an unchecked exception, handled either locally when meaningful or left to propagate to one common handler.
- `throws Exception` as a blanket workaround for checked-exception fatigue is explicitly called out as something that should never be done — it silently defeats the compiler's ability to flag any new, genuinely important checked exception introduced later.
- `try`-with-resources' core mechanical requirement: the resource type in `try (...)` must implement `AutoCloseable`, or the code fails to compile — there's no fallback behavior for a non-`AutoCloseable` type placed there.
- This closes out Section 11 and the exception-handling arc that began in Section 10: the goal throughout was to build an intuition for *which* kind of exception to reach for and *where* unrecoverable failures should ultimately be resolved — not just the `try`/`catch`/`finally`/`throws` syntax itself.
