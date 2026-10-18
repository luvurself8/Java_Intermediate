# Lecture 75 — Nested Classes, Inner Classes 2: Local Class - Local Variable Capture 1

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 09. 중첩 클래스, 내부 클래스2 (2/11)

Previous: [Lecture 74 - Local Class - Getting Started](Lecture%2074%20-%20Local%20Class%20-%20Getting%20Started.md)

This is the first of a three-part deep dive into what the instructor calls a topic almost no book
or course properly explains: **how** a local class can keep accessing a local variable that, by
every rule learned in the basics course, should already be gone.

> 참고: 지금부터 설명할 지역 변수 캡처에 관한 내용은 너무 깊이있게 이해하지 않아도 된다. 이해가
> 어렵다면 단순하게 지역 클래스가 접근하는 지역 변수의 값은 변경하면 안된다 정도로 이해하면 충분하다.
> (Note: the upcoming content on local variable capture doesn't need to be understood too deeply.
> If it's hard to follow, it's enough to simply understand: don't change the value of a local
> variable that a local class accesses.)

The instructor is explicit that this is presented in depth specifically because it's a nagging
question that trips up most people learning local classes, and almost nothing explains it well —
so this lecture builds the mystery, and the resolution comes in Lecture 76.

---

## Refresher: Variable Lifecycles

Before touching the mystery, the lecture reviews how long each kind of variable lives:

| Variable kind | Lives in | Lifespan |
|---|---|---|
| Class variable (`static`) | Method area | From class loading until the program ends — the longest |
| Instance variable | Heap | As long as its owning instance survives — until that instance is GC'd |
| Local variable (incl. parameters) | Stack (inside a stack frame) | Only while the method call is executing |

- A local variable lives inside its method's **stack frame**. The frame is created when the method
  is called and destroyed the instant the method call ends — and every local variable inside it is
  destroyed along with it.
- This gives local variables by far the shortest lifespan of the three.
- A parameter is explicitly called out again as just a special case of a local variable, so it
  follows the exact same rule.

---

## Example 3: Returning the Local Class Instance Instead of Using It Immediately

This example deliberately restructures the earlier `process()` method so that the local class
instance **outlives** the method that created it.

```java
package nested.local;

public class LocalOuterV3 {

    private int outInstanceVar = 3;

    public Printer process(int paramVar) {

        int localVar = 1; //지역 변수는 스택 프레임이 종료되는 순간 함께 제거된다.

        class LocalPrinter implements Printer {
            int value = 0;

            @Override
            public void print() {
                System.out.println("value=" + value);

                //인스턴스는 지역 변수보다 더 오래 살아남는다.
                System.out.println("localVar=" + localVar);
                System.out.println("paramVar=" + paramVar);
                System.out.println("outInstanceVar=" + outInstanceVar);
            }
        }

        LocalPrinter printer = new LocalPrinter();

        //printer.print();를 여기서 실행하지 않고 Printer 인스턴스만 반환한다.
        return printer;
    }

    public static void main(String[] args) {
        LocalOuterV3 localOuter = new LocalOuterV3();
        Printer printer = localOuter.process(2);
        //printer.print()를 나중에 실행한다. process()의 스택 프레임이 사라진 이후에 실행
        printer.print();
    }
}
```

```
value=0
localVar=1
paramVar=2
outInstanceVar=3
```

The key structural change from Examples 1 and 2:

- `process()` now returns the `Printer` type (the interface), instead of calling `print()` itself.
- Inside `process()`, `LocalPrinter` is constructed but **never invoked** — only the reference is
  returned.
- `main()` receives that reference into its own `printer` variable, and only calls
  `printer.print()` *after* `process()` has already returned — i.e., **after `process()`'s stack
  frame is already gone.**

