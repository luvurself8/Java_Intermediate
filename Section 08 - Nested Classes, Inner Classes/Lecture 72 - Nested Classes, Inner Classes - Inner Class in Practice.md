# Lecture 72 — Nested Classes, Inner Classes: Inner Class in Practice

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 08. 중첩 클래스, 내부 클래스1 (5/6)

Previous: [Lecture 71 - Nested Classes, Inner Classes - Inner Class](Lecture%2071%20-%20Nested%20Classes%2C%20Inner%20Classes%20-%20Inner%20Class.md)

This is the lecture the instructor explicitly promised in Lecture 68: a worked example that makes
"inner classes give you encapsulation" click. A `Car`/`Engine` example is refactored twice — once
to show the problem with keeping them as separate top-level classes, and once to show exactly how
much simpler an inner class makes it.

## Before: `Engine` as a Separate Top-Level Class (`ex1`)

```java
package nested.inner.ex1;

//Car에서만 사용
public class Engine {

    private Car car;

    public Engine(Car car) {
        this.car = car;
    }

    public void start() {
        System.out.println("충전 레벨 확인: " + car.getChargeLevel());
        System.out.println(car.getModel() + "의 엔진을 구동합니다.");
    }
}
```

```java
package nested.inner.ex1;

public class Car {
    private String model;
    private int chargeLevel;
    private Engine engine;

    public Car(String model, int chargeLevel) {
        this.model = model;
        this.chargeLevel = chargeLevel;
        this.engine = new Engine(this);
    }

    //Engine에서만 사용하는 메서드
    public String getModel() {
        return model;
    }

    //Engine에서만 사용하는 메서드
    public int getChargeLevel() {
        return chargeLevel;
    }

    public void start() {
        engine.start();
        System.out.println(model + " 시작 완료");
    }
}
```

```java
package nested.inner.ex1;

public class CarMain {
    public static void main(String[] args) {
        Car myCar = new Car("Model Y", 100);
        myCar.start();
    }
}
```

```
충전 레벨 확인: 100
Model Y의 엔진을 구동합니다.
Model Y 시작 완료
```

- The domain idea: `Engine` needs to know the car's charge level and model name before it can
  "start" — think of this as an electric car checking its battery before ignition.
- Since `Engine` is a completely separate top-level class, it has no built-in way to reach `Car`'s
  fields. It has to be handed a reference to its owning `Car` explicitly in the constructor
  (`this.car = car`, where `this` is the `Car` being constructed), and then call back through that
  reference: `car.getChargeLevel()`, `car.getModel()`.
- To make that possible, `Car` is forced to expose `getModel()` and `getChargeLevel()` as `public`
  methods — **even though, by the comment directly on them, they exist for one reason only: so
  `Engine` can call them.** No other code in the program is meant to use these getters at all.

### The Problem

- `Car` class now has to provide methods whose entire purpose is accommodating `Engine`.
- Those methods leak `Car`'s internal state (`model`, `chargeLevel`) out to the public API of
  `Car`, where any other code could also call them — information that `Car` otherwise has no
  reason to expose.
- This directly weakens encapsulation: `Car`'s "public surface" now contains implementation
  details that only exist to serve one tightly-coupled helper class.

---

## After: `Engine` as an Inner Class (`ex2`)

```java
package nested.inner.ex2;

public class Car {
    private String model;
    private int chargeLevel;
    private Engine engine;

    public Car(String model, int chargeLevel) {
        this.model = model;
        this.chargeLevel = chargeLevel;
        this.engine = new Engine();
    }

    public void start() {
        engine.start();
        System.out.println(model + " 시작 완료");
    }

    private class Engine {
        public void start() {
            System.out.println("충전 레벨 확인: " + chargeLevel);
            System.out.println(model + "의 엔진을 구동합니다.");
        }
    }
}
```

```java
package nested.inner.ex2;

public class CarMain {

    public static void main(String[] args) {
        Car myCar = new Car("Model Y", 100);
        myCar.start();
    }
}
```

```
충전 레벨 확인: 100
Model Y의 엔진을 구동합니다.
Model Y 시작 완료
```

Walking through what changed, in the order the instructor builds it up in the recording:

1. `Engine` moves inside `Car`'s body and is marked `private class Engine` — private because,
   just like the static nested class in Lecture 70, it's purely an internal implementation detail.
2. The constructor parameter is gone entirely: `Engine(Car car)` becomes a no-arg `Engine()`, and
   `Car`'s constructor just calls `new Engine()`.
   - This works because an inner class, as established in Lecture 71, automatically holds a
     reference back to the outer instance that created it — there's no need to pass that
     reference manually anymore.
