# Lecture 93 — Exception Handling 2 - Practice: Introducing Exception Handling - Exception Recovery

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (2/11)

Previous: [Lecture 92 - Introducing Exception Handling - Getting Started](Lecture%2092%20-%20Introducing%20Exception%20Handling%20-%20Getting%20Started.md)

## This Time: Actually Recover From the Exception

Lecture 92 left `NetworkServiceV2_1` just propagating the checked `NetworkClientExceptionV2` all the way up — any error crashed the whole program. This lecture catches it and brings the flow back to normal, the "defuse the bomb" mechanic from Lecture 90.

Copy `NetworkServiceV2_1` forward to a new `NetworkServiceV2_2`.

---

## `NetworkServiceV2_2`: Catch Each Risky Call

```java
package exception.ex2;

public class NetworkServiceV2_2 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV2 client = new NetworkClientV2(address);
        client.initError(data); //추가

        try {
            client.connect();
        } catch (NetworkClientExceptionV2 e) {
            System.out.println("[오류] 코드: " + e.getErrorCode() + ", 메시지: " + e.getMessage());
            return;
        }

        try {
            client.send(data);
        } catch (NetworkClientExceptionV2 e) {
            System.out.println("[오류] 코드: " + e.getErrorCode() + ", 메시지: " + e.getMessage());
            return;
        }

        client.disconnect();
    }
}
```

- Each place that can throw — `connect()` and `send(data)` — gets its own `try { ... } catch (NetworkClientExceptionV2 e) { ... }` block.
- Inside each `catch`, log both pieces of information stored in the exception: `e.getErrorCode()` (the custom field from Lecture 92) and `e.getMessage()` (inherited from `Throwable`, set via `super(message)`).
- `return` after logging: just like `connect()` failing shouldn't let `send()` run in the return-value version (Lecture 87), the same rule applies here — if `connect()`'s `catch` fires, the method returns immediately, skipping `send()` entirely.
- `sendMessage(data)` itself no longer declares `throws` at all — every exception that could occur inside it is now fully caught, so there's nothing left to propagate.

> IntelliJ's Alt-Enter quick-fix on an unhandled checked exception offers exactly two choices: add a `throws` declaration, or wrap it in `try`/`catch`. This lecture picks the catch option for both calls.

---

## Run

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: error1
[오류] 코드: connectError, 메시지: http://example.com 서버 연결 실패

전송할 문자: error2
http://example.com 서버 연결 성공
[오류] 코드: sendError, 메시지: http://example.com 서버에 데이터 전송 실패: error2

전송할 문자: exit
프로그램을 정상 종료합니다.
```

Walking through `error1`:

1. `connect()` detonates `NetworkClientExceptionV2`.
2. The first `try`'s `catch` intercepts it — logs the code/message, then `return`s. The bomb is defused right there.
3. Execution returns normally from `sendMessage(...)`, back to the `while` loop in `MainV2`, which prompts for input again.

Walking through `error2`: `connect()` succeeds (flow never enters the first `catch`), `send(data)` throws, the second `catch` intercepts it, logs, and returns — skipping `disconnect()` at the bottom.

---

## Problem Solved: the Program No Longer Crashes

> "예외를 잡아서 처리했다. 따라서 예외가 복구되고, 프로그램도 계속 수행할 수 있다."

- Compare directly to Lecture 92's `error1`/`error2` runs, which printed a stack trace and **terminated the process**.
- Here, after an error the loop keeps running and accepts more input — `hello`, `error1`, `error2`, `exit` can all be typed in the same session without the program ever dying.
- This is exactly what "recovering" an exception means in practice: catching it turns the error back into normal control flow, instead of letting it keep propagating until it kills the whole program.

---

## Remaining Problems

Two problems are still open, and this lecture's own code makes the second one especially visible:

1. **Normal flow and error-handling flow are still interleaved.** There are now two separate `try`/`catch` pairs sitting between the three steps of the "real" normal flow (`connect` → `send` → `disconnect`) — reading this method still means mentally stepping through alternating normal/error blocks, not so different in spirit from the return-value version this is meant to replace.
2. **`disconnect()` still isn't guaranteed to run.** Look at the `error1`/`error2` output above: neither one prints `"서버 연결 해제"`. Both `catch` blocks `return` before reaching `client.disconnect()` at the bottom of the method — so a connection established before `send()` failed is never explicitly closed.

> "분명히 이런 것 때문에 우리가 예외 처리를 배운 것 같은데?" — the instructor flagging that this is exactly the gap Java's exception handling is supposed to close, and that it isn't closed yet.

Both problems are picked up in the next lectures.

---

## Summary

- `NetworkServiceV2_2` wraps `connect()` and `send(data)` each in their own `try`/`catch (NetworkClientExceptionV2 e)`, logging `e.getErrorCode()` + `e.getMessage()` and `return`ing immediately on failure.
- Catching the exception and `return`ing is what "recovers" the flow — once caught, execution resumes normally from right after the catch, instead of propagating further and eventually crashing the program.
- Running `error1`/`error2` now logs a clean one-line error message and the program keeps accepting new input afterward — a direct improvement over Lecture 92, where the same inputs printed a stack trace and killed the process.
- Two problems remain: normal flow and error flow are still visually interleaved across two separate `try`/`catch` blocks, and `disconnect()` is skipped on every error path (both `catch` blocks `return` before reaching it) — exactly the two things the next lectures fix using `try`/`catch` properly and, eventually, `finally`.
