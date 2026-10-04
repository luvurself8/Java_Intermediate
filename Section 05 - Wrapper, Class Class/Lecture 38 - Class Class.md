# Lecture 38 — Class Class

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (6/11)

Previous: [Lecture 37 - Key Methods and Performance](./Lecture%2037%20-%20Wrapper%20Class%20-%20Key%20Methods%20and%20Performance.md)

This lecture leaves wrapper classes behind and introduces a different topic that shares the
section: Java's `Class` class — the type used to represent **metadata about a class itself**.

---

## 1. What is the `Class` class?

- `Class` is used to handle a class's own information — its metadata.
- Through a `Class` object, a developer can **look up and manipulate** the properties and methods
  of a class, while the application is running.

### 1.1 Main capabilities

- **Type information** — name, superclass, interfaces, access modifiers, and similar facts about
  a class.
- **Reflection** — look up a class's methods, fields, and constructors; use that information to
  create instances or invoke methods.
- **Dynamic loading** — `Class.forName(...)` loads a class dynamically (by name, as a string), and
  `newInstance()`-style calls can create a new instance from it.
- **Annotation handling** — read and process `@`-style annotations attached to a class.

### 1.2 A concrete example

- `String.class` is the `Class` object representing the `String` class.
- Through it, you can look up or manipulate `String`'s metadata.

---

## 2. Naming: `class` vs `clazz`

Before writing any code, there's an immediate naming problem:

- `class` is a **reserved word** in Java — it can't be used as a variable name or package name.
- Convention: Java developers use **`clazz`** instead.
  - `clazz` sounds similar to "class", so the intent reads clearly.
  - This is why the example class below is `ClassMetaMain` in package `lang.clazz` (not
    `lang.class`, which wouldn't even compile).

---

## 3. Reading class metadata: `ClassMetaMain`

```java
package lang.clazz;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ClassMetaMain {
    public static void main(String[] args) throws Exception {
        //Class 조회
        Class clazz = String.class; // 1. 클래스에서 조회
        //Class clazz = new String().getClass(); // 2. 인스턴스에서 조회
        //Class clazz = Class.forName("java.lang.String"); // 3. 문자열로 조회

        // 모든 필드 출력
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            System.out.println("field = " + field.getType() + " " + field.getName());
        }

        // 모든 메서드 출력
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("method = " + method);
        }

        // 상위 클래스 정보 출력
        System.out.println("Superclass: " + clazz.getSuperclass().getName());

        // 인터페이스 정보 출력
        Class[] interfaces = clazz.getInterfaces();
        for (Class i : interfaces) {
            System.out.println("Interface: " + i.getName());
        }
    }
}
```

**Output (abridged — `String` has many fields/methods):**

```
field = class [B value
...
method = public boolean java.lang.String.equals(java.lang.Object)
method = public int java.lang.String.length()
...
Superclass: java.lang.Object
Interface: java.io.Serializable
Interface: java.lang.Comparable
...
```

### 3.1 `throws Exception` on `main`

- This code won't compile without `throws Exception` added next to `main`.
- Reason: this is a **checked exception** situation — the course covers checked exceptions
  properly later, in the exception-handling section.
- For now: just know it needs to be there, or you'll get a compile error. (IntelliJ can add this
  automatically via Alt-Enter → "Add Exception to Method Signature".)

### 3.2 Three ways to obtain a `Class` object

```java
Class clazz = String.class;                        // 1. From the class itself
Class clazz = new String().getClass();              // 2. From an instance
Class clazz = Class.forName("java.lang.String");    // 3. From a fully-qualified string name
```

- **From the class directly** — `String.class`. Simplest option when you already know the type at
  compile time.
- **From an instance** — `someInstance.getClass()`. Useful when you have an object in hand and
  want to know its runtime type.
- **From a string** — `Class.forName("java.lang.String")`. The string must be the **fully
  qualified** class name (package + class name). This is the one that enables truly dynamic
  behavior — e.g. the class name could come from user input (a `Scanner`, a config file, etc.),
  and the program decides which class to work with at runtime.

All three return equivalent `Class` metadata for the same underlying type.

### 3.3 Inspecting fields: `getDeclaredFields()`

- Returns every field declared on the class, as `Field[]`.
- Running this on `String` reveals, among others, a field whose name is `value` and whose type is
  a `byte[]` — this connects back to an earlier part of the course: `String` stores its characters
  internally as a byte array.
- `field.getType()` and `field.getName()` give the field's type and name respectively.

### 3.4 Inspecting methods: `getDeclaredMethods()`

- Returns every method declared on the class, as `Method[]`.
- `String` has a large number of methods — printing the array surfaces familiar names like
  `valueOf` and `indexOf` from earlier lectures.

### 3.5 Inspecting the superclass: `getSuperclass()`

- `clazz.getSuperclass().getName()` → `java.lang.Object`.
- `String` doesn't declare `extends` anything explicitly.
- Reminder: in Java, if a class has no explicit `extends` clause, it implicitly extends `Object`.