3. The `private Car car;` field inside `Engine` is removed — unnecessary now.
4. **`getModel()` and `getChargeLevel()` are deleted from `Car` entirely.**
5. Inside `Engine.start()`, `car.getChargeLevel()` becomes just `chargeLevel`, and
   `car.getModel()` becomes just `model` — direct field access, no getter, no `car.` prefix at
   all, because `Engine` is an inner class and already has implicit access to `Car`'s instance
   fields (`private` included).

> 이 카에 대한 정보가 없어도 나는 이 안에서 여러분, 여기 다 접근할 수가 있어요. 왜냐? 내부 클래스는
> 바깥 클래스의 인스턴스에 접근할 수가 있죠. (I don't even need a reference to the `Car` — I can
> access everything from in here, because an inner class can access the outer class's instance.)

---

## Inner Class Creation, Reconfirmed From This Example

- Inside `Car`'s own body, `new Engine()` is enough — no `Car.` prefix needed, matching the
  "from inside the owning class, drop the outer-class name" rule from Lecture 70.
- The instance created by `new Engine()` **automatically references the specific `Car` instance
  that created it** — this is the mechanism, stated plainly here, that lets `Engine.start()` reach
  `chargeLevel` and `model` with no explicit wiring at all.

> `new Engine()`로 생성된 Engine 인스턴스는 자신을 생성한 바깥의 Car 인스턴스를 자동으로
> 참조한다. (The `Engine` instance created by `new Engine()` automatically references the `Car`
> instance that created it.)

---

## Before vs. After, Side by Side

| | Before (`ex1`, separate classes) | After (`ex2`, inner class) |
|---|---|---|
| `Engine` constructor | `Engine(Car car)` — must receive `Car` explicitly | `Engine()` — no arguments needed |
| How `Engine` reaches `Car`'s data | `car.getChargeLevel()`, `car.getModel()` via a stored `Car` reference | Direct field access: `chargeLevel`, `model` |
| `Car`'s public API | Must expose `getModel()`, `getChargeLevel()` just for `Engine`'s sake | Neither getter needs to exist |
| Encapsulation | Weaker — internal state leaked for one caller's convenience | Stronger — internal state stays fully private |

---

## Why This Matters: Revisiting "When to Nest" and "Why to Nest"

This lecture closes by looping back to the reasoning first introduced in Lecture 68, now backed
by a concrete before/after:

> 중첩 클래스는 특정 클래스가 다른 하나의 클래스 안에서만 사용되거나, 둘이 아주 긴밀하게 연결되어 있는
> 특별한 경우에만 사용해야 한다. 외부 여러곳에서 특정 클래스를 사용한다면 중첩 클래스로 사용하면 안된다.
> (A nested class should only be used when a class is used exclusively inside one other class, or
> the two are extremely tightly coupled. If used from many places externally, don't nest it.)

- **Logical grouping**: `Engine` is only ever relevant to `Car`, so keeping it inside `Car`'s body
  means opening the package doesn't expose a standalone `Engine` class that looks reusable but
  actually isn't.
- **Encapsulation** (the payoff this specific example demonstrates): because the inner class can
  reach the outer class's `private` members directly, the two classes can be tightly coupled
  *without* the outer class needing to expose any `public` method purely to accommodate the inner
  one. `getModel()` and `getChargeLevel()` simply cease to exist once the inner-class refactor is
  done — `Car`'s public surface shrinks to only what's actually needed externally (`start()`), and
  all the internal wiring stays fully hidden.

---

## Summary

- Keeping `Engine` as a separate top-level class forced `Car` to add `public` getters
  (`getModel()`, `getChargeLevel()`) whose sole purpose was feeding data to `Engine` — leaking
  internal state for no real external benefit.
- Making `Engine` a `private` inner class of `Car` eliminates the need for a stored `Car`
  reference, a constructor parameter, and both getters — `Engine` just reaches `chargeLevel` and
  `model` directly as if they were its own fields.
- This works because an inner-class instance automatically holds a reference to the specific outer
  instance that created it, as established in Lecture 71 — `new Engine()` from inside `Car`
  captures that link for free.
- The net result is a smaller, more honest public API for `Car`: only `start()` is exposed, and
  all of the engine-related wiring is fully hidden — this is the concrete form of the
  "encapsulation" benefit promised back in Lecture 68.
- General rule reaffirmed: nest a class only when it's used exclusively by (or extremely tightly
  coupled with) one outer class; if other code needs it too, keep it as a top-level class instead.
