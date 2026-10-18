# Lecture 77 — Nested Classes, Inner Classes 2: Local Class - Local Variable Capture 3

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 09. 중첩 클래스, 내부 클래스2 (4/11)

Previous: [Lecture 76 - Local Class - Local Variable Capture 2](Lecture%2076%20-%20Local%20Class%20-%20Local%20Variable%20Capture%202.md)

This is the lecture the instructor has been building toward since the "note" at the very start of
Lecture 75: the rule that a local variable accessed by a local class must be `final` or
effectively final, **and**, more importantly, an explanation of *why* — something the instructor
says is almost never properly explained in books or other courses.

## The Rule

> 지역 클래스가 접근하는 지역 변수는 절대로 중간에 값이 변하면 안 돼요. 따라서 final로 선언하거나 또는
> 사실상 final이어야 됩니다. 이것은 자바 문법이고 규칙이에요. (A local variable that a local class
> accesses must never have its value change partway through. So it must be declared `final`, or be
> effectively final. This is Java syntax and a rule.)

```java
Printer printer = new LocalPrinter();
// 만약 localVar의 값을 변경한다면? 다시 캡처해야 하나??
// localVar = 10; // 컴파일 오류
// paramVar = 20; // 컴파일 오류
```

- Both lines are commented out in the source because **they do not compile** — this is a
  compile-time error enforced by the Java language, not a runtime check or a style guideline.

### Terminology: Effectively Final

