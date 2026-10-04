# Lecture 39 — System Class

Course: [김영한의 실전 자바 중급 1](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-1/dashboard?cid=333308) — Section 5. 래퍼, Class 클래스 (7/11)

Previous: [Lecture 38 - Class Class](./Lecture%2038%20-%20Class%20Class.md)

`System` provides a grab-bag of basic utilities related to the underlying system Java is running
on — time, environment, properties, fast array copying, and program termination.

---

## 1. The example: `SystemMain`

```java
package lang.system;

import java.util.Arrays;
import java.util.Map;

public class SystemMain {

    public static void main(String[] args) {
        // 현재 시간(밀리초)를 가져온다.
        long currentTimeMillis = System.currentTimeMillis();
        System.out.println("currentTimeMillis = " + currentTimeMillis);

        // 현재 시간(나노초)를 가져온다.
        long currentTimeNano = System.nanoTime();
        System.out.println("currentTimeNano = " + currentTimeNano);

        // 환경 변수를 읽는다.
        System.out.println("getenv= " + System.getenv());

        // 시스템 속성을 읽는다.
        System.out.println("properties = " + System.getProperties());
        System.out.println("Java version: " + System.getProperty("java.version"));

        // 배열을 고속으로 복사한다.
        char[] originalArray = {'h', 'e', 'l', 'l', 'o'};
        char[] copiedArray = new char[5];
        System.arraycopy(originalArray, 0, copiedArray, 0, originalArray.length);

        // 배열 출력
        System.out.println("copiedArray = " + copiedArray);
        System.out.println("Arrays.toString = " + Arrays.toString(copiedArray));

        // 프로그램 종료
        System.exit(0);
    }
}
```

**Output (values are environment/machine-specific):**

```
currentTimeMillis: 1703570732276
currentTimeNano: 2863721061045 83

getenv = {IDE_INITIAL_DIRECTORY=/, COMMAND_MODE=unix2003,
LC_CTYPE=ko_KR.UTF-8, SHELL=/bin/zsh, HOME=/Users/yh, PATH=/opt/homebrew/bin:/usr/local/bin: ...}

properties = {java.specification.version=21, java.version=21.0.1,
sun.jnu.encoding=UTF-8, os.name=Mac OS X, file.encoding=UTF-8 ...}

Java version: 21.0.1

copiedArray = [C@77459877
Arrays.toString = [h, e, l, l, o]
```

---

## 2. Measuring time

- `System.currentTimeMillis()` — current time in **milliseconds**.
- `System.nanoTime()` — current time in **nanoseconds**.
- Both are used the same way shown in earlier wrapper-class benchmarking code (Lecture 37): read a
  start time, do work, read an end time, subtract.

> Caveat worth remembering: `nanoTime()` isn't necessarily "wall-clock" time — it's measured from
> some arbitrary reference point defined by the JVM or OS. It's reliable for measuring an
> **elapsed duration** between two calls, not for reading an actual calendar time.

---

## 3. Reading environment variables: `System.getenv()`

- Returns the OS-level **environment variables** the system has configured — e.g. things like
  `JAVA_HOME`, `PATH`, and so on, that get set up when installing Java or configuring a shell.
- The return type is a `Map` (hence the `import java.util.Map` at the top) — a proper deep dive
  into what a `Map` is comes later in the course (collections), so for now it's enough to just
  print it directly and see the raw output.
- Printing it directly dumps the whole environment variable set at once, e.g. `SHELL`, `HOME`,
  `PATH`, etc.

---

## 4. Reading system properties: `System.getProperties()` / `System.getProperty(key)`

- `System.getProperties()` — the full set of properties **Java itself** uses: things like the
  current encoding, the Java version, the OS name, and other Java-specific configuration.
- Distinction worth keeping straight:
  - `System.getenv()` → OS-level environment variables.
  - `System.getProperties()` → Java-level system properties.
- `System.getProperty("java.version")` — reads a single specific property by key. For example,
  requesting `"java.version"` returns the running JVM's version (`21.0.1` in the instructor's
  environment) — this exact key (`java.version`) is visible inside the full properties dump too.

---

## 5. Fast array copying: `System.arraycopy(...)`

### 5.1 The problem it solves

- Given a source array (e.g. `{'h','e','l','l','o'}`) and a need to copy its contents into another
  array, the "obvious" approach is a loop: copy index 0, then 1, then 2, ... one at a time.
- A manual loop like that is comparatively **slow** — every single element gets copied one step at
  a time, in Java code.

