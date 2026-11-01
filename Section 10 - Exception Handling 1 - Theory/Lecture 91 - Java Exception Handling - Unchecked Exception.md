# Lecture 91 — Exception Handling 1 - Theory: Java Exception Handling - Unchecked Exception

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 10. 예외 처리1 - 이론 (7/7)

Previous: [Lecture 90 - Java Exception Handling - Checked Exception](Lecture%2090%20-%20Java%20Exception%20Handling%20-%20Checked%20Exception.md)

## Unchecked Exceptions: Basically the Same, One Difference

- `RuntimeException` and everything beneath it are **unchecked exceptions**. Unchecked literally means "the compiler doesn't check it."
- Everything else about how exceptions behave is identical to checked exceptions covered last lecture: you still either catch-and-handle or let it propagate to the caller, parent-type catch/throws still covers child types, a stack trace still prints if it reaches `main()` unhandled.
- **The one actual difference:** declaring `throws` on the method is **optional** for unchecked exceptions. If omitted, the exception still gets thrown outward automatically — the compiler simply doesn't force you to write it down.

> "언체크 예외는 체크 예외와 기본적으로 동일해요. 그런데 차이가 딱 하나 있어요 — throws를 생략해도 된다는 것. 생략한 경우 예외를 자동으로 밖으로 던집니다."

New package: `exception.basic.unchecked` — same class shapes as the checked example, for direct comparison.

---

## Creating an Unchecked Exception Class

```java
package exception.basic.unchecked;

/**
 * RuntimeException을 상속받은 예외는 언체크 예외가 된다.
 */
public class MyUncheckedException extends RuntimeException {
    public MyUncheckedException(String message) {
        super(message);
    }
}
```

- Only difference from `MyCheckedException`: extends `RuntimeException` instead of `Exception`. That one choice of parent class is what makes it unchecked — a rule Java's grammar defines directly.

## `Client`: No `throws` Needed

```java
package exception.basic.unchecked;

public class Client {
    public void call() {
        throw new MyUncheckedException("ex");
    }
}
```

- `call()` throws `MyUncheckedException` but has **no `throws` declaration** on the signature — and this compiles without any error or warning.
- Compare to the checked version, where leaving off `throws MyCheckedException` was a hard compile error. Here it's simply allowed: the bomb still detonates and travels outward exactly the same way, the compiler just isn't tracking it.

## `Service`: Catch Optionally, Throw Without Declaring

```java
package exception.basic.unchecked;

/**
 * UnChecked 예외는
 * 예외를 잡거나, 던지지 않아도 된다.
 * 예외를 잡지 않으면 자동으로 밖으로 던진다.
 */
public class Service {

    Client client = new Client();

    /**
     * 필요한 경우 예외를 잡아서 처리할 수 있다.
     */
    public void callCatch() {
        try {
            client.call();
        } catch (MyUncheckedException e) {
            //예외 처리 로직
            System.out.println("예외 처리, message=" + e.getMessage());
        }
        System.out.println("정상 로직");
    }

    /**
     * 예외를 잡지 않아도 된다. 자연스럽게 상위로 넘어간다.
     * 체크 예외와 다르게 throws 예외 선언을 하지 않아도 됩니다.
     */
    public void callThrow() {
        client.call();
    }

}
```

- `callCatch()` catches `MyUncheckedException` with ordinary `try`/`catch` — identical mechanics to the checked example. Unchecked exceptions can absolutely still be caught when that's useful.
- `callThrow()` doesn't declare anything at all — no `throws`. It just calls `client.call()` and lets whatever happens, happen. If an exception occurs inside and isn't caught, it automatically propagates to `callThrow()`'s own caller — "자연스럽게 상위로 넘어간다."
- Not declaring `throws` does **not** mean the exception is suppressed or disappears — the bomb still travels, it's just not written down in the method signature.

---

## Case 1: Catch and Handle — `UncheckedCatchMain`

```java
package exception.basic.unchecked;

public class UncheckedCatchMain {

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
정상 로직
정상 종료
```

- Identical shape to the checked exception's `CheckedCatchMain` run — `callCatch()` catches the exception, logs the message, flow resumes normally, and `main()` finishes printing `"정상 종료"`.
- The instructor is explicit: **the execution result is completely identical to the checked exception case.** Catching behaves exactly the same whether the exception is checked or unchecked.

## Case 2: Never Caught — `UncheckedThrowMain`

```java
package exception.basic.unchecked;

public class UncheckedThrowMain {
    public static void main(String[] args) {
        Service service = new Service();
        service.callThrow();
        System.out.println("정상 종료");
    }
}
```

Note `main()` here has **no `throws` declaration at all** — and this still compiles, unlike `CheckedThrowMain` which required `throws MyCheckedException`.

### Run

```
Exception in thread "main" exception.basic.unchecked.MyUncheckedException: ex
	at exception.basic.unchecked.Client.call(Client.java:5)
	at exception.basic.unchecked.Service.callThrow(Service.java:29)
	at exception.basic.unchecked.UncheckedThrowMain.main(UncheckedThrowMain.java:7)
```

