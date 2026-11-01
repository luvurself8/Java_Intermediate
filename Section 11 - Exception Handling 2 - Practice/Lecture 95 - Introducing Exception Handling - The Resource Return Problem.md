# Lecture 95 — Exception Handling 2 - Practice: Introducing Exception Handling - The Resource Return Problem

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 11. 예외 처리2 - 실습 (4/11)

Previous: [Lecture 94 - Introducing Exception Handling - Separating Normal and Exception Flow](Lecture%2094%20-%20Introducing%20Exception%20Handling%20-%20Separating%20Normal%20and%20Exception%20Flow.md)

## The Obvious-Looking Fix

The open problem from last lecture: `client.disconnect()` sat as the last line *inside* the `try` block, so an exception from `connect()`/`send()` always skipped it.

The seemingly obvious fix: **move `disconnect()` outside the `try`/`catch` entirely, as the very last line of the method.** Copy `NetworkServiceV2_3` forward to `NetworkServiceV2_4`.

---

## `NetworkServiceV2_4`: Move `disconnect()` After the `try`/`catch`

```java
package exception.ex2;

public class NetworkServiceV2_4 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV2 client = new NetworkClientV2(address);
        client.initError(data); //추가

        try {
            client.connect();
            client.send(data);
        } catch (NetworkClientExceptionV2 e) {
            System.out.println("[오류] 코드: " + e.getErrorCode() + ", 메시지: " + e.getMessage());
        }

        client.disconnect();
    }
}
```

- `client.disconnect()` is no longer inside the `try`/`catch` at all — it's the final statement of `sendMessage(...)`, placed *after* the whole `try`/`catch` block.
- The reasoning: whether the `try` block finishes normally, or an exception is thrown and the `catch` block handles it, execution always eventually reaches the line right after the `try`/`catch` construct — so `disconnect()` placed there should always run.

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

- `hello`: `try` completes with no exception, falls through to `disconnect()` below it.
- `error1`: `connect()` throws, jumps to `catch`, `catch` finishes, falls through to `disconnect()` below it.
- `error2`: `connect()` succeeds, `send()` throws, jumps to `catch`, `catch` finishes, falls through to `disconnect()`.
- In all three cases, `"서버 연결 해제"` now prints — this approach does appear to work.

---

## But There's a Catch: What If `catch` Can't Catch the Exception?

> "지금과 같은 방식에는 큰 문제가 있습니다. 바로 catch에서 잡을 수 없는 예외가 발생할 때."

To demonstrate, `NetworkClientV2.send(...)` is temporarily modified to throw an unrelated, unexpected exception instead of the declared one:

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

- `RuntimeException` is used here to stand in for "some unanticipated error nobody wrote a `catch` for" — a stand-in for anything that could go wrong that wasn't planned for.
- Being a `RuntimeException`, it's unchecked — `send(...)` doesn't need (and doesn't declare) `throws RuntimeException`. The compiler has no way to flag that this method can now throw something new.

### Run with the substituted `RuntimeException`

```
전송할 문자: error2
http://example.com 서버 연결 성공
Exception in thread "main" java.lang.RuntimeException: ex
	at exception.ex2.NetworkClientV2.send(NetworkClientV2.java:25)
	at exception.ex2.NetworkServiceV2_4.sendMessage(NetworkServiceV2_4.java:13)
	at exception.ex2.MainV2.main(MainV2.java:22)
```

Note: `"서버 연결 해제"` **does not print** this time.

Walking through why:

- `connect()` succeeds. `send(data)` throws — but this time it's `RuntimeException`, not `NetworkClientExceptionV2`.
- The `catch (NetworkClientExceptionV2 e)` block only matches `NetworkClientExceptionV2` (and its subtypes) — `RuntimeException` is unrelated to it in the hierarchy, so this `catch` simply doesn't apply.
- Since nothing in `sendMessage(...)` catches it, the exception propagates straight out of the method — **skipping every line after the point it was thrown, including `client.disconnect()` sitting right after the `try`/`catch`.**
- It keeps propagating (`sendMessage` → `main`), is never caught anywhere, and terminates the program with the stack trace shown above.

> "이거를 캐치를 못해요. 캐치는 런타임 예외랑 전혀 상관없죠. 그래서 여기서 나가버리니까 disconnect() 호출이 안 된 거예요."

This exposes the real flaw in last lecture's fix: placing `disconnect()` right after `try`/`catch` only guarantees it runs for exceptions that the `catch` actually catches. Any exception type the `catch` clause *doesn't* match bypasses that line entirely.

---

## Restoring the Code

> "주의! 실행을 완료했다면 기존 예제들이 정상 작동하도록 코드를 복구해둡니다."

Before moving on, `NetworkClientV2.send(...)` is restored to its working form — the `RuntimeException` line commented back out, the original `throw new NetworkClientExceptionV2(...)` restored — so later lectures in this section continue to run correctly:

```java
public void send(String data) throws NetworkClientExceptionV2 {
    if (sendError) {
        throw new NetworkClientExceptionV2("sendError", address + " 서버에 데이터 전송 실패: " + data);
        //중간에 다른 예외가 발생했다고 가정
        //throw new RuntimeException("ex");
    }
    //전송 성공
    System.out.println(address + " 서버에 데이터 전송: " + data);
}
```

---

## There's No Clean Way to Guarantee This With Only `try`/`catch`

> "결론적으로는 방법이 없고, 뭔가 이거를 깔끔하게 해결할 수 있는 방법 자체가 존재하지 않아요."

- Guaranteeing `disconnect()` always runs would require anticipating *every* possible exception type that could ever be thrown from inside the `try` — including types nobody wrote a `catch` for, because they weren't anticipated at all. That's not something a developer can realistically enumerate in advance.
- Even brainstorming workarounds (catching broader types, wrapping things in more `try`/`catch`) doesn't actually close the gap — any exception path that isn't explicitly caught still skips straight past `disconnect()` on its way out of the method.
- **Guaranteeing "this must run, no matter what happens" is not something `try`/`catch` alone can express.** A different language feature is needed — one built specifically for "always run this, success or failure, caught or uncaught."

That feature — `finally` — is introduced next lecture.

---

## Summary

- Moving `client.disconnect()` to immediately after the `try`/`catch` block (rather than inside `try`) looked like a fix, and it does work for the exception types the `catch` clause actually matches — `hello`, `error1`, and `error2` (with `NetworkClientExceptionV2`) all printed `"서버 연결 해제"` correctly.
- The fix breaks down the moment an exception type the `catch` doesn't match is thrown — demonstrated by swapping in a plain `RuntimeException` inside `send(...)`. Since `catch (NetworkClientExceptionV2 e)` doesn't match `RuntimeException`, the exception propagates straight out of `sendMessage(...)`, skipping `disconnect()` entirely and crashing the program.
- This reveals a structural limit, not just a bug in this one version: with ordinary `try`/`catch`, any code placed after the block only runs for exception types that got caught — anything else bypasses it on the way out.
- There is no way to fully guarantee "always run this cleanup code" using `try`/`catch` alone, because it's impossible to anticipate and explicitly catch every exception type that could ever occur.
- This motivates `finally` — a block guaranteed to execute regardless of whether the `try` succeeded, an anticipated exception was caught, or an unanticipated exception propagated straight through uncaught — covered starting next lecture.
