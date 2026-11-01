# Lecture 96 — Exception Handling 2 - Practice: Introducing Exception Handling - finally

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (5/11)

Previous: [Lecture 95 - Introducing Exception Handling - The Resource Return Problem](Lecture%2095%20-%20Introducing%20Exception%20Handling%20-%20The%20Resource%20Return%20Problem.md)

## A Block Guaranteed to Always Run

Lecture 95 ended with no clean way to guarantee `disconnect()` always runs using `try`/`catch` alone. Java's actual answer: **`finally`**.

```java
try {
    //정상 흐름
} catch (...) {
    //예외 흐름
} finally {
    //반드시 호출해야 하는 마무리 흐름
}
```

- `try` → `catch` → `finally` together provide three distinct flows: normal flow, exception flow, and a **mandatory wrap-up flow**.
- As long as `try` is entered at all, the `finally` block is **guaranteed to run, no matter what** — regardless of how the `try`/`catch` finishes.

### All three ways `finally` still runs

| What happens in `try`/`catch` | Then `finally` runs |
|---|---|
| Normal flow completes without error | ✅ |
| An exception is thrown and `catch` handles it | ✅ (after `catch` finishes) |
| An exception is thrown that `catch` **can't** match, so it propagates out | ✅ (`finally` runs first, *then* the exception continues propagating outward) |

> "finally 코드 블럭은 어떤 경우라도 반드시 호출됩니다. 심지어 try, catch 안에서 잡을 수 없는 예외가 터져서 밖으로 나가도 finally는 반드시 호출돼요."

Because of this guarantee, `finally` is the natural place for **resource cleanup** — exactly the `disconnect()` problem this whole arc of lectures has been chasing.

---

## `NetworkServiceV2_5`: Move `disconnect()` Into `finally`

Copy `NetworkServiceV2_4` forward to `NetworkServiceV2_5`, add a `finally` block to the existing `try`/`catch`, and move `client.disconnect()` inside it:

```java
package exception.ex2;

public class NetworkServiceV2_5 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV2 client = new NetworkClientV2(address);
        client.initError(data); //추가

        try {
            client.connect();
            client.send(data);
        } catch (NetworkClientExceptionV2 e) {
            System.out.println("[오류] 코드: " + e.getErrorCode() + ", 메시지: " + e.getMessage());
        } finally {
            client.disconnect();
        }

    }
}
```

- Just one small structural move from `NetworkServiceV2_4`: `client.disconnect()` moves from after the `try`/`catch` block to inside a new `finally` block.
- The normal flow (`try`), the error flow (`catch`), and the cleanup flow (`finally`) are now each visually distinct and physically separated — three clean sections instead of two.

### Run

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: error1
[오류] 코드: connectError, 메시지: http://example.com 서버 연결 실패
http://example.com 서버 연결 해제

전송할 문자: error2
http://example.com 서버 연결 성공
[오류] 코드: sendError, 메시지: http://example.com 서버에 데이터 전송 실패: error2
http://example.com 서버 연결 해제

전송할 문자: exit
프로그램을 정상 종료합니다.
```

- `hello`: `try` completes normally → `finally` runs → `"서버 연결 해제"` prints.
- `error1`: `connect()` throws → `catch` handles it and logs → `finally` runs → `"서버 연결 해제"` prints.
- `error2`: `connect()` succeeds, `send()` throws → `catch` handles it and logs → `finally` runs → `"서버 연결 해제"` prints.

---

## Confirming the Hard Case: an Uncaught Exception Still Triggers `finally`

Same test as Lecture 95 — temporarily swap `send(...)`'s thrown exception for an unrelated, uncatchable `RuntimeException`:

```java
public void send(String data) throws NetworkClientExceptionV2 {
    if (sendError) {
        //throw new NetworkClientExceptionV2("sendError", address + " 서버에 데이터 전송 실패: " + data);
        //중간에 다른 예외가 발생했다고 가정
        throw new RuntimeException("ex");
    }
    //전송 성공
    System.out.println(address + " 서버에 데이터 전송: " + data);
}
```

### Run

```
전송할 문자: error2
http://example.com 서버 연결 성공
http://example.com 서버 연결 해제
Exception in thread "main" java.lang.RuntimeException: ex
	at exception.ex2.NetworkClientV2.send(NetworkClientV2.java:24)
	at exception.ex2.NetworkServiceV2_5.sendMessage(NetworkServiceV2_5.java:13)
	at exception.ex2.MainV2.main(MainV2.java:22)