> 사실상 final 지역 변수는 지역 변수에 final 키워드를 사용하지는 않았지만, 값을 변경하지 않는 지역
> 변수를 뜻한다. final 키워드를 넣지 않았을 뿐이지, 실제로는 final 키워드를 넣은 것 처럼 중간에 값을
> 변경하지 않은 지역 변수이다. (An effectively final local variable is one that doesn't use the
> `final` keyword, but whose value is never changed. It's simply missing the `final` keyword —
> functionally, it behaves exactly as if `final` were there, since it's never reassigned.)

- "Effectively final" (사실상 final) means: the `final` keyword wasn't written, but the variable
  was never reassigned after its initial value anywhere in the code.
- Consequence: an effectively final variable would behave identically if `final` were explicitly
  added — so the compiler accepts it under the same rule as a genuinely `final` variable.
- Historical note from the instructor: in an older Java version, you had to write `final`
  explicitly. The effectively-final allowance is a later syntactic relaxation of that original,
  stricter rule.

---

## The Full Example: `LocalOuterV4`

```java
package nested.local;

import java.lang.reflect.Field;

public class LocalOuterV4 {

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
        // 만약 localVar의 값을 변경한다면? 다시 캡쳐해야 하나??
        // localVar = 10;
        // paramVar = 20;

        return printer;
    }

    public static void main(String[] args) {
        LocalOuterV4 localOuter = new LocalOuterV4();
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
}
```

```
value=0
localVar=1
paramVar=2
outInstanceVar=3
필드 확인
field = int nested.local.LocalOuterV4$1LocalPrinter.value
field = final int nested.local.LocalOuterV4$1LocalPrinter.val$localVar
field = final int nested.local.LocalOuterV4$1LocalPrinter.val$paramVar
field = final nested.local.LocalOuterV4 nested.local.LocalOuterV4$1LocalPrinter.this$0
```

- This is the same structure as `LocalOuterV3` from Lectures 75–76, with the attempted
  reassignments (`localVar = 10;`, `paramVar = 20;`) added as commented-out lines, since they fail
  to compile if left active.

---

## Why Can't You Change It? The Instructor's Full Reasoning

The question posed directly: changing a captured local variable *seems* like it should be fine —
so why does Java forbid it?

### The Core Problem: Keeping Two Copies in Sync

- Recall from Lecture 76: when `new LocalPrinter()` runs, Java **copies** `localVar` and
  `paramVar` into the instance at that moment.
- From that point forward there are **two separate copies** of each value: the original on the
  stack, and the captured copy inside the `LocalPrinter` instance.
- If you were allowed to change the stack-side `localVar`, Java would then have to also update the
  captured copy inside the instance to match — otherwise the two copies would simply disagree.
- And the reverse would be just as necessary: if you changed the captured copy inside the
  instance, the original stack variable would need updating too.

> 이렇게 되면 스택 영역에 존재하는 지역 변수의 값과 인스턴스에 캡처한 캡처 변수의 값이 서로 달라지는
> 문제가 발생한다. 이것을 동기화 문제라 한다. (This causes the value of the stack-area local
> variable and the value of the captured variable in the instance to diverge. This is called a
> synchronization problem.)

### Why Java Didn't Just "Fix" the Synchronization

The instructor is explicit that Java's designers *could* have chosen to make the two copies stay
in sync automatically — but lays out several concrete reasons that path is worse:

1. **Unpredictable side effects for the developer.** Code that only appears to touch a local
   variable could silently cause a value somewhere else (inside a long-lived instance) to change
   too — and vice versa. This makes debugging significantly harder, because effects show up in
   places the developer never directly touched.
2. **Real synchronization difficulty, especially with multithreading.** Keeping two copies of a
   value correctly synchronized at all times is a genuinely hard problem — the instructor notes
   this becomes especially visible once multithreading is introduced later in a developer's
   learning path, since coordinating simultaneous updates across threads is inherently difficult
   and can also hurt performance.
3. **No single clean rule emerges from allowing mutation** — every way you might try to keep stack
   and heap copies synchronized introduces its own category of new problems.

### Java's Actual Solution: Just Forbid the Change

> 자바는 캡처한 지역 변수의 값을 변하지 못하게 막아서 이런 복잡한 문제들을 근본적으로 차단한다. (Java
> prevents the value of a captured local variable from changing at all, fundamentally blocking
> these complicated problems from occurring in the first place.)

> 이거 보니까 동기화하려고 하니까 이거 문제가 너무 많네. 자, 그냥 깔끔하게 우리는 변경하지 못하게
> 막자. 왜냐면 바꾸니까 문제가 생기니까. 안 바꾸면 문제가 안 생기겠죠? (Looking at it, trying to
> synchronize causes too many problems. So let's just cleanly block changes entirely — because
> changing it is what causes the problem; if it never changes, the problem never arises.)

- If a developer genuinely needs a variable with a different, changing value, the straightforward
  fix is simply to declare a new, separate variable — there's no need to fight the compiler to
  reuse the same captured variable for a different purpose.
- This is presented as a deliberate, pragmatic design trade-off by Java's creators: rather than
  solving a genuinely hard synchronization problem, eliminate the possibility of it mattering at
  all.

---

## The Instructor's Closing Note

- If all of this feels like a lot to internalize, the instructor repeats the same fallback given at
  the start of Lecture 75:

  > 단순하게 지역 클래스가 접근하는 지역 변수의 값은 변경하면 안 된다. final이거나 사실상
  > final이어야 합니다. 이 정도로 이해하셔도 충분합니다. (Simply put: the value of a local variable
  > accessed by a local class must not be changed. It must be `final` or effectively final.
  > Understanding just this much is enough.)
- The deeper "why" was deliberately included because the instructor felt it was a gap in how this
  topic is usually taught — most explanations state the rule without explaining its motivation,
  which leaves a nagging unanswered question for learners.

## Summary

- A local class may only access local variables (including parameters) that are `final` or
  effectively final — i.e., never reassigned anywhere after their initial value.
- This is enforced as a **compile-time error**, not a convention — reassigning a captured local
  variable simply won't compile.
- The underlying reason: at instance-creation time, Java captures a **separate copy** of the
  variable into the instance (Lecture 76). Allowing the original to change afterward would require
  keeping two separate copies (stack-side and instance-side) perpetually synchronized.
- Java's designers chose not to solve that synchronization problem, because doing so would
  introduce unpredictable side effects, make debugging harder, and create real difficulty
  (especially under multithreading) with little upside.
- Instead, Java closes off the possibility entirely by forbidding any change to a captured
  variable — if a different, changing value is genuinely needed, the straightforward fix is simply
  declaring a separate variable for that purpose.
- This closes out the three-part local variable capture deep dive. Next up: **anonymous classes**
  — a special variant of the local class with no name.
