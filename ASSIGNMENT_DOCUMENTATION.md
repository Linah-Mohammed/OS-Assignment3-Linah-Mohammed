# Assignment 3 - Complete Documentation

**Student Name**: [Linah Mohammed Al-Tamimi]  
**Student ID**: [ 445052202 ]  
**Date Submitted**: [ May 7 , 2026 ]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [ May 5, 2026, 6:00 PM ]
**What I implemented**: 
I launched the Java project in Visual Studio Code, forked and cloned the assignment repository, and looked over the initial code. Additionally, I determined which common resources, such as `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`, and `executionLog`, needed to be synchronized.

**Challenges encountered**: 
Because the software comprises multiple methods that update shared data, it was first unclear which parts of the code were crucial.

**How I solved it**: 
I found every variable that multiple threads may access by tracing the methods inside the `SharedResources` class.

**Testing approach**: 
I reviewed the program structure and checked where each shared variable was updated.

**Time spent**: 
1 hour

---

### Entry 2 - [ May 5, 2026, 8:00 PM n]
**What I implemented**: 
I included the necessary synchronization imports: `ReentrantLock` and `Semaphore`. The `CounterLock`, `logLock`, and `cpuSemaphore` were then added to the `SharedResources` class.

**Challenges encountered**: 
I needed to decide whether to use one lock for all counters or separate locks.

**How I solved it**: 
I used one `ReentrantLock` for the shared counters to keep the solution simple and clear, and I used a separate lock for the execution log because it protects a different type of shared resource.

**Testing approach**: 
I checked that the imports were placed at the top of the file and that the new objects were declared as `public static final`.

**Time spent**: 
1 hour
---

### Entry 3 - [May 6, 2026, 5:30 PM ]
**What I implemented**: 
I used `counterLock` to safeguard the shared counter methods. Among these were `incrementContextSwitch()`, `incrementCompletedProcess()`, and `addWaitingTime(long time)`.

**Challenges encountered**: 
The main challenge was making sure every lock was released correctly after entering the critical section.

**How I solved it**: 
I used `try-finally` blocks so that `counterLock.unlock()` always runs even if an exception occurs.

**Testing approach**: 
I checked each method to make sure the lock is acquired before updating the shared variable and released in the `finally` block.

**Time spent**: 
1.5 hours
---

### Entry 4 - [ May 6, 2026, 8:00 PM ]
**What I implemented**: 
I protected the `executionLog` list using `logLock` inside the `logExecution(String message)` method.

**Challenges encountered**: 
`ArrayList` is not thread-safe, so concurrent updates could cause incorrect log entries or runtime exceptions.


**How I solved it**: 
I added a separate `ReentrantLock` for the execution log and placed `executionLog.add(message)` inside a locked critical section.


**Testing approach**:
 I reviewed all calls to `SharedResources.logExecution()` and confirmed that all log updates go through the protected method.

**Time spent**: 
1 hour

---

### Entry 5 - [ May 7, 2026, 4:30 PM ]
**What I implemented**:  
I added the binary semaphore to control CPU access in both `run()` and `runToCompletion()`. I used `cpuSemaphore.acquire()` before execution and `cpuSemaphore.release()` inside the `finally` block.

**Challenges encountered**:  
`acquire()` can throw `InterruptedException`, so I had to handle interruption correctly.

**How I solved it**:  
I added a `catch (InterruptedException e)` block and used a boolean variable called `permitAcquired` to release the semaphore only if the process actually acquired it.

**Testing approach**:  
I ran the program and verified that all processes completed successfully and that the final statistics appeared correctly.

**Time spent**:  
4 hours


---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**: 
The shared counter variables, including `contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`, have a single race condition. Multiple process threads change these variables, and because actions like `contextSwitchCount++` involve reading, changing, and writing the value, they are not atomic. One update may be lost if two threads update the same counter simultaneously, leading to inaccurate final statistics.

The `executionLog` resource, which is a `ArrayList`, had a second race condition. Because `ArrayList` is not thread-safe, it may become inconsistent or result in a `ConcurrentModificationException` if multiple threads add log messages simultaneously. This can lead to erroneous log sizes, missing log entries, or erratic program performance.


---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
`ReentrantLock` is used to provide mutual exclusion for a critical section. It allows only one thread at a time to access protected shared data. In my code, I used `counterLock` to protect the shared counters and `logLock` to protect the shared `executionLog`.

A `Semaphore` controls how many threads can access a limited resource at the same time. In my code, I used a binary semaphore called `cpuSemaphore` with one permit. This means only one process can access the simulated CPU execution section at a time. I used `ReentrantLock` for protecting shared variables and `Semaphore` for controlling CPU access.


---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:
When two or more threads are stuck waiting for a resource that another thread is holding, this is known as a deadlock. Always releasing locks and semaphores in a `finally` block is one preventive measure. Every `lock()` in my code has a corresponding `unlock()` inside of `finally`, and every semaphore permit that is obtained is released within of `finally`.