> 이 예제를 실행하면서 뭔가 이상한 느낌이 들었다면 제대로 공부를 하고 있는 것이다. (If running this
> example gives you a strange, unsettling feeling, you're studying it properly.)

---

## Walking Through Memory: Right After `LocalPrinter` Is Created

Picture the moment `process()` has just executed `new LocalPrinter()`, inside its call from
`main()`:

- **Stack area**: `process()`'s stack frame holds its three local variables — `paramVar`,
  `localVar`, and `printer` (the reference to the newly created `LocalPrinter`).
- **Heap area**: the `LocalOuterV3` instance (call it `x001`) holds `outInstanceVar`. The new
  `LocalPrinter` instance (call it `x002`) also lives here.
- Because `LocalPrinter` is a local class — and therefore an inner class — it automatically holds
  a reference back to the specific `LocalOuterV3` instance it was created from, exactly as
  described for inner classes in Lecture 71. `x002` references `x001`.

### How Long Does Each Piece Survive?

- `LocalPrinter`'s instance (`x002`) is a heap object just like any other instance — it survives
  until garbage collected.
- `process()` returns `printer` (pointing at `x002`), and `main()` stores that reference in its own
  `printer` variable. As long as `main()` keeps holding that reference, `x002` **cannot be
  garbage collected** — it survives all the way until `main()` itself finishes.
- Meanwhile, `paramVar` and `localVar` only exist inside `process()`'s stack frame. The instant
  `process()` returns, that stack frame — and both local variables inside it — are destroyed.

> 로컬 프린터 인스턴스는 메인이 종료될 때까지 지금 계속 생존합니다. 왜냐하면 메인에서 지금 x002에
> 대한 참조를 계속 가지고 있죠. (The `LocalPrinter` instance keeps surviving until `main` finishes,
> because `main` keeps holding a reference to `x002`.)

---

## Inside `print()`: One Access Is Fine, Two Should Be Impossible

Looking at what `LocalPrinter.print()` tries to read:

### `outInstanceVar` — no problem

- This reaches `x001` (the `LocalOuterV3` instance) through the stored back-reference that every
  inner class keeps.
- `x001` is a heap object, independently alive — this is ordinary inner-class instance-field
  access, nothing unusual.

### `localVar` and `paramVar` — this *should* be impossible

- These are local variables that only ever lived inside `process()`'s now-destroyed stack frame.
- By the time `print()` actually runs (called from `main()`, *after* `process()` has already
  returned), `process()`'s stack frame is gone, and `localVar`/`paramVar` were destroyed along
  with it.
- And yet — the program runs, and both values print out correctly: `localVar=1`, `paramVar=2`.

> 어떻게 제거된 지역 변수들에 접근할 수 있는 것일까? (How is it possible to access local variables
> that have already been removed?)

---

## The Cliffhanger

The instructor closes by sharpening exactly how strange this is, and adds one refinement to the
mental model:

- To be fully precise: when `main()` calls `printer.print()`, that call gets **its own stack
  frame**, stacked on top of `main()`'s frame — the same way any method call works.
- But this doesn't change the core puzzle at all: `process()`'s stack frame is *still* long gone
  by the time `print()`'s frame exists, so `localVar` and `paramVar` were destroyed regardless of
  which frame is currently on top of the stack.
- The values are being read out and printed correctly despite this — which is exactly the mystery
  the next lecture resolves.

## Summary

- Local variables (stack) have a dramatically shorter lifespan than instances (heap): a local
  variable dies the instant its method call ends, while an instance survives until garbage
  collected.
- `LocalOuterV3.process()` is restructured to *return* its `LocalPrinter` instance rather than
  call `print()` on it directly — so the instance is used only after `process()` has already
  returned and its stack frame is destroyed.
- The `LocalPrinter` instance itself survives fine past `process()`'s return, because `main()`
  keeps a live reference to it.
- Accessing `outInstanceVar` (an instance field on the outer object) through the inner class's
  stored back-reference is completely unremarkable — ordinary inner-class behavior.
- Accessing `localVar` and `paramVar` *after* `process()`'s stack frame has already been destroyed
  **should**, by every rule covered so far, be impossible — yet the program runs and prints the
  correct values anyway.
- This deliberate contradiction is the setup for Lecture 76, which reveals the actual mechanism —
  **local variable capture** — that makes this work.
