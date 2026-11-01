# Lecture 87 — Exception Handling 1 - Theory: Need for Exception Handling - Handling Exceptions with Return Values

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 10. 예외 처리1 - 이론 (3/7)

Previous: [Lecture 86 - Need for Exception Handling - Creating Error Situations](Lecture%2086%20-%20Need%20for%20Exception%20Handling%20-%20Creating%20Error%20Situations.md)

## The Two Problems to Fix

Carried over from Lecture 86:

1. If `connect()` fails, `send()` still gets called anyway — it shouldn't.
2. No detailed error logging when something fails.

`NetworkClientV1.connect()` / `send()` already return a String result code (`"success"`, `"connectError"`, `"sendError"`). This lecture uses that return value to actually fix both problems — step by step, in three successive versions of `NetworkService`.

---

## V1_2: Check the Result, Log, and Stop

```java
package exception.ex1;

public class NetworkServiceV1_2 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV1 client = new NetworkClientV1(address);
        client.initError(data); //추가

        String connectResult = client.connect();
        if (isError(connectResult)) {
            System.out.println("[네트워크 오류 발생] 오류 코드: " + connectResult);
            return;
        }

        String sendResult = client.send(data);
        if (isError(sendResult)) {
            System.out.println("[네트워크 오류 발생] 오류 코드: " + sendResult);
            return;
        }

        client.disconnect();
    }

    private static boolean isError(String connectResult) {
        return !connectResult.equals("success");
    }
}
```

- Capture `connect()`'s return value as `connectResult`, check it, and if it isn't `"success"` — log a detailed `[네트워크 오류 발생] 오류 코드: ...` message and `return` immediately, **before** calling `send()`.
- Same pattern for `send()`'s result: log and `return` immediately if it failed, before reaching `disconnect()`.
- This fixes both problems from the top of this lecture: the log now shows the actual error code, and a failed `connect()` now genuinely stops `send()` from ever running (early `return` exits the method).

### Why `isError(...)` instead of `!connectResult.equals("success")` inline

The instructor extracted this as a dedicated method (via IntelliJ's Extract Method, `⌃⌥M` on Mac / `Ctrl+Alt+M` on Windows) and it's worth understanding *why*, not just copying it:

- `if (!connectResult.equals("success"))` forces the reader to negate a *success* check in their head to realize it means "this is an error." Negation is mentally expensive — the brain has to flip it.
- `if (isError(connectResult))` reads as a direct statement of intent: "if this is an error." No mental flip required.
- A good method name lets you compress a small, slightly-awkward expression into something immediately legible — a technique the instructor says he personally favors a lot.

> "결과가 성공이 아니다라는 건가? 성공을 뒤집는 게 쉬워요, 아니면 이게 오류다라는 게 쉬워요?" — on why `isError(...)` beats `!result.equals("success")`.

### Run

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: error1
http://example.com 서버 연결 실패
[네트워크 오류 발생] 오류 코드: connectError

전송할 문자: error2
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송 실패: error2
[네트워크 오류 발생] 오류 코드: sendError

전송할 문자: exit
프로그램을 정상 종료합니다.
```

Compare to Lecture 86's `error1` output — there's no longer a stray `서버에 데이터 전송` line after the connection failure. Problem 1 (send-after-failed-connect) and problem 2 (no error logging) are both fixed.

---

## A New Problem: `disconnect()` Doesn't Always Run

Going back to the original usage rules for `NetworkClient`:

| Rule | Status after V1_2 |
|---|---|
| Don't call `send()` if `connect()` failed | ✅ Fixed |
| Always call `disconnect()`, even if `connect()`/`send()` errored | ❌ Still broken |

Look at the `error2` run above: `connect()` succeeds (a connection gets established), `send()` fails, and the method returns immediately — **`disconnect()` never runs**. The connection is left open.

### Why this matters — resource leaks

- Java's garbage collector can automatically clean up in-JVM-memory instances, but **external resources** (network connections, file handles, etc.) are not managed by the GC — they must be explicitly released.
- If `disconnect()` is skipped every time `send()` fails, the external connection resource is never returned. Each subsequent failed send leaks another connection.
- Left unchecked, this kind of leak is exactly the kind of bug that doesn't show up on day one or day two — it creeps in over days/weeks until the system runs out of connections and starts failing mysteriously. The instructor recalls this being common enough in the past that teams would routinely restart servers before the weekend just to clear accumulated leaks, before tooling got better at catching this.

> "자바가 자동으로 해체할 수 있는 것은 인스턴스 같은 것을 자동으로 해체할 수 있는 거지, 자바 외부에 있는 자원은 자동으로 해체가 되지 않습니다."

The requirement, restated: **no matter what happens in `connect()`/`send()`, `disconnect()` must always run.**

---

## V1_3: Guarantee `disconnect()` Always Runs

```java
package exception.ex1;

public class NetworkServiceV1_3 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV1 client = new NetworkClientV1(address);
        client.initError(data); //추가

        String connectResult = client.connect();
        if (isError(connectResult)) {
            System.out.println("[네트워크 오류 발생] 오류 코드: " + connectResult);
        } else {
            String sendResult = client.send(data);
            if (isError(sendResult)) {
                System.out.println("[네트워크 오류 발생] 오류 코드: " + sendResult);
            }
        }

        client.disconnect();
    }

    private static boolean isError(String connectResult) {
        return !connectResult.equals("success");
    }
}
```

- The two early `return` statements are gone, replaced with an `if (isError(...)) { ... } else { ... }` branch structure.
- `send()` is now nested **inside the `else`** — it only runs when `connect()` did *not* error.
- Neither branch of the `if/else` returns out of the method, so control always falls through to `client.disconnect()` at the very end — whether `connect()` failed, `send()` failed, or everything succeeded.
- This is a deliberate trade: instead of "exit early on error," the logic branches so that both the error path and the success path converge back to the same final step.

### Run

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: error1
http://example.com 서버 연결 실패
[네트워크 오류 발생] 오류 코드: connectError
http://example.com 서버 연결 해제

전송할 문자: error2
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송 실패: error2
[네트워크 오류 발생] 오류 코드: sendError
http://example.com 서버 연결 해제

전송할 문자: exit
프로그램을 정상 종료합니다.
```

