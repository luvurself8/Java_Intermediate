# Lecture 90 — Exception Handling 1 - Theory: Java Exception Handling - Checked Exception

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 10. 예외 처리1 - 이론 (6/7)

Previous: [Lecture 89 - Java Exception Handling - Basic Exception Rules](Lecture%2089%20-%20Java%20Exception%20Handling%20-%20Basic%20Exception%20Rules.md)

## Checked Exceptions, Defined

- `Exception` and everything beneath it in the hierarchy are **checked exceptions** — with one carve-out, `RuntimeException` (covered next lecture).
- A checked exception **must** either be caught and handled, or declared as thrown outward — otherwise the code doesn't compile.
- Time to build this hands-on. New package: `exception.basic.checked`.

---

## Creating a Checked Exception Class

```java
package exception.basic.checked;

/**
 * Exception을 상속받은 예외는 체크 예외가 된다.
 */
public class MyCheckedException extends Exception {
    public MyCheckedException(String message) {
        super(message);
    }
}
```

- To create a custom exception, just extend some existing exception class. Extending `Exception` specifically makes it a **checked** exception.
- The constructor takes a `message` and forwards it straight to `super(message)` — i.e. `Exception`'s own constructor.
- Why forward to `super` instead of storing the message as a field yourself: `Throwable` (an ancestor of `Exception`) already has a built-in `detailMessage` field for exactly this. Passing the message up through `super(...)` stores it there, and `getMessage()` (inherited, also from `Throwable`) retrieves it. No need to reinvent this — the parent class already provides it.

```
Object → Throwable (has `detailMessage`) → Exception → MyCheckedException
```

---

## `Client`: Throwing the Exception

```java
package exception.basic.checked;

public class Client {
    public void call() throws MyCheckedException {
        //문제 상황
        throw new MyCheckedException("ex");
    }
}
```

- `throw new MyCheckedException("ex")`: this is where the exception actually "detonates." Since an exception is just an object, you first construct it with `new`, then detonate it with `throw`.
- `throws MyCheckedException` next to the method signature: this is a **declaration**, not the detonation itself — it says "if this exception comes up inside me, I can't resolve it, so I'm throwing it out to whoever calls me."

> `throw` is the keyword that actually sets off the bomb. `throws` is the keyword that declares "I can't handle this, I'm throwing it to my caller." Don't confuse the two.

- For a checked exception specifically, this `throws` declaration on the method is **mandatory** if the method doesn't catch the exception itself — without it, the code won't compile.

---

## `Service`: Two Ways to Respond — Catch It, or Throw It Further

```java
package exception.basic.checked;

public class Service {
    Client client = new Client();

    /**
     * 예외를 잡아서 처리하는 코드
     */
    public void callCatch() {
        try {
            client.call();
        } catch (MyCheckedException e) {
            //예외 처리 로직
            System.out.println("예외 처리, message=" + e.getMessage());
        }
        System.out.println("정상 흐름");
    }

    /**
     * 체크 예외를 밖으로 던지는 코드
     * 체크 예외는 예외를 잡지 않고 밖으로 던지려면 throws 예외를 메서드에 필수로 선언해야 한다.
     */
    public void catchThrow() throws MyCheckedException {
        client.call();
    }

}
```

- Both methods call `client.call()`, which always detonates `MyCheckedException`.
- `callCatch()`: wraps the call in `try { ... } catch (MyCheckedException e) { ... }`. The `try` block attempts the risky call; if the declared exception type occurs inside it, `catch` intercepts ("defuses") it, and execution continues with whatever comes **after** the `catch` block — in this case, printing `"정상 흐름"`.
- `catchThrow()`: doesn't try to handle anything. It just repeats the same "I can't resolve this, I'm sending it to my caller" declaration with its own `throws MyCheckedException`.
- This is the exact same handle-or-throw choice from Lecture 89, now written in real syntax: `callCatch` chooses to catch; `catchThrow` chooses to throw further.

---

## Case 1: Catching and Handling — `CheckedCatchMain`

```java
package exception.basic.checked;

public class CheckedCatchMain {

    public static void main(String[] args) {
        Service service = new Service();
        service.callCatch();
        System.out.println("정상 종료");
    }
}
```

### Run

```
예외 처리, message=ex
정상 흐름
정상 종료
```

Walking through the call chain:

1. `main()` → `service.callCatch()` → `client.call()` — exception **occurs and is thrown** out of `call()`.
2. `callCatch()`'s `try` block catches it — `catch (MyCheckedException e)` intercepts the bomb, and `e.getMessage()` retrieves `"ex"` (the message passed in at construction).
3. Once caught, the flow is **defused** and becomes normal flow again: `"정상 흐름"` prints, `callCatch()` returns normally, and `main()` continues on to print `"정상 종료"`.

> "예외는 날라오는 겁니다. 그럼 이걸 여기서 잡을 수 있어요. 잡으면 어떻게 되느냐? 원한 로직을 출력한 다음에 이게 정상 흐름으로 바뀌어요."

Because `callCatch()` fully resolved the exception, it never reaches `main()` — that's why `"정상 종료"` prints at all.

### `catch` catches the declared type **and all of its subtypes**

Changing `catch (MyCheckedException e)` to `catch (Exception e)` still works — and still catches the `MyCheckedException` that was thrown:

