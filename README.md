### Types of Concurrency Bugs

There are various ways to categorise concurrency bugs. In this article, we classify them into four different categories: _atomicity violations_, _order violations_, _data races_, and _deadlocks_.

1. Atomicity Violations
- Atomicity violations occur when atomicity assumptions made about the execution 
- Aomicity refers to the property of a multithreaded program segment that allows the segment to appear as if it occurred instantaneously to the rest of the system.
<!-- [[1]](#1) -->

2. Order Violations
- Order violations occur when memory accesses from two or more threads happen in an order inconsistent with the order desired by programmers.
- They are caused by incorrect assumptions or absence of correct assumptions made on the order of execution of concurrent threads.

3. Data Races
- Data races occur when two or more threads access the same memory location while at least one of them performs a write operation.
- Data races and race conditions are not the same. There are data races that are not race conditions as well as race conditions that are not data races.
- At the same time, not all data races are buggy. There are benign data races.
- Often, it requires a deeper understanding of the software to decide whether data races identified are true bugs or not.

4. Deadlocks
- Deadlocks are defined as a condition in a system where a process cannot proceed because it needs to obtain a resource held by another process but the process itself is holding a resource that the other process needs.


#### Note: These examples are collected in an attempt to find interdependencies between different types of concurrency bugs.


## Buggy programs containing both data races, atomicity violations, and/or order violations

1. Example #1 : data races + atomicity violations

- Root case of the bugs can be diagnosed in relation to data races or atomicity violations.
+ In the context of data races, there may be threads writing to the list `a` via `a.add(x);` while others simultaneously read it via `!a.contains(x)`.
+ In the context of atomicity violations, the fact that checking `if (!a.contains(x))` and modifying `a.add(x)` were not an atomic operation caused the bug.
+ This bug can be fixed by adding a synchronisation primitive and making the operations atomic, as shown in the comments.

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ConcurrencyTest {
    static final List a = Collections.synchronizedList(new ArrayList());

    public static void main(String[] args) {
        Thread t = new Thread(() -> addIfAbsent(17));
        t.start();
        addIfAbsent(17);
        t.join();
        System.out.println(a);
    }

    private static void addIfAbsent(int x) {
        // synrhonized(a) {
        if (!a.contains(x)) {
            a.add(x);
        }
        // }
    }
}
// code source : IntelliJ IDEA Debugging Tutorial
```

2. Example #2 : data races + atomicity violations


```java


```


3. Example #3 : data races + order violations






1.1. Detection with existing tools

- Running RacerD results in the following:

```console
$ infer --racerd-only .java

```

- Running Chord


1.2. Fixing with existing tools

- Running Hippodrome results in the following outcome:
```console
$ 
```

- Running HFix
+ Not feasible as HFix was unavailable.




<!-- ### References
<a id="1">[1]</a> 
 -->
