# Lecture 98 — Exception Handling 2 - Practice: Exception Hierarchy - Usage

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (7/11)

Previous: [Lecture 97 - Exception Hierarchy - Getting Started](Lecture%2097%20-%20Exception%20Hierarchy%20-%20Getting%20Started.md)

## The Problem: Too Many Exceptions to Catch One-By-One

`NetworkClientV3` currently has two possible exceptions, but imagine many more — ten, say. Writing ten separate `catch` blocks for ten exception types would get unmanageable fast. This lecture shows how to use the hierarchy from Lecture 97 to **catch at different levels of specificity**, grouping less-important exceptions together instead of handling every single one individually.

## The Plan

A three-tier policy, in order of importance:

1. **Connection errors are critical.** If `ConnectExceptionV3` occurs, log a clear, specific message (using its `address`).
2. **Everything else from `NetworkClient` gets a generic treatment.** Any other exception that's a child of `NetworkClientExceptionV3` (e.g. `SendExceptionV3`, or any future exception added to this hierarchy) just logs a plain "network error" message — treated as less severe than a connection failure.
3. **Anything completely unanticipated still gets logged**, as a last-resort fallback.

Copy `NetworkServiceV3_1` forward to `NetworkServiceV3_2`.

---

## `NetworkServiceV3_2`: Three `catch` Blocks, Three Levels of Specificity

```java
package exception.ex3;

import exception.ex3.exception.ConnectExceptionV3;
import exception.ex3.exception.NetworkClientExceptionV3;
import exception.ex3.exception.SendExceptionV3;

public class NetworkServiceV3_2 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV3 client = new NetworkClientV3(address);
        client.initError(data); //추가

        try {
            client.connect();
            client.send(data);
        } catch (ConnectExceptionV3 e) {
            System.out.println("[연결 오류] 주소: " + e.getAddress() + ", 메시지: " + e.getMessage());
        } catch (NetworkClientExceptionV3 e) {
            System.out.println("[네트워크 오류] 메시지: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("[알 수 없는 오류] 메시지: " + e.getMessage());
        } finally {
            client.disconnect();
        }

    }
}
```

- `catch (ConnectExceptionV3 e)`: the specific, high-priority case — logs the `address` alongside the message, same as Lecture 97.
- `catch (NetworkClientExceptionV3 e)`: a catch-all for "any other `NetworkClient` exception." Since `SendExceptionV3` (and any sibling exception not yet written) is a child of `NetworkClientExceptionV3`, this single `catch` handles all of them with one generic message — no need for a dedicated `catch` per sibling exception.
- `catch (Exception e)`: the final fallback for anything completely unrelated to this hierarchy — a true catch-all, since `Exception` sits above everything.
- `finally { client.disconnect(); }`: unchanged, carried over from Lecture 96.

> "여기서는 어마어마하게 많은 예외가 터지는데, 안에서. 일단 이거 중요한 거 내가 남기고, 너무 많으니까 좀 퉁 쳐서 여기서 이제 나머지 쫙 잡아주고, 그다음에 이걸로도 해결이 안 되는 건 여기서 잡고, 요렇게 한 겁니다."

### Why `NetworkClientExceptionV3` still catches `SendExceptionV3`

Same polymorphism rule from Lecture 90/97: `NetworkClientExceptionV3` is `SendExceptionV3`'s parent, so a `catch` targeting the parent also matches the child. One `catch (NetworkClientExceptionV3 e)` transparently absorbs every current and future sibling exception under it — without needing to list each one individually.

---

## `catch` Blocks Are Tried in Order — First Match Wins

Each thrown exception is matched against the `catch` clauses **top to bottom**, stopping at the first one that fits:

