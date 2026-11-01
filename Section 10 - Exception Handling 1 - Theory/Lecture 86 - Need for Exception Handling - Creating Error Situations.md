# Lecture 86 — Exception Handling 1 - Theory: Need for Exception Handling - Creating Error Situations

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 10. 예외 처리1 - 이론 (2/7)

Previous: [Lecture 85 - Need for Exception Handling - Getting Started](Lecture%2085%20-%20Need%20for%20Exception%20Handling%20-%20Getting%20Started.md)

## Why Simulate Errors

Real communication with an external server can fail in all kinds of ways:

- **Connection failure** — the network has a problem, a cable gets physically damaged, the remote server just isn't up yet, and so on. Lots of possible causes.
- **Send failure** — the connection itself succeeds, but something goes wrong while transmitting the data (or the other side never receives it properly).

> "네트워크 오류 때문에 야근한 집에 못 간 기억들이 주마등처럼 스쳐서 웃음이 나오네요." — the instructor, only half-joking about how common and painful network errors are in real jobs.

Since V0 doesn't actually talk to a real network, these failures can't happen naturally. So this lecture **fakes them on purpose** — add switches that can be flipped on to pretend a failure occurred, driven by what the user types in.

---

## Building V1

New package: `exception.ex1`, copied forward from `exception.ex0`. Three classes, same shape as before, but `NetworkClientV1` now has failure switches and `NetworkServiceV1_1` wires them up.

### `NetworkClientV1` — adding failure switches

```java
package exception.ex1;

public class NetworkClientV1 {

    private final String address;
    public boolean connectError;
    public boolean sendError;

    public NetworkClientV1(String address) {
        this.address = address;
    }

    public String connect() {
        if (connectError) {
            System.out.println(address + " 서버 연결 실패");
            return "connectError";
        }

        //연결 성공
        System.out.println(address + " 서버 연결 성공");
        return "success";
    }

    public String send(String data) {
        if (sendError) {
            System.out.println(address + " 서버에 데이터 전송 실패: " + data);
            return "sendError";
        }

        //전송 성공
        System.out.println(address + " 서버에 데이터 전송: " + data);
        return "success";
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

- `connectError` / `sendError`: `public boolean` fields, default `false` (Java's default for `boolean`). Kept `public` on purpose here — this is a quick-and-dirty simulation harness, not production design.
- `connect()`: if `connectError` is `true`, prints a failure message and returns the error code `"connectError"` instead of `"success"`.
- `send(data)`: same idea — if `sendError` is `true`, prints a failure message and returns `"sendError"`.
- `initError(data)`: flips the switches based on substring matching against the user's raw input:
  - input **contains** `"error1"` → `connectError = true`
  - input **contains** `"error2"` → `sendError = true`
- Why `contains` and not `equals`: typing `"Hello Java error1"` still trips the connect-error path, because `error1` just needs to appear *somewhere* in the string. This is intentional — it keeps the simulation forgiving to trigger during testing.

### `NetworkServiceV1_1` — wiring in the simulation

```java
package exception.ex1;

public class NetworkServiceV1_1 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV1 client = new NetworkClientV1(address);
        client.initError(data); //추가

        client.connect();
        client.send(data);
        client.disconnect();
    }
}
```

- Only one line changed from `NetworkServiceV0`: `client.initError(data)` is called right after construction, before `connect()`.
- This is the only place the error switches actually get flipped — `NetworkClientV1` itself doesn't know *when* to simulate failure, it just exposes the switches and `initError` as the trigger.

### `MainV1`

Same shape as `MainV0`, just pointed at `NetworkServiceV1_1` (and `import`s cleaned up — copy/paste from `MainV0` otherwise).

```java
NetworkServiceV1_1 networkService = new NetworkServiceV1_1();
```

### Run

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: error1
http://example.com 서버 연결 실패
http://example.com 서버에 데이터 전송: error1
http://example.com 서버 연결 해제

전송할 문자: error2
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송 실패: error2
http://example.com 서버 연결 해제

전송할 문자: exit
프로그램을 정상 종료합니다.
```

| Input contains | Simulated failure | Error code returned |
|---|---|---|
| `error1` | `connect()` fails | `"connectError"` |
| `error2` | `send()` fails | `"sendError"` |
| neither | everything succeeds | `"success"` |

---

## Remaining Problems

Running `error1` exposes the bug this whole section builds toward:

- **Connection failed, but data still got sent anyway.** Look at the `error1` output above — `서버 연결 실패` prints, but the very next line is still `서버에 데이터 전송: error1`. `NetworkServiceV1_1.sendMessage` calls `connect()`, `send(data)`, and `disconnect()` unconditionally in sequence — it never looks at what `connect()` returned.
  - In a real network, sending data over a connection that was never established is a real problem.
  - This is exactly the usage rule from Lecture 85 being violated: *don't call `send()` if `connect()` failed.*
- **No error logging.** Right now a failure just prints a generic `"...서버 연결 실패"` / `"...데이터 전송 실패"` line. There's no detail captured about *which* error code occurred or *why* — information a developer would need for debugging in a real incident.

These two gaps are exactly what the next lecture (`NetworkServiceV1_2`) fixes, by actually checking the returned result code before proceeding.

---

## Summary

- Real network failures (connect failure, send failure) can't happen against a fake/no-op client, so this lecture **simulates** them with boolean switches on `NetworkClientV1`.
- `connectError` / `sendError` fields default to `false`; `initError(data)` flips them to `true` based on `data.contains("error1")` / `data.contains("error2")`.
- `contains` (not `equals`) means the trigger word just needs to appear anywhere in the input — e.g. `"Hello Java error1"` still trips `connectError`.
- `connect()` / `send()` now return a String result code (`"success"`, `"connectError"`, `"sendError"`) instead of always succeeding.
- `NetworkServiceV1_1.sendMessage` calls `initError`, then still blindly runs `connect() → send() → disconnect()` without checking the result — so a failed `connect()` doesn't stop `send()` from running anyway.
- Two problems carried forward to the next lecture: (1) a failed connection shouldn't let `send()` proceed, and (2) failures need detailed logging, not just a generic print.
