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


## Thought Process Reg. Interdependencies between Concurrency Bugs

1\. Data races are often said to be associated with, if not the cause of, other types of concurrency bugs such as atomicity violations or order violations.

2\. How exactly are they interconnected? Is there any to measure this interdependencies in order to target multiple kinds of concurrency bugs at the same time? 

  2.1. Evaluating correlations is one way to measure it, but in this concurrency bug context, is it even posible to measure correlations (or interdependencies via some alternative way)?

  2.2. By the way, I couldn't find any work covering their dependencies or correlations, other than a qualitative, general statement. Let me see if it is any possible, though.
  
3\. What would be examples of buggy programs that contain more than one types of concurrency bugs? Found some examples, as shown in Examples #1-5 below.
  
  3.1. Findings from the examples:
  - Quite a few concurrency bugs can be classified as more than one type of concurrency bugs. It is not that one type of concurrency bugs is a cause of another.
  - Data races (buggy, not the benign, ones) and atomicity violations often overlap. There is even some paper on benchmark suite which classifies them as one type (call it "Races and atomicity violations").
  - Data races and order violations also overlap due to write happening out of order between read operations.

4\. Seems to me that their interdependencies are hard to measure unless all possible patterns of each type are identified and we make connections between these patterns.

  4.1. These patterns would vary based on the programming language used in concurrent programs.