Keeping important portions brief and avoiding holding numerous locks needlessly are two more preventative strategies. Only the shared variable updates, like adding a log message or increasing a counter, are contained in the locked areas of my code. This lessens the possibility of deadlock and the amount of time a thread retains a lock.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:
For Task 1, I used one lock for the three counter variables. This is a coarse-grained locking approach because the same `counterLock` protects `contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`. I chose this design because it is simple, easy to understand, and reduces the chance of programming mistakes.

The trade-off is that coarse-grained locking provides less concurrency because only one thread can update any of the counters at a time. Fine-grained locking would use a separate lock for each counter, which could allow better concurrency because the counters are independent. However, fine-grained locking makes the code more complex. Since this assignment focuses on correctness and clarity, using one lock for the counters is a reasonable design choice.


---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
`contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`.

**Why they need protection**: 
They are shared variables updated by multiple threads. Without synchronization, simultaneous updates may cause lost updates and incorrect final statistics.

**Synchronization mechanism used**: 
`ReentrantLock` using `counterLock`.

**Code snippet**:
```java

// Lock for shared counter variables
public static final ReentrantLock counterLock = new ReentrantLock();

public static void incrementContextSwitch() {
    counterLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        counterLock.unlock();
    }
}

public static void incrementCompletedProcess() {
    counterLock.lock();
    try {
        completedProcessCount++;
    } finally {
        counterLock.unlock();
    }
}

public static void addWaitingTime(long time) {
    counterLock.lock();
    try {
        totalWaitingTime += time;
    } finally {
        counterLock.unlock();
    }
}
```

**Justification**: 
The lock ensures mutual exclusion. Only one thread can update the shared counters at a time, which prevents race conditions and keeps the final statistics correct.

---

### Critical Section #2: Execution Log

**What resource**: 
executionLog, which is an ArrayList<String>.

**Why it needs protection**:
ArrayList is not thread-safe. Multiple threads adding messages at the same time can cause inconsistent data or runtime exceptions.

**Synchronization mechanism used**: 
ReentrantLock using logLock.

**Code snippet**:
```java

// Lock for protecting execution log
public static final ReentrantLock logLock = new ReentrantLock();

public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}
```

**Justification**: 
Using a separate lock for the execution log keeps log updates safe and organized without mixing them with counter updates.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
The semaphore controls access to the simulated CPU execution section.

**Number of permits and why**: 
The semaphore has one permit because it is a binary semaphore. This allows only one process to execute on the simulated CPU at a time.

**Where implemented**: 
It was implemented in run() and runToCompletion().

**Code snippet**:
```java
// Semaphore for controlling CPU access
public static final Semaphore cpuSemaphore = new Semaphore(1);

boolean permitAcquired = false;

try {
    SharedResources.cpuSemaphore.acquire();
    permitAcquired = true;

    // process execution code

} catch (InterruptedException e) {
    System.out.println(Colors.RED + "\n  ✗ " + name + " was interrupted while waiting for CPU." + Colors.RESET);
} finally {
    if (permitAcquired) {
        SharedResources.cpuSemaphore.release();
    }
}

```

**Effect on program behavior**: 
The semaphore prevents more than one process from entering the CPU execution section at the same time. This makes the simulation safer and closer to a controlled single-CPU execution mode
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync

