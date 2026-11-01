# Lecture 101 — Exception Handling 2 - Practice: try-with-resources

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (10/11)

Previous: [Lecture 100 - Practical Exception Handling Plan - Implementation](Lecture%20100%20-%20Practical%20Exception%20Handling%20Plan%20-%20Implementation.md)

## The Recurring Pattern This Solves

Every version of `NetworkClient` so far (V2 through V4) followed the same shape:

```java
try {
    //정상 흐름
} catch {
    //예외 흐름
} finally {
    //반드시 호출해야 하는 마무리 흐름
}
```

- Using an external resource (a network connection, in this running example) **always** requires releasing it afterward — a connection that's never explicitly closed stays open indefinitely, which is a real problem.
- That means `finally` is mandatory wherever a resource is opened — and the pattern of "acquire in `try`, release in `finally`" repeats itself in every single method that touches a resource.
- Java 7 introduced **try-with-resources** specifically to eliminate this repetition: a convenience feature that lets a resource be declared *together with* the `try` statement, so the resource is automatically released when the `try` block ends — no explicit `finally` needed at all.

> "try에서 외부 자원을 사용하고, try가 끝나면 외부 자원을 반납하는 패턴이 반복되면서 자바에서는 Try with resources라는 편의 기능을 자바 7에서 도입했다. 이름 그대로 try에서 자원을 함께 사용한다는 뜻이다."

---

## Step 1: Implement `AutoCloseable`

To use try-with-resources, the resource class must implement the `AutoCloseable` interface:

```java
package java.lang;

public interface AutoCloseable {
    void close() throws Exception;
}
```

- A single abstract method, `close()`, declared to throw `Exception` (the broadest possible checked exception — individual implementations are free to narrow this, throw nothing, or not throw at all).
- Implementing this interface is what makes a class eligible to appear inside a try-with-resources' parentheses: the JVM guarantees `close()` gets called automatically the moment the `try` block ends.

### `NetworkClientV5` — copied from `NetworkClientV4`, with `close()` added

```java
package exception.ex4;

import exception.ex4.exception.ConnectExceptionV4;
import exception.ex4.exception.SendExceptionV4;

public class NetworkClientV5 implements AutoCloseable {

    private final String address;
    public boolean connectError;
    public boolean sendError;

    public NetworkClientV5(String address) {
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

    @Override
    public void close() {
        System.out.println("NetworkClientV5.close");
        disconnect();
    }
}
```

- `implements AutoCloseable` is the only structural addition at the class-declaration level.
- `close()` is the `AutoCloseable` method's implementation: it logs `"NetworkClientV5.close"`, then calls `disconnect()` — the exact cleanup logic that previously lived in a `finally` block.
- **Crucially, `close()` here declares no `throws Exception`** — because this particular implementation never actually throws anything. The interface method's `throws Exception` is only the *maximum* allowed; an override is always free to narrow it away entirely when it genuinely can't fail.

> "참고로 여기서는 예외를 안 던지기 때문에 이 인터페이스 메서드는 throws Exception은 제거해 주셔야 됩니다."

---

## Step 2: Use the Resource Inside `try (...)`

```java
package exception.ex4;

public class NetworkServiceV5 {

    public void sendMessage(String data) {
        String address = "http://example.com";

        try (NetworkClientV5 client = new NetworkClientV5(address)) {
            client.initError(data); //추가
            client.connect();
            client.send(data);
        } catch (Exception e) {
            System.out.println("[예외 확인]: " + e.getMessage());
            throw e;
        }
    }
}
```

Compare this to `NetworkServiceV4` (Lecture 100), which had:

```java
NetworkClientV4 client = new NetworkClientV4(address);
client.initError(data);

try {
    client.connect();
    client.send(data);
} finally {
    client.disconnect();
}
```

### What changed, and why

- The resource (`client`) is now declared **inside the parentheses right after `try`**: `try (NetworkClientV5 client = new NetworkClientV5(address))`. This is the try-with-resources syntax itself.
- There's **no `finally` block at all anymore** — because `client` implements `AutoCloseable`, the JVM automatically calls `client.close()` the moment the `try` block finishes, for any reason (normal completion, an exception being thrown, doesn't matter).
- `client.initError(data)` moved **inside** the `try` block. This is a direct consequence of where `client` is declared: a variable declared in the `try (...)` header is scoped to the `try` block (and its `catch`/`finally`, if any) — it simply doesn't exist outside it. Anything that needs `client` must happen inside that block now.

> "자, 여기서 이제 어떻게 되느냐. 여기서 이제 객체 만들어서 실행하죠, 쭉. ... 이 변수는 여기 안에서만 쓸 수 있습니다. 그래서 여기 client.initError도 여기 안에, try 안에 들어와야 돼요."

### `catch (Exception e)` here is doing something different from Lecture 100

- Unlike `MainV4.exceptionHandler(...)` (the *common* exception-handling point from Lecture 100), this `catch` isn't the final destination for the exception — it's a brief stop.
- It prints a short diagnostic line (`"[예외 확인]: " + e.getMessage()`) purely to demonstrate **timing** (explained below), then immediately re-throws the same exception object with `throw e`.
- The exception continues propagating upward from here exactly as it would have without this `catch` at all — this block exists in the lecture specifically to let the instructor show *when*, relative to `catch`, the resource actually gets closed.