| Exception thrown | 1st `catch` (`ConnectExceptionV3`) | 2nd `catch` (`NetworkClientExceptionV3`) | 3rd `catch` (`Exception`) |
|---|---|---|---|
| `ConnectExceptionV3` | ✅ matches here, stops | (not reached) | (not reached) |
| `SendExceptionV3` | ❌ not a match (sibling, not a subtype) | ✅ matches here, stops | (not reached) |
| `RuntimeException` | ❌ not a match | ❌ not a match | ✅ matches here (`RuntimeException`'s ancestor is `Exception`) |

### Run (`error1`, `error2`)

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: error1
[연결 오류] 주소: http://example.com, 메시지: http://example.com 서버 연결 실패
http://example.com 서버 연결 해제

전송할 문자: error2
http://example.com 서버 연결 성공
[네트워크 오류] 메시지: http://example.com 서버에 데이터 전송 실패: error2
http://example.com 서버 연결 해제

전송할 문자: exit
프로그램을 정상 종료합니다.
```

- `error1`: `ConnectExceptionV3` is thrown — matches the **first** `catch` directly.
- `error2`: `SendExceptionV3` is thrown — not a match for `ConnectExceptionV3` (they're siblings, neither is a subtype of the other), but it **is** a `NetworkClientExceptionV3`, so the second `catch` picks it up, printing the generic `"[네트워크 오류]"` message.

### Confirming the fallback with an uncaught type

Temporarily throwing a plain `RuntimeException` instead of `SendExceptionV3` inside `send(...)`:

- It isn't a `ConnectExceptionV3` or a `NetworkClientExceptionV3` — but it **is** an `Exception` (every exception used normally descends from `Exception`), so the third `catch (Exception e)` picks it up, printing `"[알 수 없는 오류] 메시지: ex"`.
- This confirms `Exception` as a true last-resort fallback: anything that reaches this point and isn't already matched gets caught here, because `Exception` sits above virtually everything an application would normally throw.

The code is restored to its working state (`SendExceptionV3` thrown again) afterward.

---

## Order Matters — and the Compiler Enforces It

> "이 순서를 바꾸면 안 됩니다. 디테일한 자식을 먼저 잡아야 됩니다."

If `catch (NetworkClientExceptionV3 e)` were placed **before** `catch (ConnectExceptionV3 e)`:

```java
// Hypothetical — does not compile
} catch (NetworkClientExceptionV3 e) {
    ...
} catch (ConnectExceptionV3 e) {   // compile error
    ...
}
```

- Since `catch` is evaluated top to bottom and `ConnectExceptionV3` is a subtype of `NetworkClientExceptionV3`, the broader `catch (NetworkClientExceptionV3 e)` would *always* match first — the more specific `catch (ConnectExceptionV3 e)` beneath it could **never** actually run.
- Java's compiler catches this directly: it's a **compile error** ("already caught by the alternative"), not a silent bug. The more specific exception type must always be listed *before* its broader ancestor.
- The ordering rule, generalized: **list more specific (child) exception types first, broader (parent) types further down**, ending with `Exception` as the final catch-all if one is needed.

---

## Catching Several Sibling Types in One `catch`

Java also supports combining multiple unrelated exception types in a single `catch` using `|`:

```java
try {
    client.connect();
    client.send(data);
} catch (ConnectExceptionV3 | SendExceptionV3 e) {
    System.out.println("[연결 또는 전송 오류] 메시지: " + e.getMessage());
} finally {
    client.disconnect();
}
```

- Handles both `ConnectExceptionV3` and `SendExceptionV3` with identical logic in one block, instead of duplicating the same handling code across two separate `catch` clauses.
- **The trade-off**: inside this combined `catch`, `e`'s type is effectively narrowed to whatever the two types have in common — their nearest shared ancestor. `e.getAddress()` (only on `ConnectExceptionV3`) and `e.getSendData()` (only on `SendExceptionV3`) are **not** accessible here, since the compiler can't know which of the two actually occurred. Only methods available on their common parent, `NetworkClientExceptionV3` (here, just `getMessage()`), can be called.
- Use this form when the handling logic is genuinely identical across the combined types and none of their type-specific methods are needed; otherwise, prefer separate `catch` blocks to retain access to each exception's own data.

---

## Trade-offs of Hierarchy Design

> "예외를 계층화하고 다양하게 만들면 더 세밀한 동작들을 깔끔하게 처리할 수 있습니다. 그리고 특정 분류의 공통 예외들도 한 번에 catch로 잡아서 처리할 수가 있습니다."

- This isn't free, though — there's a genuine trade-off between how many exception classes to create.
  - Too few: reaching back for a single generic exception with an error-code field (Lecture 92's approach), losing the benefits of per-type data and precise catching.
  - Too many: an overwhelming number of tiny, barely-differentiated exception classes, which gets just as hard to navigate.
- The instructor's practical guidance: **hierarchy where it clearly adds value, and a looser/simpler approach (shared exception + logic) where exact precision isn't worth the extra classes.** There's no single universally "right" number — judgment calls based on the actual variety of failure modes in a given system.

---

## Summary

- `NetworkServiceV3_2` uses three `catch` blocks of decreasing specificity: `ConnectExceptionV3` (specific, high-priority), `NetworkClientExceptionV3` (catches any other sibling exception under it, like `SendExceptionV3`), and `Exception` (true fallback for anything unanticipated).
- `catch` clauses are evaluated top-to-bottom, and the **first type that matches wins** — so more specific (child) exception types must be listed before broader (parent) ones, or the code won't even compile (Java flags the unreachable `catch` as a compile error).
- Catching a broader parent type (`NetworkClientExceptionV3`) absorbs every current and future subtype beneath it without needing a dedicated `catch` for each one — the core payoff of building a hierarchy in the first place.
- `catch (TypeA | TypeB e)` lets a single block handle multiple unrelated sibling types identically, at the cost of only being able to call methods available on their common ancestor type — not either type's own specific methods.
- Exception hierarchy design is a genuine trade-off: too few exception types loses precision and per-type data; too many becomes unwieldy. A sensible middle ground — hierarchy where it clearly helps, simpler shared types elsewhere — tends to work best in practice.
- This closes out the theoretical/architectural side of exception design for this running example — the next lectures turn to real-world, practical exception-handling strategy.