- `Client.call()` throws → `Service.callThrow()` doesn't catch it, so it propagates → `main()` doesn't catch it either, so it propagates out of `main()`.
- Since it reached `main()`'s outer boundary unhandled, Java prints the stack trace and terminates — `"정상 종료"` never prints, for the exact same reason as the checked example: once the exception leaves a method, nothing after that call in that method runs.
- The **only** thing that changed from the checked version: none of `Client.call()`, `Service.callThrow()`, or `main()` needed a `throws` declaration anywhere. The runtime behavior — stack trace, program termination — is otherwise identical.

### Declaring `throws` on an unchecked exception is still legal — just optional

```java
public void callThrow() throws MyUncheckedException {
    client.call();
}
```

- Perfectly valid to write `throws MyUncheckedException` even though it's unchecked — but it changes nothing about compiler enforcement.
- The only effect is **documentation/discoverability**: a developer reading or calling this method can see in the signature (or via IDE hints) that this exception might come out of it.
- It is *not* equivalent to the checked-exception guarantee — the compiler still won't force the caller to do anything about it. If the caller ignores it, the unchecked exception still just propagates through, same as if `throws` had been omitted entirely.
- Spring Framework, for example, sometimes documents especially important unchecked exceptions this way. But as a general default, the instructor's guidance is to **omit `throws` for unchecked exceptions** and only write it for exceptions important enough that callers really should be aware of them at a glance.

---

## Unchecked Exceptions: Pros and Cons

| | |
|---|---|
| **Pro** | Unchecked exceptions a developer doesn't want to think about can simply be ignored — no `throws` boilerplate required anywhere in the call chain. |
| **Con** | Precisely because it's optional, a developer can **accidentally** forget that a call might throw an exception — there's no compiler safety net catching the oversight, unlike checked exceptions. |

The instructor demonstrates the con directly: writing a throwaway `hello()` method that calls `client.call()` without realizing it can throw — the code compiles fine, and the mistake only surfaces at runtime when the exception actually fires. With a checked exception, this same mistake is impossible to make — the compiler refuses to build until it's acknowledged.

> "신경 쓰고 싶지 않은 언체크 예외를 무시할 수 있지만, 문제는 누락할 수도 있다는 거예요. 이게 양날의 칼이죠." — the same trait (optional `throws`) is simultaneously the advantage and the risk.

---

## The Real Takeaway: Checked vs. Unchecked Comes Down to One Thing

- Checked and unchecked exceptions are **fundamentally identical** in every other respect — object hierarchy, catching, propagation, stack traces on an unhandled exit.
- The entire distinction boils down to: when you can't handle an exception yourself, is declaring `throws` **mandatory** (checked) or **optional** (unchecked)?
- Modern application development mostly avoids checked exceptions, even though the "compiler safety net" sounds appealing in theory. The instructor flags there's a concrete, practical reason for this industry-wide shift — and promises to explain it in the next lecture (moving on to Section 11, 예외 처리2 - 실습).

---

## Where This Goes Next

With the theory of Java's exception mechanism now fully covered, the course is ready to go back and fix the running `NetworkService` example from Lectures 85–87. Recall where that example was left off:

- `NetworkServiceV1_3` used return-value-based error handling, and normal flow and error-handling flow ended up completely tangled together — described as "지옥" (a mess) by the instructor when re-reading it just now.
- The next section refactors that exact code using real Java exceptions (`try`/`catch`/`throw`/`throws`), so the difference between the two approaches can be seen directly, side by side, in a concrete example rather than just in theory.

---

## Summary

- Unchecked exceptions (`RuntimeException` and its subclasses) behave identically to checked exceptions in every respect except one: declaring `throws` on a method is optional, not compiler-enforced.
- Omitting `throws` doesn't suppress or prevent the exception — it still propagates to the caller automatically if uncaught, exactly like a checked exception would if declared.
- Catching an unchecked exception (`try`/`catch`) works exactly the same way as catching a checked one — same mechanics, same result.
- An unhandled unchecked exception that escapes `main()` prints the same kind of stack trace and terminates the program the same way a checked exception would.
- `throws` can still be written for an unchecked exception as documentation/IDE-hint value, but it carries no compiler enforcement — the default convention is to omit it, reserving it only for exceptions important enough that callers should be alerted at a glance.
- The trade-off is symmetric: skipping `throws` lets developers ignore exceptions they don't care about, but the same lack of compiler enforcement means a genuinely important exception can be forgotten by accident — something a checked exception's compile error would have caught.
- Checked vs. unchecked ultimately differs in exactly one place: whether declaring an unhandled exception's `throws` is mandatory or optional. Everything else about exception semantics is shared. Modern Java development leans heavily toward unchecked exceptions, for practical reasons the course picks up starting next lecture — alongside finally refactoring the running `NetworkService` example with real exception handling.