> "여기서 catch block은 단순히 발생한 예외를 잡아서 예외 메시지를 출력하고, 잡은 예외를 throw를 통해서 다시 그냥 밖으로 던지는 일을 합니다."

### Catch block isn't even required

- The instructor notes explicitly that `try (...) { ... }` works fine with **no `catch` at all** — `close()` still gets called correctly; the `catch` in this example exists purely for the demonstration above, not because try-with-resources requires one.

---

## `MainV4` Swapped to `NetworkServiceV5`

```java
public static void main(String[] args) throws NetworkClientExceptionV2 {
    //NetworkServiceV4 networkService = new NetworkServiceV4();
    NetworkServiceV5 networkService = new NetworkServiceV5();
}
```

- `MainV4` is updated (by commenting out the `NetworkServiceV4` line and instantiating `NetworkServiceV5` instead) to run this lecture's version — the rest of `main` (the `while (true)` loop, `catch (Exception e) { exceptionHandler(e); }`) is unchanged from Lecture 100.

---

## Run and Trace: Proving *When* `close()` Fires

### Normal case (`hello`)

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
NetworkClientV5.close
http://example.com 서버 연결 해제
```

- `connect()` and `send(...)` both succeed, printing their usual success messages.
- Right after `send(...)` returns, the `try` block has nothing left to do — it ends, and `close()` fires immediately: `"NetworkClientV5.close"` then `"서버 연결 해제"` (from `disconnect()` inside `close()`).
- Compare to `NetworkServiceV4`'s equivalent `hello` run (Lecture 100), where `"서버 연결 해제"` only appeared *after* the full `try` **and then** `finally` both ran — here, `close()` plays the same role but fires right at the point the `try` block's code finishes, with no separate `finally` phase afterward.

### `error1` — the key timing demonstration

```
전송할 문자: error1
NetworkClientV5.close
http://example.com 서버 연결 해제
[예외 확인]: https://example.com 서버 연결 실패
사용자 메시지: 죄송합니다. 알 수 없는 문제가 발생했습니다.
==개발자용 디버깅 메시지==
exception.ex4.exception.ConnectExceptionV4: https://example.com 서버 연결 실패
    at exception.ex4.NetworkClientV5.connect(NetworkClientV5.java:18)
    at exception.ex4.NetworkServiceV5.sendMessage(NetworkServiceV5.java:10)
    at exception.ex4.MainV4.main(MainV4.java:23)
```

Walking through the order of operations here is the entire point of this lecture:

1. `connect()` throws `ConnectExceptionV4` — the `try` block is abandoned at that instant.
2. **Before** the `catch (Exception e)` block runs at all, `close()` fires: `"NetworkClientV5.close"`, then `"서버 연결 해제"`.
3. **Only after** `close()` has fully finished does control reach `catch (Exception e)`, which prints `"[예외 확인]: ..."` and re-throws.
4. The re-thrown exception propagates up to `MainV4`'s own `catch (Exception e) { exceptionHandler(e); }`, producing the generic user message plus the stack trace — same common-handling behavior as Lecture 100.

> "예외 확인이 먼저 출력이 됐어요, 아니면 서버 연결 해제가 먼저 출력돼요? try가 빠져나가는 순간, try에서 나가는 순간 AutoCloseable은 바로 호출됩니다. 캐치로 가든 파이널리로 가든 어디로 가든 이 try를 빠져나가는 순간 바로 AutoCloseable이 호출이 됩니다."

### `error2` follows the identical shape

```
전송할 문자: error2
http://example.com 서버 연결 성공
NetworkClientV5.close
http://example.com 서버 연결 해제
[예외 확인]: https://example.com 서버에 데이터 전송 실패: error2
사용자 메시지: 죄송합니다. 알 수 없는 문제가 발생했습니다.
==개발자용 디버깅 메시지==
exception.ex4.exception.SendExceptionV4: https://example.com 서버에 데이터 전송 실패: error2
    at exception.ex4.NetworkClientV5.send(NetworkClientV5.java:26)
    at exception.ex4.NetworkServiceV5.sendMessage(NetworkServiceV5.java:11)
    at exception.ex4.MainV4.main(MainV4.java:23)
