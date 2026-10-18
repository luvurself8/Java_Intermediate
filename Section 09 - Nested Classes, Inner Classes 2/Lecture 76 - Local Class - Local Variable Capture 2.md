# Lecture 76 — Nested Classes, Inner Classes 2: Local Class - Local Variable Capture 2

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 09. 중첩 클래스, 내부 클래스2 (3/11)

Previous: [Lecture 75 - Local Class - Local Variable Capture 1](Lecture%2075%20-%20Local%20Class%20-%20Local%20Variable%20Capture%201.md)

The mystery left open at the end of Lecture 75 — how `LocalPrinter.print()` still reads
`localVar` and `paramVar` correctly after `process()`'s stack frame has already been destroyed —
gets resolved here. The answer is a mechanism called **local variable capture**.

## The Problem, Restated

- A local class can access local variables — but local variables live on the stack and die the
  instant their method call ends.
- An instance created from that local class lives on the heap and can easily outlive the method
  call that created it (as shown in Lecture 75).
- So: by the time the instance actually needs to read a local variable, that variable's stack
  frame may already be gone. There's a mismatch between the two lifespans that somehow has to be
  resolved.

## The Solution: Local Variable Capture

> 자바는 이런 문제를 해결하기 위해 지역 클래스의 인스턴스를 생성하는 시점에 필요한 지역 변수를 복사해서
> 생성한 인스턴스에 함께 넣어둔다. 이런 과정을 변수 캡처(Capture)라 한다. (To solve this, Java
> copies the local variables it needs — at the moment the local class's instance is created — and
> stores them together inside that instance. This process is called variable capture.)

- The word "capture" is meant to evoke a screen capture: at a specific instant, you take a
  snapshot copy of something and keep it, independent of what happens to the original afterward.
- Only the local variables the local class actually accesses are captured — not every local
  variable in scope. (`LocalPrinter` only ever reads `paramVar` and `localVar`, so only those two
  get captured.)

### The Capture Process, Step by Step

1. **Instance creation begins**: when `new LocalPrinter()` runs, Java checks which local variables
   the local class actually accesses — here, `paramVar` and `localVar`.
2. **Copy the used local variables**: the values of those local variables (on the stack) are
   copied. (Remember: a parameter is just a kind of local variable, so it's copied the same way.)
3. **Store the copies inside the instance**: the copied values are bundled in as part of the new
   `LocalPrinter` instance itself.
4. **Instance creation completes**: the instance now exists complete with its own private copies
   of those variables, reachable through the instance from now on.

### What This Means for Access

> `LocalPrinter` 인스턴스에서 `print()` 메서드를 통해 `paramVar`, `localVar`에 접근하면 사실은
> 스택 영역에 있는 지역 변수에 접근하는 것이 아니다. 대신에 인스턴스에 있는 캡처한 변수에 접근한다.
> (When `print()` accesses `paramVar`/`localVar` through the `LocalPrinter` instance, it is not
> actually reaching into the stack-area local variables. Instead, it reaches the captured copies
> stored in the instance.)

- This is the key realization: `print()` was never reading the original stack-based
  `paramVar`/`localVar` all along — it reads its own private copies, which live inside the
  instance itself.
- Because the captured copies are now just ordinary fields of the `LocalPrinter` instance, they
  share that instance's lifecycle — they live exactly as long as the instance does, completely
  independent of whatever happened to `process()`'s stack frame.

> 캡처한 `paramVar`, `localVar`의 생명주기는 `LocalPrinter` 인스턴스의 생명주기와 같다. 따라서
> `LocalPrinter` 인스턴스는 지역 변수의 생명주기와 무관하게 언제든지 `paramVar`, `localVar` 캡처
> 변수에 접근할 수 있다. (The captured copies' lifespan matches the `LocalPrinter` instance's
> lifespan. So the instance can access these captured variables whenever it wants, regardless of
> the original local variables' own lifespan.)

---

## Seeing It With Your Own Eyes: Reflection on the Captured Fields

The instructor adds a debugging step to the `main()` method from Lecture 75's `LocalOuterV3`, to
prove the captured variables really exist as hidden fields on the instance.

```java
public static void main(String[] args) {
    LocalOuterV3 localOuter = new LocalOuterV3();
    Printer printer = localOuter.process(2);
    //printer.print()를 나중에 실행한다. process()의 스택 프레임이 사라진 이후에 실행
    printer.print();

    //추가
    System.out.println("필드 확인");
    Field[] fields = printer.getClass().getDeclaredFields();
    for (Field field : fields) {
        System.out.println("field = " + field);
    }
}
```

```
필드 확인

//인스턴스 변수
field = int nested.local.LocalOuterV3$1LocalPrinter.value

//캡처 변수
field = final int nested.local.LocalOuterV3$1LocalPrinter.val$localVar
field = final int nested.local.LocalOuterV3$1LocalPrinter.val$paramVar

//바깥 클래스 참조
field = final nested.local.LocalOuterV3 nested.local.LocalOuterV3$1LocalPrinter.this$0
```

- `printer.getClass().getDeclaredFields()` lists every field the JVM actually generated for the
  `LocalPrinter` class — not just the ones visible in the source code.
- The source code only shows **one** field, `value` — but the reflection output reveals **four**:

| Field shown by reflection | What it actually is |
|---|---|
| `value` | The field explicitly written in the source code |
| `val$localVar` | The hidden, captured copy of `localVar` |
| `val$paramVar` | The hidden, captured copy of `paramVar` |
| `this$0` | The hidden reference back to the owning `LocalOuterV3` instance |

- The `val$` prefix is the compiler's internal naming convention for a captured local variable —
  this is literally the mechanism described above, made visible.
- `this$0` is the same "reference back to the owning outer instance" mechanism every inner class
  carries (established back in Lecture 71) — here it's proof that a local class really does carry
  this field too, since a local class is a kind of inner class.
- Note that both captured fields are declared `final` in the generated bytecode — this detail is
  the setup for the next lecture's rule about why a captured local variable must never change.

> 추가로 바깥 클래스를 참조하기 위한 필드도 확인할 수 있었습니다. 참고로 이런 필드들은 자바가 내부에서
> 만들어서 사용하는 필드들입니다. (We could also confirm the field used to reference the outer
> class. Note: these fields are generated and used internally by Java.)

---

## Summary

- A local class instance can outlive the local variables it reads, because Java **captures** those
  variables: at instance-creation time, it copies the values of every local variable the class
  actually accesses and stores those copies as hidden fields inside the new instance.
- Accessing a "local variable" from inside the local class's methods is therefore never really
  reaching back into the (possibly already-destroyed) stack frame — it's reading the instance's
  own private captured copy.
- Captured copies share the instance's lifecycle, not the original local variable's — this is
  exactly what resolves the lifespan mismatch from Lecture 75.
- Using reflection (`getClass().getDeclaredFields()`) on a `LocalPrinter` instance reveals hidden
  compiler-generated fields beyond what's visible in the source: `val$localVar` and `val$paramVar`
  (the captured copies, both `final`) plus `this$0` (the back-reference to the outer instance).
- These generated fields are purely an internal implementation detail that the Java compiler
  manages automatically — you never write or see them directly in source code.
- Next up: why these captured variables must never change — the `final`/effectively-final rule.
