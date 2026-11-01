# Lecture 100 — Exception Handling 2 - Practice: Practical Exception Handling Plan - Implementation

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (9/11)

Previous: [Lecture 99 - Practical Exception Handling Plan - Explanation](Lecture%2099%20-%20Practical%20Exception%20Handling%20Plan%20-%20Explanation.md)

## From Theory to Code

Lecture 99 laid out the plan conceptually: convert the running `NetworkClient` example to unchecked exceptions, let unrecoverable exceptions propagate automatically, and build one common exception-handling point near the application boundary. This lecture builds exactly that — a new package, `exception.ex4`, copied forward from `exception.ex3` with every piece converted to runtime exceptions.

---

## Step 1: Convert the Exception Hierarchy to Unchecked

`NetworkClientExceptionV3` (Lecture 97) extended `Exception` — a checked exception. The V4 version just swaps the parent to `RuntimeException`:

```java
package exception.ex4.exception;

public class NetworkClientExceptionV4 extends RuntimeException {
    public NetworkClientExceptionV4(String message) {
        super(message);
    }
}
```

```java
package exception.ex4.exception;

public class ConnectExceptionV4 extends NetworkClientExceptionV4 {

    private final String address;

    public ConnectExceptionV4(String address, String message) {
        super(message);
        this.address = address;
    }

    public String getAddress() {
        return address;
    }
}
```

```java
package exception.ex4.exception;

public class SendExceptionV4 extends NetworkClientExceptionV4 {

    private final String sendData;

    public SendExceptionV4(String sendData, String message) {
        super(message);
        this.sendData = sendData;
    }

    public String getSendData() {
        return sendData;
    }
}
```

- `ConnectExceptionV4` and `SendExceptionV4` are otherwise **structurally identical** to their V3 counterparts — same fields (`address`, `sendData`), same getters. Only the ultimate parent (`RuntimeException` instead of `Exception`) changes.
- Because `NetworkClientExceptionV4`'s parent is now `RuntimeException`, every exception in this hierarchy is automatically unchecked — nothing else in the hierarchy needs to change for that to be true.

> "언체크 예외, 런타임 예외를 만들려면 RuntimeException을 상속받아야 됩니다. 그래서 NetworkClientException이 런타임 익셉션을 상속받게 만들고, 기존 이런 예외들은 또 네트워크 클라이언트 익셉션을 상속받도록 하겠습니다. 그러면 얘들은 전부 다 언체크, 런타임 예외가 됩니다."

---

## Step 2: `NetworkClientV4` — Drop the `throws` Declarations

```java
package exception.ex4;

import exception.ex4.exception.ConnectExceptionV4;
import exception.ex4.exception.SendExceptionV4;

public class NetworkClientV4 {

    private final String address;
    public boolean connectError;
    public boolean sendError;

    public NetworkClientV4(String address) {
        this.address = address;
    }

    public void connect() {
        if (connectError) {
            throw new ConnectExceptionV4(address, address + " 서버 연결 실패");
        }
        //연결 성공
        System.out.println(address + " 서버 연결 성공");
    }

    public void send(String data) {
        if (sendError) {
            throw new SendExceptionV4(data, address + " 서버에 데이터 전송 실패: " + data);
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

- Compare to `NetworkClientV3.connect()`/`send(...)` (Lecture 97), which each declared `throws ConnectExceptionV3` / `throws SendExceptionV3`. Here, both method signatures have **no `throws` at all**.
- This isn't optional cleanup — it's **mandatory**, in the sense that it's now the correct, idiomatic signature: since `ConnectExceptionV4`/`SendExceptionV4` are unchecked, the compiler doesn't require (or even expect) a `throws` declaration for them.

> "throws 이제 다 지울게요. 지워도 되죠. 왜냐? 이거 런타임 익셉션이잖아요. 그렇기 때문에, Unchecked Exception이기 때문에 이제 throws Exception 여기서는 지울 겁니다."

---

## Step 3: `NetworkServiceV4` — Stop Catching What Can't Be Fixed

```java
package exception.ex4;

