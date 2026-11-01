# Lecture 92 — Exception Handling 2 - Practice: Introducing Exception Handling - Getting Started

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (1/11)

## Bringing Real Exceptions Into the NetworkService Example

Back to the running example from Section 10 (Lectures 85–87): `NetworkServiceV1_3` used return-value-based error handling, which tangled normal flow and error-handling flow together — the problem the section's theory lectures explained Java's exception mechanism exists to solve.

This lecture starts actually introducing Java exceptions into that example, step by step across the section. **First step: convert the return-code errors into a real checked exception.**

New package: `exception.ex2` (version "V2," even though there's no "V1" exception class — matching the `ex1`/`V1` naming already used for this running example).

---

## `NetworkClientExceptionV2`: a Custom Checked Exception with an Error Code

```java
package exception.ex2;

public class NetworkClientExceptionV2 extends Exception {

    private String errorCode;

    public NetworkClientExceptionV2(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() {
        return errorCode;
    }
}
```

```
Object → Throwable (String detailMessage) → Exception → NetworkClientExceptionV2 (String errorCode, getErrorCode())
```

- Extends `Exception` → this is a **checked** exception, just like `MyCheckedException` from Lecture 90.
- **Error code**: previously (`NetworkClientV1`), the error code (`"connectError"`, `"sendError"`) was returned as a plain `String` from `connect()`/`send()`. Now it's stored **inside the exception object itself**, as an `errorCode` field with a `getErrorCode()` getter.
- **Error message**: delegated entirely to the inherited `Throwable` functionality via `super(message)` — no need to store it manually, same pattern as `MyCheckedException`.
- Why this works at all: **exceptions are just objects**, so they can carry whatever fields and methods are useful — there's nothing special stopping an exception from holding an error code alongside its message.

---

## `NetworkClientV2`: Throw Instead of Return

```java
package exception.ex2;

public class NetworkClientV2 {

    private final String address;
    public boolean connectError;
    public boolean sendError;

    public NetworkClientV2(String address) {
        this.address = address;
    }

    public void connect() throws NetworkClientExceptionV2 {
        if (connectError) {
            throw new NetworkClientExceptionV2("connectError", address + " 서버 연결 실패");
        }
        //연결 성공
        System.out.println(address + " 서버 연결 성공");
    }

    public void send(String data) throws NetworkClientExceptionV2 {
        if (sendError) {
            throw new NetworkClientExceptionV2("sendError", address + " 서버에 데이터 전송 실패: " + data);
        }
        //전송 성공
        System.out.println(address + " 서버에 데이터 전송: " + data);
    }

    public void disconnect() {
        System.out.println(address + " 서버 연결 해제");
    }

    public void initError(String data) {
        if (data.contains("error1")) {
            connectError = true;
        }
        if (data.contains("error2")) {
            sendError = true;
        }
    }
}
```

Compared to `NetworkClientV1`, the core change is in `connect()` and `send()`:

- Instead of `return "connectError"` / `return "sendError"`, each now does `throw new NetworkClientExceptionV2(errorCode, message)` — constructing the exception with both the error code and a descriptive message, then detonating it.
- Both methods now **return `void`** instead of `String`.
  - Why: a return value is no longer needed to signal success/failure. If `connect()` returns normally (no exception thrown), that *is* the success signal. If something goes wrong, an exception flies out instead — there's nothing left for a return value to communicate.
  - "성공" is no longer a string that needs to be returned and checked — it's just "the method call completed without an exception."
- Both methods now declare `throws NetworkClientExceptionV2` — required, since `NetworkClientExceptionV2` is a checked exception and neither method catches it internally.
- `connectError`/`sendError`/`initError(...)` are unchanged from `NetworkClientV1` — still the same simulation switches driven by `"error1"`/`"error2"` substrings in user input.

---

## `NetworkServiceV2_1`: Just Propagate for Now

```java
package exception.ex2;

public class NetworkServiceV2_1 {

    public void sendMessage(String data) throws NetworkClientExceptionV2 {
        String address = "http://example.com";

        NetworkClientV2 client = new NetworkClientV2(address);
        client.initError(data);

        client.connect();
        client.send(data);
        client.disconnect();
    }
}
```

- Notice how close this is to the very first version, `NetworkServiceV0` — `connect()`, `send(data)`, `disconnect()` called back-to-back, with **no** error-checking branches at all.
- This version makes no attempt to catch anything — it simply declares `throws NetworkClientExceptionV2` and lets any exception from `client.connect()`/`client.send(data)` propagate straight through.
- This is a deliberate first step, not the final state: the next lecture will add actual exception *recovery* here. For now, the point is just to see what the normal-flow code looks like once error handling via return codes is gone entirely.

## `MainV2`

```java
package exception.ex2;

import java.util.Scanner;

public class MainV2 {

    public static void main(String[] args) throws NetworkClientExceptionV2 {
        NetworkServiceV2_1 networkService = new NetworkServiceV2_1();

        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.print("전송할 문자: ");
            String input = scanner.nextLine();
            if (input.equals("exit")) {
                break;
            }
            networkService.sendMessage(input);
            System.out.println();
        }
        System.out.println("프로그램을 정상 종료합니다.");
    }
}
```

- `main()` itself also doesn't catch anything — it just adds its own `throws NetworkClientExceptionV2` declaration, since `networkService.sendMessage(input)` can throw it and `main()` makes no attempt to handle it either.
- This is the simplest possible way to satisfy the compiler for a checked exception: keep declaring `throws` all the way up the call chain instead of catching it anywhere.

---

## Run

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: exit
프로그램을 정상 종료합니다.
```

```
전송할 문자: error1
Exception in thread "main" exception.ex2.NetworkClientExceptionV2: http://example.com 서버 연결 실패
	at exception.ex2.NetworkClientV2.connect(NetworkClientV2.java:15)
	at exception.ex2.NetworkServiceV2_1.sendMessage(NetworkServiceV2_1.java:11)
	at exception.ex2.MainV2.main(MainV2.java:22)
```

```
전송할 문자: error2
http://example.com 서버 연결 성공
Exception in thread "main" exception.ex2.NetworkClientExceptionV2: http://example.com 서버에 데이터 전송 실패: error2
	at exception.ex2.NetworkClientV2.send(NetworkClientV2.java:23)
	at exception.ex2.NetworkServiceV2_1.sendMessage(NetworkServiceV2_1.java:12)
	at exception.ex2.MainV2.main(MainV2.java:22)
```

- `hello`: everything succeeds, exactly as before.
- `error1`: `connect()` throws `NetworkClientExceptionV2`. Nothing catches it anywhere in the chain (`NetworkServiceV2_1.sendMessage` → `MainV2.main`), so it propagates all the way out of `main()`, printing a stack trace and terminating the program.
- `error2`: connection succeeds, then `send()` throws. Same unhandled propagation and termination.
- The exception message itself is printed right alongside the stack trace (`"...서버에 데이터 전송 실패: error2"`) — reading it directly tells you *why* it failed, on top of *where*. The instructor emphasizes how valuable a well-written exception message is in practice: a vague message means a developer has to dig for the cause later; a specific one (as here) tells you immediately what happened.

> "오류 메시지를 잘 정의하는 게 진짜 중요해요. 안 그러면 여러분, 누가 고생할까요? 내가 고생합니다."

---

## Remaining Problems

Exception handling has been introduced, but two problems from the original `NetworkClient` usage rules are still unresolved:

1. **No exception recovery.** The program just crashes whenever an error occurs — there's no catching happening anywhere, so any failure takes down the whole process instead of being handled gracefully.
2. **`disconnect()` isn't guaranteed to run.** Just like with the return-value approach back in Lecture 87, if `send()` throws, `disconnect()` never gets called — the external connection resource still leaks.

Both are picked up starting next lecture.

---

## Summary

- `NetworkClientExceptionV2 extends Exception` is a custom checked exception that bundles an `errorCode` field alongside the inherited `message` — exceptions are objects, so they can carry whatever data is useful, same as any other class.
- `NetworkClientV2.connect()`/`send()` now `throw` this exception on failure instead of returning an error-code string, and both return `void` — a normal return now *is* the success signal, and an exception *is* the failure signal; there's nothing left for a return value to communicate.
- `NetworkServiceV2_1` and `MainV2` don't catch anything yet — they just declare `throws NetworkClientExceptionV2` and let it propagate, to first see what the de-tangled normal-flow code looks like on its own.
- Running `error1`/`error2` crashes the program with a stack trace, same as any unhandled checked exception — the message embedded in the exception is what makes that stack trace genuinely useful for debugging, rather than just a noisy crash.
- Two problems remain: exceptions aren't actually recovered from (the program just dies), and `disconnect()` still isn't guaranteed to run on failure — both addressed in the upcoming lectures.