```java
public void callCatch() {
    try {
        client.call();
    } catch (Exception e) {
        //예외 처리 로직
    }
    System.out.println("정상 흐름");
}
```

- `MyCheckedException` is a child of `Exception` — a parent-typed `catch` can hold a child-typed exception, same polymorphism rule as anywhere else in Java.
- The reverse isn't guaranteed: if `catch` names a type unrelated to what was actually thrown (e.g. only catching `RuntimeException` when a `MyCheckedException` was thrown), the compiler flags it — `UnhandledException` / "this still isn't handled."
- If exact precision matters — only catching `MyCheckedException` and nothing broader — declare `catch (MyCheckedException e)` specifically.

---

## Case 2: Throwing and Never Catching — `CheckedThrowMain`

```java
package exception.basic.checked;

public class CheckedThrowMain {

    public static void main(String[] args) throws MyCheckedException {
        Service service = new Service();
        service.catchThrow();
        System.out.println("정상 종료");
    }
}
```

### Run

```
Exception in thread "main" exception.basic.checked.MyCheckedException: ex
	at exception.basic.checked.Client.call(Client.java:5)
	at exception.basic.checked.Service.catchThrow(Service.java:28)
	at exception.basic.checked.CheckedThrowMain.main(CheckedThrowMain.java:7)
```

Walking through the call chain:

1. `main()` → `service.catchThrow()` → `client.call()` — exception occurs, thrown.
2. `catchThrow()` doesn't catch it either, so it throws it further out, to `main()`.
3. `main()` itself doesn't catch it, so it propagates out of `main()`.
4. Since the exception exits `main()` unhandled, Java prints the exception log and **terminates the program** — notice `"정상 종료"` never prints. The exception leaving the method means execution never reaches any code after the call that threw it.

### Reading the stack trace

- The printed block above is a **stack trace** — it shows exactly where the exception first occurred and the full path of calls it propagated through.
- Reading top to bottom: `MyCheckedException` originated at `Client.call` (line 5), propagated through `Service.catchThrow` (line 28), and finally out through `CheckedThrowMain.main` (line 7).
- This is one of the most useful debugging tools available: when something breaks, read the stack trace first — it tells you exactly where it broke and the exact path it traveled to get there, which narrows down the problem area immediately instead of guessing.

### What happens if `main()` doesn't declare `throws MyCheckedException`

```java
public void callThrow() {
    client.call();   // compile error
}
```

Without the `throws` declaration, this produces a compile error:

```
java: unreported exception exception.basic.checked.MyCheckedException;
  must be caught or declared to be thrown
```

- `client.call()` is declared `throws MyCheckedException`. Anyone calling it is therefore forced to make a choice: catch it, or declare `throws` themselves.
- Skipping both isn't allowed — the compiler won't let the code build at all.

### Throwing via a parent type also works

```java
public void callThrow() throws Exception {
    client.call();
}
```

- Declaring `throws Exception` (the parent of `MyCheckedException`) instead of `throws MyCheckedException` compiles and runs fine — `MyCheckedException` is still thrown along with it, since the parent type covers all its children.
- Trade-off to be aware of: declaring the broader `throws Exception` tells callers "some `Exception` might come out of this," without being specific about which one — so they may end up having to catch `Exception` generically rather than something precise. If precision matters, declare the exact type (`throws MyCheckedException`).
- This mirrors the `catch` behavior from earlier: declaring (or catching) a parent type covers the child type too, because exceptions are objects and polymorphism applies.

---

## Checked Exceptions: Pros and Cons

| | |
|---|---|
| **Pro** | The compiler forces every checked exception to be either caught or declared as thrown — a developer simply cannot forget to acknowledge it. If code is missing this, it won't even compile, so the gap is caught immediately rather than discovered later in production. |
| **Con** | In real projects, this means *every* checked exception — even ones you don't particularly care about — has to be explicitly caught or declared somewhere. This can get tedious fast when there are many such exceptions scattered through a call chain. |

> "자바 컴파일러님께서: 너 이거는 이렇게 적어주면 내가 던지겠다 이거 안 적어주면 나 이거 진행시킬 수 없어! 라고 해서 컴파일 오류를 내버립니다."

---

## Summary

- A custom exception class is created simply by extending an existing exception type — extending `Exception` (and not `RuntimeException`) makes it a checked exception.
- `super(message)` forwards the error message up to `Throwable`'s built-in `detailMessage` field; `getMessage()` retrieves it later — no need to store it manually.
- `throw new X(...)` is the actual act of raising an exception (constructing the object, then detonating it); `throws X` on a method signature is the *declaration* that this method can't resolve the exception and is sending it to its caller.
- `try { ... } catch (X e) { ... }` catches `X` and lets execution fall through to whatever follows the `catch` block — the flow is "defused" back to normal.
- `catch` (and `throws`) on a parent exception type also covers every subtype beneath it, thanks to ordinary object polymorphism.
- If a checked exception is never caught and propagates all the way out of `main()`, Java prints a **stack trace** (origin point + full propagation path) and terminates — reading that trace top-to-bottom is the fastest way to locate where a bug actually started.
- Checked exceptions must be explicitly caught or declared via `throws`, or the code fails to compile — a strong safety net against forgotten error handling, at the cost of extra boilerplate for exceptions a developer may not actually care much about. This tension is exactly what unchecked (runtime) exceptions, covered next, exist to address.
