# Lecture 85 — Exception Handling 1 - Theory: Need for Exception Handling - Getting Started

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 10. 예외 처리1 - 이론 (1/7)

## Why This Section Exists

- Exception handling is a big topic, and the course builds it up through **one running example that keeps evolving**, lecture after lecture.
- The instructor stresses: don't fall behind on this example. Code it yourself, run it, and fully understand it before moving to the next lecture — later lectures keep extending this exact same code.
- This first lecture just sets the stage: a working program with **no error handling at all**, so the next lectures can show what goes wrong and why.

---

## The Scenario

The example program:

- Takes a string typed by the user at the keyboard.
- Sends that string to an external server over the network.
- (The course hasn't covered networking yet, so there's no real socket code — the network calls are simulated with `println` statements that pretend a connection/send/disconnect happened.)

### Program structure

```
사용자 입력 → Main --sendMessage(data)--> NetworkService --connect()/send(data)/disconnect()--> NetworkClient --(data)--> 외부 서버
```

- **`NetworkClient`**: the lowest-level class. Knows how to `connect()` to the external server, `send(data)` to it, and `disconnect()` from it.
- **`NetworkService`**: a wrapper around `NetworkClient`. Using `NetworkClient` correctly requires calling `connect → send → disconnect` in the right order — `NetworkService` hides that sequencing behind a single `sendMessage(data)` method.
- **`Main`**: reads user input in a loop and forwards each line to `NetworkService.sendMessage(...)`.

> The whole point of `NetworkService` is that callers shouldn't have to know the connect/send/disconnect protocol — they just call `sendMessage` once.

### Usage rules for `NetworkClient` (important — these come back later)

- Call `connect()` first. Only if it **succeeds** should you call `send(data)`.
- After you're done, you must **always** call `disconnect()` — even if `connect()` or `send()` failed partway through. Otherwise connections can leak and pile up.

These two rules are not enforced by the code yet in this lecture. That gap is exactly what later lectures in this section exist to fix.

---

## Building V0: the Happy-Path-Only Version

Package: `exception.ex0`. Everything in this first version assumes success — no failure cases are modeled yet (those come in the next lecture).

### `NetworkClientV0`

```java
package exception.ex0;

public class NetworkClientV0 {

    private final String address;

    public NetworkClientV0(String address) {
        this.address = address;
    }

    public String connect() {
        //연결 성공
        System.out.println(address + " 서버 연결 성공");
        return "success";
    }

    public String send(String data) {
        //전송 성공
        System.out.println(address + " 서버에 데이터 전송: " + data);
        return "success";
    }

    public void disconnect() {
        System.out.println(address + " 서버 연결 해제");
    }
}
```

- `address`: the external server's address, set once through the constructor.
- `connect()` / `send(data)` / `disconnect()` each just print what "would" happen and (for `connect`/`send`) return the string `"success"`.
- Returning a plain `String` result code (`"success"`) instead of throwing anything is the detail to pay attention to — it's the seed of the problem the rest of this section digs into.

### `NetworkServiceV0`

```java
package exception.ex0;

public class NetworkServiceV0 {

    public void sendMessage(String data) {
        String address = "http://example.com";
        NetworkClientV0 client = new NetworkClientV0(address);

        client.connect();
        client.send(data);
        client.disconnect();
    }
}
```

- Creates the `NetworkClientV0`, passing in the hardcoded external server address.
- Calls `connect()`, `send(data)`, and `disconnect()` in sequence — this is the "complex usage flow" that `NetworkService` exists to hide from the caller.

### `MainV0`

```java
package exception.ex0;

import java.util.Scanner;

public class MainV0 {

    public static void main(String[] args) {
        NetworkServiceV0 networkService = new NetworkServiceV0();

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

- Reads a line from the console with `Scanner`.
- Typing `exit` breaks out of the loop and ends the program.
- Any other line is forwarded straight to `networkService.sendMessage(input)`.

### Run

```
전송할 문자: hello
http://example.com 서버 연결 성공
http://example.com 서버에 데이터 전송: hello
http://example.com 서버 연결 해제

전송할 문자: exit
프로그램을 정상 종료합니다.
```

- Each message typed in produces a connect → send → disconnect trio of print lines.
- The instructor is explicit that this V0 code is not meant to be production-quality (there's plenty that could be improved) — the goal here is purely to understand *how the pieces fit together*, because this exact code is the baseline the rest of the section incrementally improves.

---

## Summary

- The running example for this whole section: `Main` reads user input → `NetworkService.sendMessage(...)` → `NetworkClient.connect()/send()/disconnect()`.
- `NetworkService` exists to hide the correct connect/send/disconnect sequencing that `NetworkClient` requires.
- Correct `NetworkClient` usage has two rules that aren't enforced yet: don't `send()` after a failed `connect()`, and always call `disconnect()` no matter what.
- V0 (`exception.ex0`) only models the success path — `connect()`/`send()` always return `"success"`, no failures are possible yet.
- This exact V0 code must be understood and recreated by hand before continuing — later lectures evolve it step by step to introduce and then fix error handling.
