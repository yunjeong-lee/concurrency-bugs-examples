## Types of Concurrency Bugs

There are various ways to categorise concurrency bugs. We classify them into four different categories: _atomicity violations_, _order violations_, _data races_, and _deadlocks_.

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


## Narrative

1. Data races are often said to be associated with, if not the cause of, other types of concurrency bugs such as atomicity violations or order violations.

2. How exactly are they interconnected? Is there any way to measure this interdependencies?

  2-a. By the way, I couldn't find any work covering their dependencies or correlations, other than a qualitative, general statement. Let me see if it is any possible, though.
  
  2-b. First, what would be examples of buggy programs that contain more than one types of concurrency bugs? You can find some of these examples below (Examples #1-5).
  
  2-c. Findings from the examples:
    - Races (buggy, not the benign ones) and atomicity violations often overlap. There is even some paper on benchmark suite which classifies them as one type (call it "Races and atomicity violations").
    - 
  
3. Moreover, if these bugs are interconnected, how can we addresss--i.e., fix--multiple (sub-)types of (race condition) concurrency bugs at the same time? (Here, race condition bugs refer to atomicity violations, order violations, and data races.)

  3-a. At the end of the day, we want to fix all the bugs, or as much as possible. It is possible to have programs that are race-free but still contain other bugs such as atomicity violations or fixing data races results in other bugs such as deadlocks. For example, you can have the following race-free program with an atomicity violation (where the read and update of `counter` should have happened inside the same critical section):
  ```java
  int counter; // shared variable
               // protected by lock L
  
  void increment() {
    int temp;
    
    lock(L);
    temp = counter;
    unlock(L);
    
    temp++;
    
    lock(L);
    counter = temp;
    unlock(L);
  }
  // source: Atom-Aid
  ```

  3-b. How do existing tools perform with respect to the abovementioned examples?
    3-b-i. With respect to detection using RacerD
    
    3-b-ii. With respect to repair using Hippodrome
    
    3-b-iii. Observations made on detection / repair using existing tools

  3-c. Atomicity violations and order violations are often caused by incorrect assumptions or absense of correct assumptions made on the atomicity of operations or order of execution of concurrent threads respectively. Can we provide these assumptions as constraints and have repair tool 





## Buggy programs containing both data races, atomicity violations, and/or order violations

#### Note: Examples below are collected in an attempt to find interdependencies between different types of concurrency bugs.

1. Example #1 : data races + atomicity violations

- Root case of the bugs can be diagnosed in relation to data races or atomicity violations.
  * In the context of data races, there may be threads writing to the list `a` via `a.add(x);` while others simultaneously read it via `!a.contains(x)`.
  * In the context of atomicity violations, the fact that checking `if (!a.contains(x))` and modifying `a.add(x)` were not an atomic operation caused the bug.
  * This bug can be fixed by adding a synchronisation primitive and making the operations atomic, as shown in the comments.

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

2. Example #2 : data races + atomicity violations + order violations


```java

class Process extends Thread {
    private int id;
    private int currCap;
    private Boolean io_pending;
    
    ...
    
    public int ReadWriteProc() {

        int temp = currCap; // read the currCap
        io_pending = true; // set io_pending to be true

        while (io_pending) { 
            temp--; 
            currCap = temp;
        }
        return currCap;
    }

    public void DoneWaiting() {
        // do other stuff here
        io_pending = false; 
    }

    ...
}

// source : Mozilla bug altered and simplified in Java
```


3. Example #3 : atomicity violations + order violations



4. Example #4 : data races + atomicity violations




5. Example #5 : data races + order violations




## Running detection and repair tools on the above examples


1.1. Detection with existing tools

- Running RacerD results in the following:

```console
$ infer --racerd-only .java

```

- Running Chord


1.2. Fixing with existing tools

- Running Hippodrome results in the following outcome:

```console
$ infer

```

- Running HFix was not feasible as HFix was unavailable. I have contacted the authors of the paper but received no response.



## Additional questions that came up in the process

* As for the benchmark suites being used in the papers, how did they come up with new example programs in their papers? How good are these examples, i.e., how similar are they to real-life concurrency bugs?

* How about we list out all possible patterns of each type of concurrency bugs and try to connect one pattern to another? 


Some technical questions:
- Can I think of initialising a thread as a write operation? If so, there could be an overlapping pattern between order violations and data races.




<!-- ### References
<a id="1">[1]</a> 
 -->