public class NetworkServiceV4 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV4 client = new NetworkClientV4(address);
        client.initError(data); //추가

        try {
            client.connect();
            client.send(data);
        } finally {
            client.disconnect();
        }

    }
}
```

This is the most important change in the whole lecture. `NetworkServiceV3_1`/`V3_2` (Lectures 97–98) had multiple `catch` blocks for `ConnectExceptionV3` and `SendExceptionV3`. `NetworkServiceV4` has **no `catch` block at all** — just `try { ... } finally { client.disconnect(); }`.

- `NetworkServiceV4` is asked to reason about this: if `ConnectExceptionV4` or `SendExceptionV4` happens, can this class actually **recover**? No — a down remote server or a failed send isn't something `sendMessage(...)` can fix by catching and retrying here.
- Since these are unchecked exceptions, `sendMessage(...)` doesn't need `throws` either — if it doesn't catch something, it's automatically propagated upward with zero boilerplate.
- `finally { client.disconnect(); }` still runs regardless — the resource-cleanup guarantee from Lecture 96 is untouched by this change. Only the exception *handling* responsibility moved; cleanup still happens here because `NetworkServiceV4` is exactly the place that knows it must disconnect the client it opened.

> "NetworkClientV4를 사용하면서 발생하는 CommandException, SendException을 잡아도 사실 해당 오류들을 복구할 수가 없어요. 그래서 예외를 바꾸러 그냥 던질 거예요. ... 이런 방식 덕분에 NetworkServiceV4는 해결할 수 없는 예외보다는 본인 스스로의 코드에 더 집중할 수 있습니다. 따라서 코드가 굉장히 깔끔해지죠."

- The instructor is explicit that this is a judgment call, not an absolute rule: `SendExceptionV4` might in principle be retried (resend the data), but `ConnectExceptionV4` almost certainly can't be (the remote server being down isn't fixed by trying again immediately) — for this example, both are treated as unrecoverable at this layer, and the far more important design point is *where the leftover unrecoverable exceptions go*, covered next.

---

## Step 4: `MainV4` — the Common Exception-Handling Point

If `NetworkServiceV4` simply lets every unrecoverable exception propagate, something still has to stop that exception from crashing the whole program. That something is `MainV4`:

```java
package exception.ex4;

import exception.ex4.exception.SendExceptionV4;

import java.util.Scanner;

public class MainV4 {

