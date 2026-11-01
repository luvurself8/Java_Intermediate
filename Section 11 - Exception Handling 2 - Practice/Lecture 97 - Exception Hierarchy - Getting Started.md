# Lecture 97 — Exception Handling 2 - Practice: Exception Hierarchy - Getting Started

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (6/11)

Previous: [Lecture 96 - Introducing Exception Handling - finally](Lecture%2096%20-%20Introducing%20Exception%20Handling%20-%20finally.md)

## Beyond a Single Exception With an Error Code

Up through `NetworkServiceV2_5`, every failure threw the same `NetworkClientExceptionV2`, distinguished only by its internal `errorCode` string (`"connectError"`, `"sendError"`). This lecture replaces that with **real class hierarchy**: a distinct exception type per failure kind.

```
                Exception
                    |
       NetworkClientExceptionV3
          /                  \
ConnectExceptionV3        SendExceptionV3
   (String address)      (String sendData)
```

- `NetworkClientExceptionV3`: the shared parent — any exception thrown from `NetworkClient` is designed to be a child of this.
- `ConnectExceptionV3`: thrown when a connection fails; carries the `address` that failed to connect.
- `SendExceptionV3`: thrown when a send fails; carries the `sendData` that failed to send.

### Why hierarchy instead of an error-code field

> "자바에서 예외는 객체이다. 따라서 부모 예외를 잡거나 던지면, 자식 예외도 함께 잡거나 던질 수 있다."

- Catching the parent `NetworkClientExceptionV3` still catches both `ConnectExceptionV3` and `SendExceptionV3` — a single `catch` can handle "anything from `NetworkClient`" when that's all that's needed.
- Catching a **specific** subtype (`ConnectExceptionV3`, or `SendExceptionV3`) handles just that one kind of failure when more precise handling is wanted — different log message, different recovery logic, whatever fits.
- Compare this to the old error-code approach: reading `catch (NetworkClientExceptionV2 e)` told you nothing about *what kind* of failure occurred — you had to branch on `e.getErrorCode()` inside the `catch` to find out. With a type hierarchy, **the exception's own class already tells you what happened** — no further branching required.

New package: `exception.ex3`, with exception classes grouped in their own sub-package `exception.ex3.exception` for clarity.

---

## Defining the Exception Hierarchy

### `NetworkClientExceptionV3` — the shared parent

```java
package exception.ex3.exception;

public class NetworkClientExceptionV3 extends Exception {
    public NetworkClientExceptionV3(String message) {
        super(message);
    }
}
```

- Plain checked exception (`extends Exception`), holding only a message — every exception `NetworkClient` can throw is designed to inherit from this one.

### `ConnectExceptionV3` — connection failure, carries the address

```java
package exception.ex3.exception;

public class ConnectExceptionV3 extends NetworkClientExceptionV3 {

    private final String address;

    public ConnectExceptionV3(String address, String message) {
        super(message);
        this.address = address;
    }

    public String getAddress() {
        return address;
    }
}
```

- Extends `NetworkClientExceptionV3` — so it's a `NetworkClientExceptionV3` too, by inheritance.
- Stores the `address` that the failed connection attempt was targeting — information specific to *this* kind of failure, that a generic error-code-based exception had no natural place to put.

### `SendExceptionV3` — send failure, carries the attempted data

```java
package exception.ex3.exception;

public class SendExceptionV3 extends NetworkClientExceptionV3 {

    private final String sendData;

    public SendExceptionV3(String sendData, String message) {
        super(message);
        this.sendData = sendData;
    }

    public String getSendData() {
        return sendData;
    }
}
```

- Same shape as `ConnectExceptionV3`, but stores `sendData` — the data that failed to transmit — instead of an address.

---

## `NetworkClientV3`: Throw the Specific Exception Type

```java
package exception.ex3;

import exception.ex3.exception.ConnectExceptionV3;
import exception.ex3.exception.SendExceptionV3;

public class NetworkClientV3 {

    private final String address;
    public boolean connectError;
    public boolean sendError;

    public NetworkClientV3(String address) {
        this.address = address;
    }

    public void connect() throws ConnectExceptionV3 {
        if (connectError) {
            throw new ConnectExceptionV3(address, address + " 서버 연결 실패");
        }

        System.out.println(address + " 서버 연결 성공");
    }

    public void send(String data) throws SendExceptionV3 {
        if (sendError) {
            throw new SendExceptionV3(data, address + " 서버에 데이터 전송 실패: " + data);
        }

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

- Each method now declares `throws` with its **exact** exception type — `connect()` declares `throws ConnectExceptionV3`, `send(...)` declares `throws SendExceptionV3` — not the shared parent, not a generic one-size-fits-all type.
- The constructor calls pass the type-specific data: `ConnectExceptionV3(address, message)`, `SendExceptionV3(data, message)`.
- No `errorCode` field anywhere — the exception's **class itself** now communicates what went wrong. Previously, `NetworkClientExceptionV2` always fired, and you had to inspect `e.getErrorCode()` to tell a connect failure from a send failure. Now a connect failure and a send failure are simply different exception types from the start.

---

## `NetworkServiceV3_1`: Catching Each Exception Type Separately

```java
package exception.ex3;