### 5.2 `System.arraycopy(src, srcPos, dest, destPos, length)`

```java
System.arraycopy(originalArray, 0, copiedArray, 0, originalArray.length);
```

Arguments, in order:

1. source array (`originalArray`)
2. starting index in the source (`0`)
3. destination array (`copiedArray`)
4. starting index in the destination (`0`)
5. number of elements to copy (`originalArray.length`)

### 5.3 Why it's faster than a loop

- `System.arraycopy` doesn't copy element-by-element in a Java-level loop at all.
- Instead, Java hands the whole copy request down to the **operating system / hardware level** —
  effectively "here's an array, copy it over there" — and the OS copies the entire block of memory
  in one shot, rather than looping value by value.
- Because the whole block gets read and copied at once (instead of one Java-level loop iteration
  per element), this is **meaningfully faster** than a manual loop.
- Exact speedup varies by system and array size, but the instructor estimates **roughly 2x–5x
  faster**, with "at least 2x" as a safe baseline expectation.

### 5.4 Printing the result

- Printing a `char[]` array object directly (`copiedArray`) does **not** print its contents — it
  prints something like `[C@77459877`: `[C` meaning "array of `char`", followed by a reference
  value. This is standard `Object.toString()` behavior for arrays — arrays don't override
  `toString()` to show their contents.
- To actually see the array's contents, use the utility method **`Arrays.toString(copiedArray)`**,
  which formats it nicely as `[h, e, l, l, o]`.

---

## 6. Standard input/output/error streams

This lecture connects back to streams seen earlier in the course:

- `System.in` — standard input (reading values from the console).
- `System.out` — standard output (the `println` calls used everywhere).
- `System.err` — standard error output — exists, but not used very often in practice.

These three represent the **standard input, output, and error streams** respectively.

---

## 7. Ending a program: `System.exit(status)`

```java
System.exit(0);
```

- Immediately terminates the running program, right at the point it's called.
- Demonstrated by placing a `println("hello")` *after* `System.exit(0)` in a quick test — it never
  prints, because the program has already ended by that point.
- Passes a **status code** back to the OS describing how the program ended:
  - `0` → the program ended **normally**.
  - any non-zero value → the program ended **abnormally** / due to some problem.

### 7.1 Why `System.exit()` is generally discouraged

This is the strongest warning in the lecture:

> "이거는 진짜 가급적이면 사용하시면 안 돼요." (This really should generally not be used.)

- A program usually needs to do cleanup before it ends — closing resources, finishing up pending
  work, etc.
- `System.exit(...)` cuts the program off **immediately**, wherever it's called — any logic that
  was supposed to run afterward simply never runs.
- This is especially dangerous in a **web application**: a server process is meant to stay running
  and keep handling requests. Calling `System.exit(...)` there could kill the entire server process
  mid-request, abruptly, for every user currently being served — not just the current request.
- The preferred approach: let the program terminate "in order," naturally, after everything that
  needs to happen has happened — rather than forcing an abrupt stop from inside arbitrary code.

### 7.2 When it's acceptable

- There are legitimate cases for it — e.g. a standalone script/tool where you, the developer, have
  deliberately handled all cleanup yourself and fully understand the termination behavior.
- In that kind of controlled, well-understood scenario, using `System.exit(...)` is fine.
- As a default/general habit for application code (especially anything server-like), avoid it.

---

## Summary

- `System` bundles together a handful of unrelated but commonly-needed system-level utilities.
- **Time**: `System.currentTimeMillis()` (ms) and `System.nanoTime()` (ns) for measuring elapsed
  time — not for reading calendar time with `nanoTime()`.
- **Environment variables**: `System.getenv()` reads OS-level environment variables (e.g. `PATH`).
- **System properties**: `System.getProperties()` / `System.getProperty(key)` read Java-level
  configuration (e.g. `java.version`).
- **Fast array copy**: `System.arraycopy(src, srcPos, dest, destPos, length)` delegates to the
  OS/hardware for a block-level copy — roughly 2x–5x faster than a manual loop. Remember
  `Arrays.toString(...)` to actually print an array's contents.
- **Streams**: `System.in` / `System.out` / `System.err` are the standard input/output/error
  streams.
- **Program termination**: `System.exit(status)` ends the program immediately (`0` = normal exit,
  non-zero = abnormal). Generally discouraged — especially in server/web applications — because it
  skips any cleanup and can terminate the process mid-work. Acceptable only when termination is
  fully, deliberately controlled.
- Next lecture covers the remaining utility classes in this section: **`Math` and `Random`**.