[전송 오류] 전송 데이터: error2
```

- `connect()` succeeds this time, so `"서버 연결 성공"` prints first — then `send(...)` throws `SendExceptionV4`, and the same ordering repeats: `close()` fires immediately, *then* `catch` runs.
- The final `[전송 오류] 전송 데이터: error2` line comes from `MainV4.exceptionHandler`'s `instanceof SendExceptionV4` check (Lecture 100) — unchanged here, since all of that logic lives in `MainV4`, not in `NetworkServiceV5`.

```
전송할 문자: exit
프로그램을 정상 종료합니다.
```

---

## Why try-with-resources Releases Resources *Faster* Than `finally`

This is the core payoff the whole trace above is building toward:

- With a plain `try/catch/finally`, the order is always: `try` runs (or fails) → `catch` runs (if matched) → **then, finally, `finally` runs** — meaning the resource stays open throughout the entire `catch` block, even though the `catch` block itself no longer needs that resource for anything.
- With try-with-resources, `close()` fires **the instant the `try` block is exited** — before `catch` even begins executing. The resource is released as early as structurally possible, rather than waiting for `catch` to finish its own (possibly unrelated) work first.

> "결과적으로 이게 왜 더 좋냐면, 자원을 더 빨리 해제한다는 거예요. finally는 이거 하고 있는 동안에도 필요 없는데 자원을 아직 해제 안 하고 계속 있죠. 그리고 그 다음에 finally 호출되죠."

---

## Try-with-resources Advantages, Summarized

| Advantage | What it means here |
|---|---|
| **Resource-leak prevention** | Guarantees every resource gets properly closed — prevents the classic mistake of forgetting a `finally` block, or forgetting the actual release call *inside* a `finally` block. |
| **More concise, more readable code** | No explicit `close()`/`disconnect()` call to remember at the call site — the compiler wires it up. |
| **Narrower scope** | A resource declared in `try (...)` is scoped strictly to that `try` block — compare `client` in `NetworkServiceV4` (usable anywhere after its declaration in the method) vs. `client` in `NetworkServiceV5` (only exists inside the `try`). A narrower scope is easier to reason about and debug — once you're past the `try` block, that resource is simply not a concern anymore. |
| **Faster resource release** | `close()` runs the moment the `try` block exits — before any `catch` logic runs — rather than waiting for `catch` to finish first, as plain `finally` does. |

> "스코프가 좁아지는 거예요, 자원이 좁아지는 거예요, 여기 안에서만. ... 아무래도 디버깅 할 때 훨씬 좋죠. 아, 이거는 여기까지만 쓰겠구나 하고, 여기를 지나면 이거에 대해서 신경 안 써도 되죠."

---

## Closing Out the Section: How Java's Exception Philosophy Evolved

The slides end this section with a short retrospective, tying together everything from Lectures 85–101:

- Early Java design favored checked exceptions, believing the compiler-enforced safety net was the better default — hence most of the JDK's original built-in APIs lean heavily on checked exceptions.
- Over time, as applications grew and depended on far more libraries, the number of genuinely unrecoverable exceptions grew faster than the number of recoverable ones — and every checked exception that couldn't be handled locally still had to be tediously declared via `throws` at every layer, compounding as dependencies piled up.
- The tempting-but-dangerous shortcut, `throws Exception`, doesn't actually solve this: it throws every exception indiscriminately, which means the compiler can no longer tell which exception is being caught versus which is being thrown — defeating the entire purpose checked exceptions exist for.
- Because of this, most modern libraries — Spring and JPA explicitly named as examples — default to unchecked (runtime) exceptions instead.
- The resulting practical pattern (this section's running thread, start to finish): catch and handle runtime exceptions when genuinely needed, let them propagate naturally otherwise, and resolve whatever can't be handled anywhere else through one common exception-handling point.

> "런타임 예외도 필요하면 잡을 수 있기 때문에 필요한 경우에는 잡아서 처리하고, 그렇지 않으면 자연스럽게 던지도록 둔다. 그리고 처리할 수 없는 예외는 예외를 공통으로 처리하는 부분을 만들어서 해결하면 된다."

---

## Summary

- Try-with-resources (Java 7+) eliminates the repeated "acquire in `try`, release in `finally`" pattern: a resource implementing `AutoCloseable` can be declared directly inside `try (...)`, and its `close()` is called automatically the instant the `try` block ends.
- `NetworkClientV5` implements `AutoCloseable`; its `close()` override logs a message and calls the same `disconnect()` used by earlier versions' `finally` blocks — and omits `throws Exception` since this particular implementation never throws.
- `NetworkServiceV5` declares `client` inside `try (...)`, which both removes the need for a `finally` block entirely and narrows `client`'s scope to just the `try` block — any code needing `client` (like `initError(data)`) must move inside that block.
- The runtime trace across `hello`/`error1`/`error2` confirms the key behavioral difference from `finally`: `close()` fires **immediately when the `try` block is exited — before any matching `catch` block runs** — whereas plain `finally` only runs *after* `catch` has already finished.
- This earlier release is the core practical advantage over `try/catch/finally`: the resource is freed the moment it's no longer needed, not after unrelated `catch`-block logic finishes running.
- Try-with-resources' four benefits: prevents resource leaks (no forgotten `finally`/`close()`), produces more concise/readable code, narrows a resource's scope strictly to the `try` block (easier debugging), and releases resources earlier than `finally` would.
- This closes out Section 11's running example and the whole exception-handling arc: early Java favored checked exceptions, but growing unrecoverable-exception counts and `throws`-declaration overhead pushed the ecosystem (Spring, JPA, and most modern libraries) toward unchecked exceptions by default, paired with a single common exception-handling point for whatever can't be resolved locally.