    public static void main(String[] args) {
        NetworkServiceV4 networkService = new NetworkServiceV4();

        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.print("전송할 문자: ");
            String input = scanner.nextLine();
            if (input.equals("exit")) {
                break;
            }

            try {
                networkService.sendMessage(input);
            } catch (Exception e) { // 모든 예외를 잡아서 처리
                exceptionHandler(e);
            }
            System.out.println();
        }
        System.out.println("프로그램을 정상 종료합니다.");
    }

    //공통 예외 처리
    private static void exceptionHandler(Exception e) {
        //공통 처리
        System.out.println("사용자 메시지: 죄송합니다. 알 수 없는 문제가 발생했습니다.");
        System.out.println("==개발자용 디버깅 메시지==");
        e.printStackTrace(System.out); //스택 트레이스 출력
        //e.printStackTrace(); // System.err에 스택 트레이스 출력

        //필요하면 예외 별로 별도의 추가 처리 가능
        if (e instanceof SendExceptionV4 sendEx) {
            System.out.println("[전송 오류] 전송 데이터: " + sendEx.getSendData());
        }
    }
}
```

### `catch (Exception e)` in `main` — one block to rule them all

- `sendMessage(input)` is wrapped in `try { ... } catch (Exception e) { exceptionHandler(e); }` — a single, broad `catch`, instead of the per-type `catch` blocks seen in `NetworkServiceV3_2` (Lecture 98).
- Catching `Exception` (the common ancestor of effectively everything thrown in normal application code) means: whatever reaches this point, unhandled by every layer underneath, gets funneled into exactly one place.
- Once `catch` captures the exception object, the exception is — in the instructor's words — already "resolved" as far as crashing the program goes: `main`'s `while (true)` loop continues to the next iteration right after, exactly as it did for caught exceptions in earlier lectures.

> "catch로 잡는 순간 예외는 복구가 됐고, 그럼 그 예외에 대한 정보를 여기서 받는 겁니다. 불은 다 꺼졌어요."

### `exceptionHandler(Exception e)` — the common-handling method itself

This is the concrete implementation of the "common exception handling" concept from Lecture 99:

- **User-facing message**: `"사용자 메시지: 죄송합니다. 알 수 없는 문제가 발생했습니다."` — intentionally generic. In a real web application this would render as an error page; here, printed to the console stands in for that. The user is never shown exception class names, stack traces, or which specific subsystem (database, network, etc.) failed — they have no way to act on that detail anyway.
- **Developer-facing debugging message**: everything printed after `"==개발자용 디버깅 메시지=="`, in particular `e.printStackTrace(System.out)` — conceptually routed somewhere entirely different from the user message (a log file, in a real system), even though both land on the same console here for demonstration purposes.
- **`e instanceof SendExceptionV4 sendEx`**: since the exception is still a real object (not just a flat error string), its *type* can be inspected even inside this single shared handler. Here, if the exception that arrived happens to be a `SendExceptionV4`, an extra line is appended logging the `sendData` that failed to send — a type-specific enhancement layered on top of the otherwise-uniform common handling, using the pattern-matching `instanceof` form (binds `sendEx` directly, no separate cast needed).

> "예외도 객체이기 때문에 공통 처리 메서드인 이 메서드에 예외 객체를 전달할 수 있습니다. ... 필요하면 예외별로 별도의 추가 처리 가능. send exception이면 우리가 이렇게 좀 남기고 싶은 거예요."

### Why `exceptionHandler` takes `Exception` as its parameter type

- `exceptionHandler(Exception e)` accepts the broad `Exception` type, not a specific one — because this method is meant to handle literally anything that reaches it, by design. Since the exception is passed as a reference (an object), the method can still narrow its behavior per-type internally via `instanceof`, without needing a narrower parameter type.

---

## `System.out` vs `System.err`: Why `e.printStackTrace(System.out)` Is Used Here

The code explicitly calls `e.printStackTrace(System.out)` rather than the simpler, more common `e.printStackTrace()`. Both print the exception message and full stack trace, but they go to different output streams:

| Call | Destination | Typical IDE color |
|---|---|---|
| `e.printStackTrace()` | `System.err` (standard error) | Red |
| `e.printStackTrace(System.out)` | `System.out` (standard output) | Default |

- `System.out` and `System.err` are two genuinely **different output streams**, even though both normally render in the same console window.
- Because they're different streams, **interleaving output between them doesn't guarantee ordering** — mixing `System.out.println(...)` calls with a bare `e.printStackTrace()` in the same block can make lines print out of the order they were written in the source, which the instructor demonstrated live and found confusing for teaching purposes.
- That's why the example code deliberately routes everything through `System.out` (`printStackTrace(System.out)`) — purely to keep output lines appearing in a predictable order while learning, not because it's the generally recommended real-world choice.

> "일반적으로는 이걸로(`e.printStackTrace()`, 즉 System.err) 그냥 많이 사용합니다. 이건 문제가 있는 거니까요." (In typical practice, routing error output to `System.err` by default makes sense precisely because it *is* a problem that should stand out — e.g. shown in red in an IDE.)

### Why neither is actually used in real production systems

- In real applications, directly calling `e.printStackTrace()`/`e.printStackTrace(System.out)` to dump straight to the console is **uncommon** — because production servers aren't watched live on a console; logs have to be checked after the fact via files.
- Real systems instead use a **logging library** (the instructor names `SLF4J` and `Logback` specifically, to be covered when Spring is introduced later) that writes to both the console and dedicated log files, with much more control over formatting, levels, and rotation than raw console output provides.
- For now, at the learning stage, `e.printStackTrace()` remains a perfectly fine tool — the instructor explicitly encourages using it liberally while studying, specifically as a way to quickly see *what exception just happened* when something isn't behaving as expected.

> "지금은 로그 라이브러리라는 게 있구나! 정도 알아보시면 됩니다. 학습 단계에서는 이 정도, e.printStackTrace를 적극적으로 사용해도 괜찜습니다."

---

## Run

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: error1
http://example.com 서버 연결 해제
사용자 메시지: 죄송합니다. 알 수 없는 문제가 발생했습니다.
==개발자용 디버깅 메시지==
exception.ex4.exception.ConnectExceptionV4: http://example.com 서버 연결 실패
    at exception.ex4.NetworkClientV4.connect(NetworkClientV4.java:18)
    at exception.ex4.NetworkServiceV4.sendMessage(NetworkServiceV4.java:12)
    at exception.ex4.MainV4.main(MainV4.java:22)

전송할 문자: error2
http://example.com 서버 연결 성공
http://example.com 서버 연결 해제
사용자 메시지: 죄송합니다. 알 수 없는 문제가 발생했습니다.
==개발자용 디버깅 메시지==
exception.ex4.exception.SendExceptionV4: http://example.com 서버에 데이터 전송 실패: error2
    at exception.ex4.NetworkClientV4.send(NetworkClientV4.java:26)
    at exception.ex4.NetworkServiceV4.sendMessage(NetworkServiceV4.java:13)
    at exception.ex4.MainV4.main(MainV4.java:22)
[전송 오류] 전송 데이터: error2

전송할 문자: exit
프로그램을 정상 종료합니다.
```

