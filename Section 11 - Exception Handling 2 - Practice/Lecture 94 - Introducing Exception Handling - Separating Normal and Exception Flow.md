# Lecture 94 — Exception Handling 2 - Practice: Introducing Exception Handling - Separating Normal and Exception Flow

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (3/11)

Previous: [Lecture 93 - Introducing Exception Handling - Exception Recovery](Lecture%2093%20-%20Introducing%20Exception%20Handling%20-%20Exception%20Recovery.md)

## The Payoff Lecture: Using `try`/`catch` Properly

`NetworkServiceV2_2` recovered from exceptions, but still had **two separate `try`/`catch` blocks** interleaved with the normal flow — the exact readability problem the whole section exists to solve wasn't actually fixed yet.

This lecture fixes it by using `try`/`catch` the way it's meant to be used: **one `try` block containing the entire normal flow**, with **one `catch` block handling every error path**.

Copy `NetworkServiceV2_2` forward to `NetworkServiceV2_3`.

---

## `NetworkServiceV2_3`: One `try`, One `catch`

```java
package exception.ex2;

public class NetworkServiceV2_3 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV2 client = new NetworkClientV2(address);
        client.initError(data); //추가

        try {
            client.connect();
            client.send(data);
            client.disconnect();
        } catch (NetworkClientExceptionV2 e) {
            System.out.println("[오류] 코드: " + e.getErrorCode() + ", 메시지: " + e.getMessage());
        }
    }
}
```

- The two separate `try`/`catch` pairs from `NetworkServiceV2_2` collapse into **one `try` block that holds all three calls** — `connect()`, `send(data)`, `disconnect()` — back to back, exactly as clean as the very first version of this example (`NetworkServiceV0`).
- A single `catch (NetworkClientExceptionV2 e)` underneath handles whichever of those three calls actually threw.
- The `return` statements from Lecture 93 are gone — no longer needed, since there's nothing after the `catch` block to skip past.
- Reading this method now takes two glances, not a line-by-line trace: look at the `try` block to see the entire normal flow in order, look at the `catch` block to see how errors are handled. Nothing is interleaved.

> "하나의 try 안에 정상 흐름을 모두 담습니다. 그리고 예외 부분은 catch block에서 해결을 합니다. 이렇게 하면 정상 흐름은 try block 안에 들어가고 예외 흐름은 catch block 안으로 들어가서 굉장히 명확하게 분리할 수 있어요."

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

- `error1`: `connect()` throws inside the `try` block. Control jumps straight to the `catch` block — `send()` and `disconnect()` on the lines after `connect()` never execute, same end result as before, but now it's the single `catch` doing the work rather than an early `return`.
- `error2`: `connect()` succeeds, `send(data)` throws — same jump to `catch`, skipping `disconnect()`.
- Either way, after the `catch` block finishes, `sendMessage(...)` returns normally and the outer `while` loop in `MainV2` keeps accepting new input.

---

## Problem Solved: Normal Flow and Error Flow Are Finally Separated

> "자바의 예외 처리 메커니즘과 try, catch 구조 덕분에 정상 흐름은 try 블럭에 모아서 처리하고, 예외 흐름은 catch 블럭에 별도로 모아서 처리할 수 있었다. 덕분에 정상 흐름과 예외 흐름을 명확하게 분리해서 코드를 더 쉽게 읽을 수 있게 되었다."

- This is the structural payoff the entire Section 10 theory build-up (and Lectures 92–93's intermediate steps) was working toward.
- Compare this `NetworkServiceV2_3` directly to `NetworkServiceV1_3` from Lecture 87 — same requirements satisfied (error logged, send-after-failed-connect prevented), but `NetworkServiceV1_3`'s `if`/`else` branching constantly interleaved normal and error logic, while this version reads as two clean, physically separate blocks.

---

## Remaining Problem: `disconnect()` Is Still Not Guaranteed

One problem is still outstanding, and this lecture's structure makes the issue very visible:

- Looking at the `try` block, `client.disconnect()` is the **last statement inside it**. If an exception is thrown by `connect()` or `send()` earlier in the same `try` block, execution jumps directly to `catch` — it never reaches `disconnect()` on the line below.
- The `error1`/`error2` runs above confirm this: neither prints `"서버 연결 해제"`.
- Same underlying issue flagged back in Lecture 87 (return-value version) and Lecture 93 (two-try-block version): skipping `disconnect()` leaks the external connection resource, since Java can't automatically reclaim resources outside the JVM.

> "이 구조에서 딱 문제가 하나가 있는 게, 여기서 예외가 빵 터져요. 그럼 어디로 가죠? [catch로] 가요." — explaining exactly why `disconnect()` gets skipped: once an exception fires anywhere inside the `try`, control leaves the `try` block entirely and jumps to `catch`, so nothing further down in the `try` ever runs.

This is exactly what the next lecture resolves — using a dedicated language feature made for "this must always run, exception or not."

---

## Summary

- `NetworkServiceV2_3` collapses the two separate `try`/`catch` pairs from `NetworkServiceV2_2` into a single `try` block holding the full normal flow (`connect → send → disconnect`) and a single `catch` block handling any error from any of those three calls.
- This finally achieves what the whole exception-handling section was building toward: normal flow and error-handling flow are visually and structurally separated — one glance at `try` shows the happy path, one glance at `catch` shows the error handling.
- The run output is functionally identical to Lecture 93's (error logged with code + message, program keeps running afterward) but the code producing it is dramatically easier to read.
- One problem remains: `client.disconnect()` sits as the last line inside the `try` block, so any exception thrown earlier in that same block (by `connect()` or `send()`) skips it entirely by jumping straight to `catch` — meaning the external connection resource is still never released on an error path. The next lecture addresses this directly.