5\. Moreover, if these bugs are interconnected, how can we addresss--i.e., fix--multiple multiple types of (race condition) concurrency bugs at the same time? (Here, race condition bugs refer to atomicity violations, order violations, and data races.)

  5.1. At the end of the day, we want to fix all the bugs, or as much as possible. It is possible to have programs that are race-free but still contain other bugs such as atomicity violations (see Example #3) or fixing data races results in other bugs such as deadlocks. 

6\. By the way, how do existing tools perform with respect to the abovementioned examples? What are and aren't they capable of? Is it posisble that race repair tools end up fixing data races but atomicity violations, order violations, or even deadlocks remain unfixed?

(Below in progress)
  6.1. Findings from running existing tools (RacerD, Hippodrome)
  - With respect to detection using RacerD
  - With respect to repair using Hippodrome
  - Observations made on detection / repair using existing tools

7\. Since these bugs are relevant, how can we go about fixing these race condition bugs? How are the approaches of fixing atomicity violations or order violations?

  7.1. Atomicity violations and order violations are often caused by incorrect assumptions or absense of correct assumptions made on the atomicity of operations or order of execution of concurrent threads respectively. Can we provide these assumptions as constraints and have repair tool 



## Examples containing data races, atomicity violations, and/or order violations

#### Note: Examples below are collected in an attempt to find interdependencies between different types of concurrency bugs.

1\. Example #1 : data races + atomicity violations

- Root case of the bugs can be diagnosed in relation to data races or atomicity violations.
    * In the context of data races, there may be threads writing to the list `a` via `a.add(x);` while others simultaneously read it via `!a.contains(x)`.
    * In the context of atomicity violations, the fact that checking `if (!a.contains(x))` and modifying `a.add(x)` were not an atomic operation caused the bug.
    * This bug can be fixed by adding a synchronisation primitive and making the operations atomic, as shown in the comments.

```java
public class User extends Thread {
    List<Integer> list;
    // List<Integer> list;
    Integer elem;

    public User(List<Integer> ls, Integer x) {
        this.list = ls;
        this.elem = x;
    }
    
    @Override
    public void run() { // addIfAbset(int x)
        // synrhonized(a) {     --- fix to the atomicity violation
        if (!list.contains(elem)) {
            list.add(elem);
        }
        // }                    --- fix to the atomicity violation
    }
}

public class Main {
    static final List l = Collections.synchronizedList(new ArrayList());

    public static void main(String[] args) {
        User[] users = new User[4];
        for (int i = 0; i < users.length; i++) {
            users[i] = new User(l, 17);
            users[i].start();
        }

        for (int j = 0; j < users.length; j++) {
            users[j].join();
            System.out.println(l);
        }

    }
}

// source : originally taken from IntelliJ IDEA Debugging Tutorial and altered
```

2\. Example #2 : order violations + atomicity violations + data races

- Root case of the bugs can be diagnosed in relation to order violations, atomicity violations, or data races.
    * In the context of order violations, `io_pending` should be set to `true` first before it is set to `false` by another thread. That is, programmers expected `S2` to happen before `S3`. However, if `S3` happens before `S2`, the thread will hang inside the while loop.
    * In the context of atomicity violations, reading the `currProc` (`S1`) and setting `io_pending` equal to `true` (`S2`) did not happen atomically and caused this bug. 
    * In the context of data races, read and write operations of shared varialbe `io_pending` without proper synchronisation caused this bug.

```java

class Process extends Thread {
    ...
    
    public int ReadWriteProc() {

        int temp = currProc; // S1: read the currProc
        io_pending = true;   // S2: set io_pending to be true

        while (io_pending) { 
          // spin    
        }
        ...
    }

    public void DoneWaiting() {
        
        // do other stuff here
        io_pending = false; // S3: set io_pending to be false
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

<!-- 4\. Example #4 : data races + atomicity violations




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


 -->
## Additional questions that came up in the process

* As for the benchmark suites being used in the papers, how did they come up with new example programs in their papers? How good are these examples, i.e., how similar are they to real-life concurrency bugs?

* How about we list out all possible patterns of each type of concurrency bugs and try to connect one pattern to another? 


<!-- Some technical questions:
- Can I think of initialising a thread as a write operation? If so, there could be an overlapping pattern between order violations and data races.
 -->
- By the way, how do existing detection tools identify benign data races from faulty ones? Assuming a high false positive rate is what makes programmers frustrated in using this tool, I am curious to know how the existing tools filter them out, if it's good enough, and if not, how to improve this filtering process.



## Other Examples

### Races caused by aliasing

```java
@ThreadSafe
class HR {
    
    Dept myDept = new Dept();
   
    protected void assignEmployees(int x) {
        Size sz = myDept.getSize();
        sz.val = x;
    }
}

class Dept {

    Size size;

    Size getSize() { 
        return size;
    }
}

class Size {
    int val;
}

```

In the above example, RacerD fails to detect races caused by aliasing:

```bash
~/Documents/program-repair/racerdfix/src/test/java/aliastest$ infer --racerd-only -- javac -cp annotations.jar *.java
Capturing in javac mode...
Found 4 source files to analyze in /home/yunjeonglee/Documents/program-repair/racerdfix/src/test/java/aliastest/infer-out
4/4 [################################################################################] 100% 50.911ms

  No issues found  
```

On the other hand, multiple HR threads simultaneously writing on the shared department data, resulting over-assigned department:

```bash
~/Documents/program-repair/racerdfix/src/test/java/aliastest$ java aliastest.Main 
Dept 3 has 4 employees
Dept 3 has 2 employees
Dept 1 has 2 employees
Dept 2 has 2 employees
Dept 4 has 2 employees
Dept 5 has 2 employees

```


### Other concurrency bugs may be left unfixed

* A data race example from HIPPODROME paper 

```java
public class CustomerInfo {  
  
 	private Account[] accounts; 
	
	// more fields and methods
  
 	public void withdraw(int accountNumber, int amount){ 
 		int temp = accounts[accountNumber].getBalance(); 
 		temp = temp - amount; 
 		accounts[accountNumber].setBalance(temp); 
 	} 
 	 
 	public void deposit(int accountNumber, int amount){ 
 		int temp = accounts[accountNumber].getBalance(); 
 		temp = temp + amount; 
 		accounts[accountNumber].setBalance(temp); 
 	} 
}

public class Account {

	private int balance;

	public int getBalance() {
		return balance;
	}

	public void setBalance(int balance) {
		this.balance = balance;
	}	
}

// Testing Thread
public class ThreadRun extends Thread{

	CustomerInfo ci;

	@Override
	public void run() {
		ci.deposit(1, 50);
	}
}

```

* After running the HIPPODROME, the repaired program has atomicity violations / order violations left unfixed without optimisation

```java
public class CustomerInfo {  
  
 	private Account[] accounts; 
	
	// more fields and methods
  
 	public void withdraw(int accountNumber, int amount){ 
 		int temp = accounts[accountNumber].getBalance(); 
 		temp = temp - amount; 
 		accounts[accountNumber].setBalance(temp); 
 	} 
 	 
 	public void deposit(int accountNumber, int amount){ 
 		int temp = accounts[accountNumber].getBalance(); 
 		temp = temp + amount; 
 		accounts[accountNumber].setBalance(temp); 
 	} 
}


public class Account { 
        
	Object lock1 =  new Object();  
  
 	private int balance; 
  
 	public int getBalance() { 
 		synchronized(lock1) { return balance; }  
 	 
 	} 
  
 	public void setBalance(int balance) { 
 		synchronized(lock1) { this.balance = balance; }  
 	 
 	} 
 	 
}
```


### Repair Algorithm Example - Most frequent lock is chosen

* If there are no existing locks, the repair algorithm creates one

```java
class Account {  int amt = 0;  }     
          
@ThreadSafe     
class Test7 {
 
    Account myAccount;           
    
    public Test7(Account a) { myAccount = a;}     
          
    public void deposit1(int x) {     
        myAccount.amt = myAccount.amt + x;
    }     
      
    public void deposit2(int x) {
        myAccount.amt = myAccount.amt + x;
    }

    public void withdraw1(int x) {     
        myAccount.amt = myAccount.amt - x;
    }
      
    public void withdraw2(int x) {  
        myAccount.amt = myAccount.amt - x;
    }      
} 
```

* Once the above is repaired, it results in the following:

```java
class Test7 { 

    Object newLock =  new Object();
        
    Account myAccount;            
     
    public Test7(Account a) { myAccount = a;}      

    public void deposit1(int x) {      
        synchronized(newLock) { myAccount.amt = myAccount.amt + x; }  
    }      
 
    public void deposit2(int x) { 
        synchronized(newLock) { myAccount.amt = myAccount.amt + x; }  
    }

    public void withdraw1(int x) {      
        synchronized(newLock) { myAccount.amt = myAccount.amt - x; }  
    } 
    
    public void withdraw2(int x) {   
        synchronized(newLock) { myAccount.amt = myAccount.amt - x; }  
    }
}
```

* Suppose there are existing locks, as shown below:

```java
class Test7 {

    Object existingLock1 =  new Object();      
    Object existingLock2 =  new Object();      
      
    Account myAccount;           
    
    public Test7(Account a) { myAccount = a;}     
          
    public void deposit1(int x) {     
        synchronized(existingLock1) { myAccount.amt = myAccount.amt + x; }
    }     
      
    public void deposit2(int x) {
        synchronized(existingLock2) { myAccount.amt = myAccount.amt + x; }
    }

    public void withdraw1(int x) {     
        synchronized(existingLock1) { myAccount.amt = myAccount.amt - x; }
    }
      
    public void withdraw2(int x) {  
        myAccount.amt = myAccount.amt - x; // unprotected, causing data races
    }      
}
```

* Repair algorithm may go for the most frequently used lock, based on the reasoning that fewer number of locks can be created if we opt for the most fequently used lock. Hence, repaired program looks like the following:

```java
 class Test7 { 
  
     Object existingLock1 =  new Object();       
     Object existingLock2 =  new Object();       
        
     Account myAccount;            
      
     public Test7(Account a) { myAccount = a;}      
            
     public void deposit1(int x) {      
         synchronized(existingLock1) { myAccount.amt = myAccount.amt + x; } 
     }      
        
     public void deposit2(int x) { 
         synchronized(existingLock2) { synchronized(existingLock1) { myAccount.amt = myAccount.amt + x; }   } 
     } 
  
     public void withdraw1(int x) {      
         synchronized(existingLock1) { myAccount.amt = myAccount.amt - x; } 
     } 
        
     public void withdraw2(int x) {   
         synchronized(existingLock1) { myAccount.amt = myAccount.amt - x; }  
      
     }       
 } 
```

* However, this may cause the performance slowdown, as this could cause more threads waiting for the shared resource (lock).


<!-- ### References
<a id="1">[1]</a> 
 -->