Tracing `error1`:

- `connect()` throws `ConnectExceptionV4` — uncaught in `NetworkClientV4`, uncaught in `NetworkServiceV4` (no `catch` there at all), but `finally { client.disconnect(); }` still runs on the way out, printing `"서버 연결 해제"` *before* the exception keeps propagating.
- It finally reaches `MainV4.main`'s `try/catch (Exception e)`, which calls `exceptionHandler(e)` — printing the generic user message, then the full stack trace via `e.printStackTrace(System.out)`.
- Because `ConnectExceptionV4` isn't a `SendExceptionV4`, the `instanceof` check inside `exceptionHandler` doesn't add the extra `[전송 오류]` line.
- After `exceptionHandler` returns, control returns to the `while (true)` loop in `main` exactly as if nothing had gone wrong — the exception has been fully "recovered" from the program's point of view, and the next `"전송할 문자: "` prompt appears.

Tracing `error2`:

- Same shape, but `send(...)` throws `SendExceptionV4` this time (after `connect()` already succeeded, so `"서버 연결 성공"` prints first).
- `exceptionHandler` runs the same generic common-handling logic — but this time `e instanceof SendExceptionV4 sendEx` is true, so the extra `[전송 오류] 전송 데이터: error2` line appears, pulled directly from `sendEx.getSendData()`.

---

## Why This Pattern Is the Real-World Default

> "실무에서는 이런 식으로 공통 예외를 처리하는 건 진짜 많이 해요. 왜냐하면 어차피 해결할 수 없는 예외가 너무 많아요."

- `Exception` is caught once, at (effectively) the boundary closest to the user — not scattered across every intermediate layer that happens to call something risky.
- Every layer in between (`NetworkClientV4`, `NetworkServiceV4`) is free to focus purely on its own logic, without `throws` clutter accumulating as more libraries/clients get added — exactly the "checked exception hell" problem from Lecture 99 is sidestepped entirely.
- The instructor adds a forward-looking note: most modern libraries — including recent parts of the JDK itself, and essentially all popular open-source libraries today — have moved toward unchecked exceptions for this exact reason. Older Java-provided APIs tended to lean on checked exceptions; that style has fallen out of favor industry-wide.

> "최근 자바가 제공하는 라이브러리뿐만 아니라 최근 오픈소스들은 거의 대부분 다 언체크, 런타임 예외만 사용합니다. ... 필요한 거는 여러분 잡으시면 되고요. 그리고 해결할 수 없는 거는 공통 예외 처리 부분을 만들어서 처리하시는 거를 권장드리겠습니다."

---

## Summary

- `NetworkClientExceptionV4` (and its children `ConnectExceptionV4`, `SendExceptionV4`) are structurally identical to their V3 counterparts, except the top-level parent changes from `Exception` to `RuntimeException` — making the entire hierarchy unchecked in one edit.
- `NetworkClientV4.connect()`/`send(...)` drop their `throws` declarations entirely — unchecked exceptions don't require (or allow meaningfully) declaring them.
- `NetworkServiceV4.sendMessage(...)` removes every `catch` block it previously had for `ConnectExceptionV3`/`SendExceptionV3` — since it can't actually recover from either failure, it lets both propagate automatically, keeping only the `finally { client.disconnect(); }` cleanup it's directly responsible for.
- `MainV4.main` adds a single `try { networkService.sendMessage(input); } catch (Exception e) { exceptionHandler(e); }` around the one risky call — the **common exception-handling point** for everything no inner layer could resolve.
- `exceptionHandler(Exception e)` implements the two-part common-handling pattern from Lecture 99: a generic, safe message for the user, and a full stack trace (`e.printStackTrace(System.out)`) for developers — plus an optional `instanceof` check (here, `SendExceptionV4`) to layer on type-specific extra logging without abandoning the shared handling path.
- `System.out` and `System.err` are different, independently-flushed output streams; mixing `e.printStackTrace()` (which uses `System.err`) with `System.out.println(...)` calls can print lines out of source order — real systems avoid this entire issue by routing exception logging through a dedicated logging library (e.g. SLF4J/Logback) instead of calling `printStackTrace` directly.
- This pattern — most libraries now throwing unchecked exceptions, with one common handling point near the application boundary — is described as the dominant real-world approach, and is exactly what modern Java libraries and open-source ecosystems have converged on.
