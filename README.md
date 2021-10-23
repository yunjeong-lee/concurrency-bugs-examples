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


## Thought Process

1\. Data races are often said to be associated with, if not the cause of, other types of concurrency bugs such as atomicity violations or order violations.

2\. How exactly are they interconnected? Is there any way to measure this interdependencies? 

  2.1. By the way, I couldn't find any work covering their dependencies or correlations, other than a qualitative, general statement. Let me see if it is any possible, though.
  
  2.2. First, what would be examples of buggy programs that contain more than one types of concurrency bugs? Found some examples, as shown in Examples #1-5 below.
  
  2.3. Findings from the examples:
      a. Quite a few concurrency bugs can be classified as more than one type of concurrency bugs. It is not that one type of concurrency bugs is a cause of another.
      b. Data races (buggy, not the benign, ones) and atomicity violations often overlap. There is even some paper on benchmark suite which classifies them as one type (call it "Races and atomicity violations").
      c. Data races and order violations also overlap due to write happening out of order between read operations.

3\. Seems to me that their interdependencies are hard to measure unless all possible patterns of each type are identified and we make connections between these patterns.

4\. Moreover, if these bugs are interconnected, how can we addresss--i.e., fix--multiple multiple types of (race condition) concurrency bugs at the same time? (Here, race condition bugs refer to atomicity violations, order violations, and data races.)

  4.1. At the end of the day, we want to fix all the bugs, or as much as possible. It is possible to have programs that are race-free but still contain other bugs such as atomicity violations (see Example #3) or fixing data races results in other bugs such as deadlocks. 

5\. By the way, how do existing tools perform with respect to the abovementioned examples? What are and aren't they capable of? Is it posisble that race repair tools end up fixing data races but atomicity violations, order violations, or even deadlocks remain unfixed?

  5.1. Findings from running existing tools (RacerD, Hippodrome) on the examples:
      a. With respect to detection using RacerD
      b. With respect to repair using Hippodrome
      c. Observations made on detection / repair using existing tools

6\. Since these bugs are relevant, how can we go about fixing these race condition bugs? How are the approaches of fixing atomicity violations or order violations?

  6.1. Atomicity violations and order violations are often caused by incorrect assumptions or absense of correct assumptions made on the atomicity of operations or order of execution of concurrent threads respectively. Can we provide these assumptions as constraints and have repair tool 



## Examples containing data races, atomicity violations, and/or order violations

#### Note: Examples below are collected in an attempt to find interdependencies between different types of concurrency bugs.

1\. Example #1 : data races + atomicity violations

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

2\. Example #2 : order violations + atomicity violations + data races

- Root case of the bugs can be diagnosed in relation to order violations, atomicity violations, or data races.
    * In the context of order violations, 
    * In the context of atomicity violations,
    * In the context of data races,

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


3\. Example #3 : atomicity violations + order violations

- The following example is free of data races, but still can be classified as atomicity violations or order violations. 
    * There are no data races as read and write operations on the shared variable `counter` are protected by lock `L`.
    * From atomicity violations perspective, the read and update of `counter` should have happened inside the same critical section.
    * In the context of order violations, `counter` should have been read before it is updated.
    * This bug can be fixed by making these read and write operations atomic, which will also correct the order.

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

4\. Example #4 : data races + atomicity violations




5\. Example #5 : data races + order violations




## Running detection and repair tools on the above examples


1\. Detection with existing tools

- Running RacerD results in the following:

```console
$ infer --racerd-only .java

```

- Running Chord : work in progress


2\. Fixing with existing tools

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

- By the way, how do existing detection tools identify benign data races from faulty ones? Assuming a high false positive rate is what makes programmers frustrated in using this tool, I am curious to know how the existing tools filter them out, if it's good enough, and if not, how to improve this filtering process.




<!-- ### References
<a id="1">[1]</a> 
 -->
