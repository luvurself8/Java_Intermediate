# Lecture 71 — Nested Classes, Inner Classes: Inner Class

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 08. 중첩 클래스, 내부 클래스1 (4/6)

Previous: [Lecture 70 - Nested Classes, Inner Classes - Static Nested Class in Practice](Lecture%2070%20-%20Nested%20Classes%2C%20Inner%20Classes%20-%20Static%20Nested%20Class%20in%20Practice.md)

Where the static nested class (Lecture 69) turned out to be "basically an unrelated class," the
(non-static) **inner class** is the opposite: it is genuinely a component of a specific outer
class instance. This lecture rebuilds the exact same experiment from Lecture 69 — minus
`static` — to show how differently it behaves.

## The Example

```java
package nested.inner;

public class InnerOuter {

    private static int outClassValue = 3;
    private int outInstanceValue = 2;

    class Inner {
        private int innerInstanceValue = 1;

        public void print() {
            // 자기 자신에 접근
            System.out.println(innerInstanceValue);

            // 외부 클래스의 인스턴스 멤버에 접근 가능, private도 접근 가능
            System.out.println(outInstanceValue);

            // 외부 클래스의 클래스 멤버에 접근 가능, private도 접근 가능
            System.out.println(outClassValue);
        }
    }
}
```

```java
package nested.inner;

public class InnerOuterMain {

    public static void main(String[] args) {
        InnerOuter outer = new InnerOuter();
        InnerOuter.Inner inner = outer.new Inner();
        inner.print();

        System.out.println("innerClass = " + inner.getClass());
    }
}
```

```
1
2
3
innerClass = class nested.inner.InnerOuter$Inner
```

- `class Inner { ... }` — note there is **no `static`** in front, which is exactly what makes this
  an inner class rather than a static nested class.
- This is deliberately the same layout as `NestedOuter`/`Nested` from Lecture 69, so the
  differences in behavior are easy to compare directly.

---

## What an Inner Class Can Access — All Three Cases Succeed

Unlike the static nested class, **every single access case here works**, including the one that
previously failed:

### 1. Its own members

```java
System.out.println(innerInstanceValue);
```

- Same as before — a class can always access its own fields.

### 2. The outer class's instance members — now allowed!

```java
System.out.println(outInstanceValue);
```

- This is the big difference from Lecture 69: because `Inner` is **not** `static`, it belongs to
  an instance of `InnerOuter`, not just to the class area.
- It can reach `outInstanceValue` — an instance field — **even though it's `private`**.
- Why `private` doesn't block it: `private` means "accessible only within the same class body,"
  and the inner class is physically declared within `InnerOuter`'s body, so it's within that same
  boundary.

### 3. The outer class's static (class) members — still allowed

```java
System.out.println(outClassValue);
```

- Static members were always reachable the same way for the static nested class too; nothing
  changes here.

| | Access own members | Access outer instance members | Access outer static members |
|---|---|---|---|
| Static nested class (Lecture 69) | ✅ | ❌ | ✅ (even `private`) |
| Inner class (this lecture) | ✅ | ✅ (even `private`) | ✅ (even `private`) |

> 내부 클래스는 바깥 클래스, 자신의 멤버에 당연히 접근할 수 있고, 바깥 클래스의 instance 멤버에 접근할
> 수 있어요. 그리고 바깥 클래스의 클래스 멤버에도 접근할 수가 있습니다. (An inner class can naturally
> access its own members, the outer class's instance members, and the outer class's class members
> too — all of it.)

---

## Creating an Inner Class: A Different, Unusual Syntax

This is the part the instructor flags as unexpected the first time you see it.

```java
InnerOuter outer = new InnerOuter();
InnerOuter.Inner inner = outer.new Inner();
```

- For the static nested class, creation was `new NestedOuter.Nested()` — no outer *instance* was
  ever required, because the static nested class has no relationship to any instance.