# Commands used (run the program at least 5 times)
```

**Results**: 
Total Context Switches: 33
Total Completed Processes: 18
Total Waiting Time: 1093858ms
Average Waiting Time: 60769ms

═══ Process Summary Table ═══
Process    Priority     Burst Time   Waiting Time
────────────────────────────────────────────────
P1         3            8382         88751       
P2         5            3481         4089        
P3         2            6949         66227       
P4         3            3095         11641       
P5         5            6700         69226       
P6         1            3702         18847       
P7         1            4869         71932       
P8         2            4522         72805       
P9         3            2153         30695       
P10        5            7005         73338       
P11        3            2517         36907       
P12        1            4308         76347       
P13        2            7300         76693       
P14        1            2484         47491       
P15        3            4768         79993       
P16        1            9771         89108       
P17        4            8551         90912       
P18        4            7857         88856       

═══ Execution Log Summary ═══
Total log entries: 66

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization,)
Synchronization is necessary because multiple threads access and modify shared resources concurrently. Without synchronization, race conditions could occur when different threads update variables such as contextSwitchCount, completedProcessCount, and totalWaitingTime. For example, two threads could update the same counter simultaneously, causing lost updates and incorrect final values.

The executionLog also requires protection because ArrayList is not thread-safe. Without synchronization, concurrent writes could corrupt the log or produce ConcurrentModificationException. The semaphore is also important because it controls access to the simulated CPU and ensures that only one process executes inside the CPU section at a time.

**Conclusion**: 
The synchronization mechanisms successfully protected the shared resources and produced consistent and reliable program behavior across multiple executions.

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 
I repeatedly executed the scheduler program while monitoring the execution log updates and the final execution log summary. I also verified that multiple processes could add log entries safely during execution.

**Results**: 
The program completed successfully in all tests without throwing ConcurrentModificationException or any thread-safety related errors. The execution log summary appeared correctly at the end of the program and reported:

Total log entries: 66

All log entries were added successfully without corruption or missing data.

**What this proves**: 
This proves that protecting executionLog with logLock successfully prevents unsafe concurrent access to the shared ArrayList. The synchronization mechanism guarantees safe logging behavior even when multiple threads attempt to update the log.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**:
 The number of completed processes should exactly match the number of created processes. Since the scheduler generated 18 processes, the expected completed process count was:

Expected Completed Processes = 18

I also expected:

Context switches to be greater than the number of processes because some processes require multiple quantums.
Waiting times to remain positive and reasonable.
Execution log entries to increase whenever a process starts, yields, or completes.

**Actual values**: 
Total Context Switches: 33
Total Completed Processes: 18
Total Waiting Time: 1093858ms
Average Waiting Time: 60769ms

**Analysis**: 
The expected scheduler behavior was consistent with the actual values. Since every produced process completed its execution, the completed process count was accurate. Because some processes needed more than one quantum to complete, the number of context switches exceeded the number of processes. Additionally, the log entries and waiting times matched the scheduling behavior seen during execution.

Concurrent thread updates were prevented from corrupting the final statistics thanks to the synchronization techniques.
---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 
The purpose of this test was to verify that synchronization still works correctly when some processes complete within one quantum while other processes require multiple quantums and repeated context switches.

**Results**: 
The scheduler handled all scenarios correctly. Processes with short burst times completed immediately after one execution quantum, while processes with longer burst times yielded the CPU and returned to the ready queue for another execution cycle.

The semaphore correctly limited CPU access to one process at a time, and the locks protected all shared counters and log updates during the entire scheduling process.

No deadlocks, race conditions, or synchronization errors appeared during execution.

**What I learned**: 
I discovered that because thread scheduling is managed by the operating system and might result in erratic execution timing, synchronization is required even when the program seems to run sequentially. Additionally, I discovered that various synchronization techniques address various issues. While semaphores are helpful for limiting access to shared resources like the CPU, locks are useful for safeguarding important areas.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:
I now have a better understanding of the significance of synchronization in multithreaded systems thanks to this assignment. I discovered that when several threads alter common resources without protection, race circumstances arise. Because they are not atomic operations, even basic activities like increasing a counter might yield inaccurate results.

Additionally, I discovered how ReentrantLock prevents concurrent access to important portions and offers mutual exclusion. I also discovered that semaphores can regulate access to finite resources, such the CPU. Because a binary semaphore only permits one process to run at a time, it was easier to simulate regulated CPU access.

The use of try-finally blocks was another crucial lesson. I discovered that, in order to prevent deadlocks, locks and semaphore permits must always be relinquished, regardless of exceptions. My comprehension of operating system concurrent execution, context switching, and process scheduling has also enhanced as a result of this project.


---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking systems use synchronization to protect account balances during concurrent deposits and withdrawals. Without synchronization, simultaneous transactions could produce incorrect balances or lost updates.

**Example 2**: 
Operating systems use synchronization when multiple processes access shared resources such as printers, files, memory, or CPU resources. Synchronization prevents resource corruption and ensures fair resource sharing.

---

### How I would explain synchronization to others:
Synchronization is comparable to sharing a room with a key. Only the person with the key is permitted to enter the room if several individuals wish to change something crucial. When they're done, they give the key back so someone else can enter safely.

In operating systems, the key is the lock or semaphore, while the shared room is the crucial area. Synchronization maintains the system accurate and stable by preventing threads from changing shared resources simultaneously.


---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Linah-Mohammed/OS-Assignment3-Linah-Mohammed.git

**Number of commits**: 14

**Commit messages**: 
1. change to student ID 445052202
2. Add synchronization libraries
3. Add locks and semaphore to shared resources
4. Protect context switch counter with ReentrantLock
5. Protect completed process counter with ReentrantLock
6. Protect total waiting time with ReentrantLock
7. Protect execution log with ReentrantLock
8. Add CPU semaphore acquire and release
9. Add CPU semaphore to runToCompletion
10. Handle semaphore interruption in run method
11. answering Part 1: Development Log & Part 2: Technical Questions
12. answering Part 3: Synchronization Analysis
13. answering Part 4: Testing and Verification
14. answering Part 5: Reflection and Learning
   
   


---

## Summary

**Total time spent on assignment**: day

**Key takeaways**: 
1. Race conditions occur when multiple threads access shared data without synchronization. 
2. ReentrantLock protects critical sections and shared variables from concurrent access.
3. Semaphore controls access to limited shared resources such as the simulated CPU.


**Most challenging aspect**: 
The most challenging aspect was correctly implementing the semaphore logic and ensuring that the semaphore permit was always released safely using try-finally blocks.

**What I'm most proud of**: 
I am most proud that I successfully protected all shared resources, prevented race conditions, and implemented synchronization mechanisms that produced correct and consistent scheduler behavior.

---

**End of Documentation**