```

- `connect()` succeeds. `send()` throws `RuntimeException` — which `catch (NetworkClientExceptionV2 e)` still cannot match, exactly as in Lecture 95.
- The difference this time: **`"서버 연결 해제"` prints before the stack trace.** `finally` runs first, performing cleanup, and only *then* does the `RuntimeException` continue propagating outward (out of `sendMessage`, out of `main`, terminating the program).
- This is the concrete proof of the guarantee: even an exception the `catch` clause was never written to handle still triggers `finally` on its way out.

> "그러니까 여기서 터져서 얘는 지금 폭탄 나가야 되는데, 일단 finally로 먼저 가요. 서버 연결 해제가 나오죠. 그 다음에 밖으로 나가게 됩니다."

As before, `NetworkClientV2.send(...)` is restored to its working form (`throw new NetworkClientExceptionV2(...)` uncommented, the `RuntimeException` line commented back out) before continuing.

---

## `try`-`finally` Without a `catch`

A `catch` block isn't mandatory — `try`/`finally` alone is valid Java:

```java
try {
    client.connect();
    client.send(data);
} finally {
    client.disconnect();
}
```

- Use this shape when there's no need to handle the exception directly — the caller doesn't want to intercept or recover from it, just make absolutely sure the resource gets released regardless.
- Any exception thrown inside `try` still propagates outward exactly as it would without the `finally` — the only difference is that `finally`'s cleanup code is still guaranteed to run first.

---

## What `try`-`catch`-`finally` Buys You

> 자바 예외 처리는 try ~ catch ~ finally 구조를 사용해서 처리할 수 있다. 덕분에 다음과 같은 이점이 있다.

- **Normal flow and exception flow are separated**, making code easier to read (the Lecture 94 win).
- **Resources used in `try` can always be guaranteed to be released**, no matter what happens inside it — including errors nobody anticipated (this lecture's win).

`NetworkServiceV2_5` is effectively the finished version of this running example's basic exception handling: `try` holds the normal flow, `catch` holds the error flow, `finally` holds the resource cleanup. (Multiple `catch` blocks can be stacked for different exception types when needed — covered in upcoming lectures — but the three-flow shape stays the same.)

---

## Summary

- `finally` is a block attached to `try` (with or without `catch`) that is **guaranteed to execute** whether the `try` completes normally, an exception is caught by `catch`, or an exception propagates out uncaught — making it the right place for mandatory cleanup like releasing external resources.
- `NetworkServiceV2_5` moves `client.disconnect()` from after the `try`/`catch` (Lecture 95's broken fix) into a `finally` block, finally guaranteeing it runs in every case.
- Verified directly: swapping in an uncatchable `RuntimeException` still prints `"서버 연결 해제"` *before* the stack trace and program termination — `finally` runs first, then the uncaught exception continues on its way out.
- `catch` is optional — `try`/`finally` alone is valid when there's no need to handle the exception directly, just guarantee cleanup.
- `try`-`catch`-`finally` together give two concrete benefits: normal/exception flow separation for readability, and a guarantee that resources acquired in `try` are always released. With this, the basic exception-handling refactor of the `NetworkService` example is essentially complete — the next lectures turn to using the exception hierarchy more deliberately (multiple `catch` blocks, more specific exception types).
