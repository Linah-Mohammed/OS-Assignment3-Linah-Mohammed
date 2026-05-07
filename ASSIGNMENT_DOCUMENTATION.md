# Assignment 3 - Complete Documentation

**Student Name**: [Linah Mohammed Al-Tamimi]  
**Student ID**: [ 445052202 ]  
**Date Submitted**: [Submission Date]

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

**Why they need protection**: 

**Synchronization mechanism used**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Justification**: 

---

### Critical Section #2: Execution Log

**What resource**: 

**Why it needs protection**: 

**Synchronization mechanism used**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Justification**: 

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 

**Number of permits and why**: 

**Where implemented**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Effect on program behavior**: 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
(Show that running multiple times produces consistent, correct results)

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

**Conclusion**: 

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 

**Results**: 

**What this proves**: 

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 

**Actual values**: 

**Analysis**: 

---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 

**Results**: 

**What I learned**: 

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[6-8 sentences about key concepts, challenges, insights]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 

**Example 2**: 

---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 

**Commit messages**: 
1. 
2. 
3. 
4. 

---

## Summary

**Total time spent on assignment**: 

**Key takeaways**: 
1. 
2. 
3. 

**Most challenging aspect**: 

**What I'm most proud of**: 

---

**End of Documentation**