### 3.6 Inspecting interfaces: `getInterfaces()`

- `clazz.getInterfaces()` lists the interfaces the class implements.
- For `String`, this includes `java.io.Serializable`, `java.lang.Comparable`, and others.

### 3.7 Recap of `Class`'s key methods used here

| Method | What it returns |
|---|---|
| `getDeclaredFields()`   | all fields declared on the class |
| `getDeclaredMethods()`  | all methods declared on the class |
| `getSuperclass()`       | the parent class |
| `getInterfaces()`       | the interfaces implemented |

(`Class` has many more methods than this — these are just the handful demonstrated here.)

---

## 4. Creating an instance from a `Class` object

A `Class` object carries *all* of a class's information — enough to create new instances, call
methods, and change field values dynamically. This section demonstrates the simplest case: just
creating an instance.

### 4.1 The target class: `Hello`

```java
package lang.clazz;

public class Hello {
    public String hello() {
        return "hello!";
    }
}
```

### 4.2 Creating a `Hello` dynamically: `ClassCreateMain`

```java
package lang.clazz;

public class ClassCreateMain {
    public static void main(String[] args) throws Exception {
        //Class helloClass = Hello.class;
        Class helloClass = Class.forName("lang.clazz.Hello");

        Hello hello = (Hello) helloClass.getDeclaredConstructor().newInstance();
        String result = hello.hello();
        System.out.println("result = " + result);
    }
}
```

**Output:**

```
result = hello!
```

Walking through it:

- `Class.forName("lang.clazz.Hello")` — loads the `Hello` class dynamically, by its fully
  qualified name (package included). The commented-out line above it, `Hello.class`, shows the
  simpler compile-time alternative — both produce equivalent `Class` metadata for `Hello` here.
- `helloClass.getDeclaredConstructor()` — selects a constructor from the class's metadata (the
  no-argument constructor, by default, since `Hello` doesn't declare any constructor explicitly).
- `.newInstance()` — creates a new instance using that selected constructor. This is conceptually
  the same as calling `new Hello()`, just driven through metadata instead of a direct `new` call.
- The result of `newInstance()` comes back typed as plain `Object`, so it has to be **cast** to
  `Hello` before `hello.hello()` can be called on it.
- Once cast, `hello.hello()` works exactly like calling it on an object created the normal way —
  it returns `"hello!"`.

### 4.3 Why this matters: dynamic object creation

- Because `Class.forName(...)` takes a **string**, the class name doesn't have to be hard-coded.
- It could come from user input, a config file, or anywhere else at runtime — and the program can
  then create an instance of *whichever* class that string names, entirely dynamically.
- This is the core idea behind **dynamic object creation**: deciding which class to instantiate at
  runtime, rather than fixing it at compile time.

---

## 5. Reflection — the umbrella term for all of this

- Using a class's metadata (via `Class`) to look up its methods, fields, and constructors — and
  then using that information to create instances or invoke methods — is called **reflection**.
- Reflection can also read annotation information to drive special behavior.
- Modern frameworks make heavy use of reflection internally.

### 5.1 How deep to go right now

The instructor is explicit that reflection itself is **not** something to dive deep into at this
stage:

> "지금은 클래스가 뭔지 그리고 대략 어떤 기능을 하는지 정도만 알아두시면 충분해요."
> (For now, it's enough to know what `Class` is and roughly what it can do.)

- Reflection is genuinely useful when building a framework, or when trying to understand Java at
  a deeper level later on.
- Day-to-day application development rarely requires reaching for reflection directly — it's
  entirely possible to build real applications without ever touching it.
- There are more foundational topics in this course still worth prioritizing over reflection.
- The instructor plans to cover reflection properly in a separate, later discussion — "거의 다
  배우고 나서" (after having learned almost everything else).

**Takeaway for now:** just understand that `Class` exists, that it holds a class's metadata, and
that this metadata can be used to inspect/create/manipulate things dynamically — that's reflection,
at a glance. No need to go further than that yet.

---

## Summary

- `Class` represents a class's own metadata — its name, fields, methods, superclass, interfaces,
  and annotations.
- `class` is a reserved word, so Java developers conventionally name variables `clazz` instead.
- A `Class` object can be obtained three ways: from the type itself (`String.class`), from an
  instance (`instance.getClass()`), or from a fully-qualified string name (`Class.forName(...)`).
- Key inspection methods: `getDeclaredFields()`, `getDeclaredMethods()`, `getSuperclass()`,
  `getInterfaces()`.
- `Class.forName(...)` requires `throws Exception` on the enclosing method, due to a checked
  exception (covered properly in the later exception-handling section).
- A `Class` object can also be used to **create a new instance dynamically**:
  `helloClass.getDeclaredConstructor().newInstance()`, then cast the result to the real type.
- This whole category of "use metadata to inspect/construct/invoke dynamically" is called
  **reflection** — foundational for frameworks, but intentionally not a deep-dive topic yet at
  this point in the course.
- Next lecture moves to the **System class**.