import exception.ex3.exception.ConnectExceptionV3;
import exception.ex3.exception.SendExceptionV3;

public class NetworkServiceV3_1 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV3 client = new NetworkClientV3(address);
        client.initError(data); //추가

        try {
            client.connect();
            client.send(data);
        } catch (ConnectExceptionV3 e) {
            System.out.println("[연결 오류] 주소: " + e.getAddress() + ", 메시지: " + e.getMessage());
        } catch (SendExceptionV3 e) {
            System.out.println("[전송 오류] 전송 데이터: " + e.getSendData() + ", 메시지: " + e.getMessage());
        } finally {
            client.disconnect();
        }

    }
}
```

- Same `try`/`catch`/`finally` shape established in Lectures 94–96 — normal flow in `try`, cleanup in `finally` — but now **two separate `catch` blocks**, one per exception type.
- `catch (ConnectExceptionV3 e)`: logs `"[연결 오류] 주소: ..."`, using `e.getAddress()` — a method that only exists on `ConnectExceptionV3`, not available on the generic exception from the old version.
- `catch (SendExceptionV3 e)`: logs `"[전송 오류] 전송 데이터: ..."`, using `e.getSendData()` — again, specific to this exception type.
- **This is the concrete payoff of the hierarchy**: each `catch` block can use methods unique to its own exception subtype and run genuinely different logic per failure kind — not just a different printed string, but access to structured data (`address` vs. `sendData`) that's only meaningful for that specific kind of failure.
- `IntelliJ note`: adding a second `catch` clause can be done via the "Add catch clause" quick-fix; the order of multiple `catch` blocks here doesn't matter since `ConnectExceptionV3` and `SendExceptionV3` are siblings (neither is a supertype of the other).

## `MainV3`

```java
package exception.ex3;

import exception.ex3.exception.NetworkClientExceptionV3;

import java.util.Scanner;

public class MainV3 {

    public static void main(String[] args) {
        NetworkServiceV3_1 networkService = new NetworkServiceV3_1();

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

- `sendMessage(...)` itself no longer declares `throws` — both exception types it could propagate are now fully caught inside it, same as the final version in Lecture 96.

---

## Run

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
[전송 오류] 전송 데이터: error2, 메시지: http://example.com 서버에 데이터 전송 실패: error2
http://example.com 서버 연결 해제

전송할 문자: exit
프로그램을 정상 종료합니다.
```

- `error1`: `connect()` throws `ConnectExceptionV3` → the first `catch` matches, printing the address alongside the message.
- `error2`: `connect()` succeeds, `send()` throws `SendExceptionV3` → the second `catch` matches, printing the attempted send data alongside the message.
- Either way, `finally` still runs afterward, printing `"서버 연결 해제"`.
- The two error messages are structurally different (`[연결 오류] 주소: ...` vs. `[전송 오류] 전송 데이터: ...`) because two different `catch` blocks, matched to two different exception classes, ran two different pieces of logic — not because of an `if` on an error-code string.

---

## Summary

- Replacing a single exception class with an internal `errorCode` field, Lecture 97 instead builds a real exception **hierarchy**: `NetworkClientExceptionV3` as the shared parent, with `ConnectExceptionV3` (carries `address`) and `SendExceptionV3` (carries `sendData`) as distinct subtypes for each specific failure.
- Because exceptions are objects, catching the parent type still catches every subtype — but catching a specific subtype (as done here) allows handling each failure kind with its own logic and its own exception-specific data.
- `NetworkClientV3.connect()`/`send(...)` each declare `throws` with their own precise exception type, rather than one shared type — the exception's class itself now communicates what went wrong, with no error-code lookup needed.
- `NetworkServiceV3_1` catches `ConnectExceptionV3` and `SendExceptionV3` in two separate `catch` blocks, each using methods unique to that exception type (`getAddress()` / `getSendData()`) to produce genuinely different error messages — the `finally` block for `disconnect()` carries over unchanged from Lecture 96.
- This hierarchy is set up now so the next lecture can build on it — catching at different levels of specificity (a specific subtype, the shared parent, or a catch-all fallback) in the same `try`/`catch` chain.