Now `서버 연결 해제` (disconnect) prints on *every* path — success, connect failure, and send failure alike.

| Usage rule | V1_2 | V1_3 |
|---|---|---|
| Don't call `send()` after a failed `connect()` | ✅ | ✅ |
| Always call `disconnect()` regardless of errors | ❌ | ✅ |

Both original usage rules are now satisfied.

---

## The Real Problem: Normal Flow and Error-Handling Flow Are Tangled Together

This is the key point the whole section has been building toward. Looking at `NetworkServiceV1_2` and `NetworkServiceV1_3`, both are genuinely hard to read.

**What the "normal flow" alone looks like**, if nothing could ever fail:

```java
client.connect();
client.send(data);
client.disconnect();
```

- Simple. Intuitive. Instantly understandable.

**But once error handling is added using return values**, this clean 3-line shape disappears — it gets interleaved with checks:

- `connect()` → check result → log-and-stop-or-continue → `send()` → check result → log-and-stop-or-continue → `disconnect()`.
- In V1_3 it's even more tangled: the error-handling `if/else` branches now *contain* the next step of the normal flow (`send()` lives inside the `else`).
- The reader has to mentally pick apart "is this line part of the success path or the error path?" at every step — and in V1_3, the answer keeps flipping.

> "정상 흐름과 예외 흐름이 섞여 있기 때문에 코드를 한눈에 이해하기 어렵다. 쉽게 얘기해서 가장 중요한 정상 흐름이 한눈에 들어오지 않는다. 심지어 예외 흐름이 더 많은 코드 분량을 차지한다."

### Why return-value-based error handling can never fix this

- The moment a method's return value is used to decide "did this succeed or fail," that check *is* the normal-flow/error-flow entanglement — there's no way around it.
- Call → get a return value → branch on it: that branching line is simultaneously part of the normal flow (what comes next) and the error flow (what if it failed). They cannot be cleanly separated as long as the mechanism for reporting failure is "a regular value returned from a regular method call."
- Every additional call that needs its own result check (`connect()`, then `send()`) compounds the problem — V1_3 has two separate branch points nested into each other.

This is exactly the motivation for Java's actual exception-handling mechanism (`try` / `catch` / `throw`, etc.), which the course picks up starting next lecture — it exists specifically to let the normal flow be written on its own, cleanly, with error handling pulled out to a separate, clearly-marked part of the code.

---

## Summary

- `NetworkServiceV1_2` uses `connect()`/`send()`'s String return values to detect failure, log a detailed error message, and `return` immediately — fixing "don't send after a failed connect" and "log errors with detail."
- `isError(resultCode)` is preferred over an inline `!resultCode.equals("success")` check purely for readability — naming a boolean condition removes the mental effort of interpreting a negation.
- V1_2 leaves a new problem: an early `return` on failure means `disconnect()` is skipped, which leaks the external connection resource — a real risk, since unlike JVM-managed memory, external resources are never auto-cleaned by the garbage collector.
- `NetworkServiceV1_3` fixes this by replacing the early `return`s with `if (isError) { ... } else { send... } ` branching, so execution always falls through to a single `disconnect()` call at the end, no matter which path was taken.
- With V1_3, both original `NetworkClient` usage rules are finally satisfied — but the code itself has become harder to read, because the normal flow (connect → send → disconnect) and the error-handling logic are now fully interleaved, with error handling taking up *more* code than the normal flow it's protecting.
- Return-value-based error handling is structurally incapable of separating normal flow from error flow — any branch on a return code mixes the two by definition. This limitation is the reason Java provides a dedicated exception-handling mechanism, which the next lecture introduces.