- For an inner class, the syntax is instead **`outerInstance.new InnerClass()`**.
- **Why the syntax has to look like this**: `Inner` can access `outInstanceValue`, an instance
  member — which means it *must* know *which* `InnerOuter` instance it belongs to. There could be
  many `InnerOuter` instances in the program (`outer`, a second one, a third one...), so you have
  to specify exactly which one this particular `Inner` is tied to.
  - `outer.new Inner()` reads as: "create this `Inner` instance *inside* `outer`'s instance."
  - If there were multiple outer instances (`outer2`, `outer3`), writing `outer2.new Inner()`
    would belong to `outer2`, and `outer3.new Inner()` would belong to `outer3` — you must choose.
- A direct consequence: **you cannot create an inner-class instance without first creating an
  outer-class instance.** There is no `new InnerOuter.Inner()` form — the outer instance must
  already exist, because the inner instance is conceptually created inside it.
- The `getClass()` output, `InnerOuter$Inner`, uses the same `OuterClass$NestedClass` naming
  scheme introduced for static nested classes in Lecture 69 — this naming applies to every kind of
  nested class.

---

## How This Actually Works: Conceptual Model vs. Real Implementation

The instructor is careful to separate "the mental model that explains the behavior" from "what
the JVM literally does under the hood."

### Conceptually

- Picture the outer instance (say `x001`) and, nested *inside* it, the inner-class instance
  (`x002`) — as if the inner object were literally created inside the outer object's memory.
- Because the inner instance is pictured as living inside the outer instance, it naturally "knows"
  how to reach the outer instance's members, like `outInstanceValue`.
- This mental picture is sufficient to reason about inner classes day-to-day — you don't need the
  implementation detail below to use them correctly.

### In Reality

- The inner instance is **not** physically nested inside the outer instance's memory — the two
  objects are actually created completely separately.
- What really happens: when the inner instance is created via `outer.new Inner()`, it **stores a
  reference back to `outer`** internally.
- When `Inner.print()` needs `outInstanceValue` and doesn't find it among its own fields, it
  follows that stored reference back to the outer instance and reads the field from there.

> 실제로는 내부 인스턴스는 바깥 인스턴스의 참조를 보관한다. 이 참조를 통해 바깥 인스턴스의 멤버에 접근할
> 수 있다. (In reality, the inner instance holds a reference to the outer instance, and reaches the
> outer instance's members through that reference.)

- This is exactly why `outer.new Inner()` is required: that call is what lets the new `Inner`
  instance capture and store the correct outer-instance reference at construction time.

---

## Nested vs. Inner — Revisited With a Concrete Instance Picture

This lecture closes by tying the behavior just observed back to the conceptual distinction from
Lecture 68:

- **Nested (중첩)**: something inside another thing but unrelated to it — "in my insides, but not
  part of me."
- **Inner (내부)**: something inside another thing that is a genuine component of it — "in my
  insides, and part of what makes me up."

Applied now with working code behind it:

- A **static nested class** really is just another class placed inside — the only special power
  it gets is access to the outer class's `private` static members; otherwise the two classes have
  zero relationship, and neither needs the other to exist.
- An **inner class** is a genuine structural component of a specific outer instance: it cannot be
  created without that instance, it holds a reference back to it, and it can freely read the outer
  instance's members (even `private` ones) through that reference.

## Summary

- An inner class is written without `static` — `class Inner { ... }` inside the outer class body.
- It can access its own members, the outer class's instance members (even `private`), and the
  outer class's static members (even `private`) — unlike a static nested class, nothing is
  off-limits.
- `private` doesn't block any of this because the inner class is physically declared within the
  outer class's body, so it shares that same access boundary.
- Creating one requires an existing outer instance first: `outerInstance.new InnerClass()` — there
  is no way to instantiate an inner class standalone.
- This requirement exists because the inner instance must know exactly *which* outer instance it
  belongs to, in order to reach that instance's members.
- Conceptually, picture the inner instance as created *inside* the outer instance; in reality, the
  inner instance just stores a reference back to the outer instance and uses it to look up members
  it doesn't have itself.
- `getClass()` reports the type as `OuterClass$InnerClass`, the same `$`-joined naming used for
  static nested classes.
- Bottom line distinction: a static nested class has no relationship to any outer instance; an
  inner class is a true component of one specific outer instance and cannot exist without it.
