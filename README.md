## Concurrency Bugs Examples

### Types of Concurrency Bugs

There are various ways to categorise concurrency bugs. In this article, we classify them into four different categories: _atomicity violations_, _order violations_, _data races_, and _deadlocks_.

1. Atomicity Violations
- Concurrency bugs that occur when atomicity assumptions made about the execution 
- Aomicity refers to the property of a multithreaded program segment that allows the segment to appear as if it occurred instantaneously to the rest of the system [[1]](#1).

2. Order Violations
- Definition: 

3. Data Races
- Definition: 

4. Deadlocks
- Definition: 


#### Notes

1. Data Races
- Data races and race conditions are not the same. There are data races that are not race conditions as well as race conditions that are not data races.
- At the same time, not all data races are buggy. There are benign data races.
- Often, it requires a deeper understanding of the software to decide whether data races identified are true bugs or not.

2. Race Conditions
- Race conditions occur when 
- In this context, we consider data races as a type of race condition bugs.

3. Examples
- These examples are collected in an attempt to find interdependencies between different types of concurrency bugs.
-  

#### Buggy programs containing both data races (D) and atomicity violations (A)

- DA Example #1

Below example includes both data raes and atomicity violations.
This can be 

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
        if (!a.contains(x)) {
            a.add(x);
        }
    }
}
```

- DA Example #2




- DA Example #3



### References
<a id="1">[1]</a> 

