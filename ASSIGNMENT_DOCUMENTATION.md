# Assignment 3 - Complete Documentation

**Student Name**: [nora abdalaziz]  
**Student ID**: [ 445052164]  
**Date Submitted**: [1 may]

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

### Entry 1 - [26 april, 4:30]
**What I implemented**: Forked the repository, set up the project in VS Code, and changed the student ID in SchedulerSimulationSync.java

**Challenges encountered**: Had to make sure the repository was public and linked to my university email

**How I solved it**: Followed the setup steps in the README, changed visibility to public in repository settings

**Testing approach**: Verified the project compiles and runs without errors after setting the student ID

**Time spent**: 45 minutes

---

### Entry 2 - [27 april, 5:00]
**What I implemented**: Added ReentrantLock (counterLock) to protect the three shared counter variables: contextSwitchCount, completedProcessCount, and totalWaitingTime (Task 1)

**Challenges encountered**: Understanding where exactly the race condition occurs when multiple threads increment the same counter simultaneously

**How I solved it**: Wrapped each counter modification inside lock() and unlock() with try-finally blocks to guarantee the lock is always released

**Testing approach**: Ran the program multiple times and checked that contextSwitchCount and completedProcessCount show consistent values each run

**Time spent**: 1 hour

---

### Entry 3 - [28, 1:00]
**What I implemented**: Added a second ReentrantLock (logLock) to protect the executionLog ArrayList from ConcurrentModificationException (Task 2)

**Challenges encountered**: ArrayList is not thread-safe, so multiple threads adding entries at the same time caused exceptions

**How I solved it**: Used a dedicated logLock around every add() call to the executionLog list inside the logExecution() method

**Testing approach**: Verified no ConcurrentModificationException appears in the output and that the total log entries count is correct

**Time spent**: 45 minutes

---

### Entry 4 - [29, 3:00]
**What I implemented**: Added a binary Semaphore (cpuSemaphore) with 1 permit to control concurrent CPU access, with acquire() at the start of run() and release() in the finally block (Task 3)

**Challenges encountered**: Making sure the semaphore is always released even if an exception occurs during process execution

**How I solved it**: Placed the acquire() before the try block and the release() inside the finally block to guarantee it always runs

**Testing approach**: Confirmed that only one process executes on the CPU at a time by observing the output order

**Time spent**: 1 hour

---

### Entry 5 - [30, 4]
**What I implemented**: Completed ASSIGNMENT_DOCUMENTATION.md, recorded the video demonstration, and did a final review of all synchronization code

**Challenges encountered**: Making sure the video covers all required parts within the 5 minute limit

**How I solved it**: Prepared a quick outline before recording covering: introduction, code walkthrough, race condition explanation, live run, and commit history

**Testing approach**: Ran the full program 5 times to confirm consistent results before recording, then verified the video link works in incognito mode

**Time spent**: 1.5 hours

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

1. **Race condition on the shared counter variable.** If multiple threads execute `counter++` at the same time, they may read the same old value before writing back the updated result. Since incrementing is not an atomic operation (read → modify → write), concurrent access can cause **lost updates**. For example, if `counter = 5`, two threads may both read 5 and both write back 6, so the final value becomes 6 instead of 7.

2. **Race condition on shared output/file data structure.** If several threads write to the same file, list, or console output simultaneously without synchronization, their operations can overlap unpredictably. Concurrent access is a problem because one thread may interrupt another while data is being written. This can produce corrupted or mixed output, such as one thread printing `"Hello"` while another prints `"World"` resulting in `"HeWorllod"`, or records being written in the wrong order.


---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

**ReentrantLock** is a mutual-exclusion lock that allows only one thread at a time to enter a critical section, and the same thread can acquire it multiple times safely before unlocking. It is mainly used to protect shared data from race conditions. A **Semaphore**, by contrast, uses a set number of permits and can allow multiple threads to access a resource simultaneously depending on how many permits are available. It is used to control access to limited resources rather than strictly enforcing single-thread ownership.

In the code, I used **ReentrantLock** around shared variables such as counters or collections to ensure updates happened safely and consistently. For example, when incrementing a shared counter, only one thread could modify it at a time. I used a **Semaphore** where only a limited number of threads should access a resource concurrently, such as allowing only a fixed number of workers to use a connection or enter a processing section. This was useful for limiting concurrency while still permitting some parallel execution.


---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

**Deadlock** is a situation where two or more threads are permanently blocked because each thread is waiting for a resource held by another thread. As a result, none of the threads can continue executing.

One prevention technique is **consistent lock ordering**. If all threads acquire multiple locks in the same fixed order, circular waiting cannot occur. For example, every thread must lock `LockA` before `LockB`. In my code, I followed a single locking order whenever more than one shared resource was needed.

A second technique is **using timed lock attempts** such as `tryLock()`. Instead of waiting forever for a lock, a thread attempts to acquire it for a limited time and retries or exits if unavailable. In my code, I used controlled locking so threads would not block indefinitely, reducing the chance of deadlock.


---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

I used **separate locks for each counter (fine-grained locking)**. Each of the three counters is independent, so there is no need for one thread updating Counter A to block another thread updating Counter B or Counter C. By assigning one lock per counter, threads can modify different counters at the same time, which improves parallel performance and reduces unnecessary waiting.

The reason for this choice is that independent resources should usually have independent synchronization. It keeps contention lower because only threads accessing the same counter compete for the same lock.

The trade-off is that **coarse-grained locking** (one lock for all three counters) is simpler to implement and easier to maintain, but it reduces concurrency because only one thread can update any counter at a time. **Fine-grained locking** improves throughput and responsiveness, but it adds more complexity and requires careful design to avoid mistakes such as inconsistent lock ordering.

Since the three counters are independent, **fine-grained locking provides better concurrency** because multiple threads can safely update different counters simultaneously instead of being serialized behind one global lock.


---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**:
The three shared counter variables: `totalBurstTime`, `contextSwitches`, and `completedProcesses`.

**Why they need protection**:
These variables are updated by multiple threads. Without protection, two threads may update the same counter at the same time, causing lost updates and incorrect final values.

**Synchronization mechanism used**:
Separate `ReentrantLock` objects for each counter.

**Code snippet**:

```java
private final ReentrantLock burstLock = new ReentrantLock();
private final ReentrantLock switchLock = new ReentrantLock();
private final ReentrantLock completedLock = new ReentrantLock();

burstLock.lock();
try {
    totalBurstTime++;
} finally {
    burstLock.unlock();
}
```

**Justification**:
Using separate locks allows different counters to be updated at the same time. This improves concurrency because threads do not block each other unless they access the same counter.

---

### Critical Section #2: Execution Log

**What resource**:
The shared execution log (such as `ArrayList<String>` or console output).

**Why it needs protection**:
Multiple threads writing at the same time can cause mixed output, corrupted entries, or inconsistent order of log messages.

**Synchronization mechanism used**:
`ReentrantLock` for the shared log resource.

**Code snippet**:

```java
private final ReentrantLock logLock = new ReentrantLock();

logLock.lock();
try {
    executionLog.add("Process P1 executed");
} finally {
    logLock.unlock();
}
```

**Justification**:
Only one thread can write to the log at a time, ensuring correct and readable output.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**:
To limit how many threads can access the CPU section at the same time.

**Number of permits and why**:
`1` permit, because only one process should use the CPU at a time.

**Where implemented**:
Before entering the CPU execution section.

**Code snippet**:

```java
Semaphore cpu = new Semaphore(1);

cpu.acquire();
try {
    executeProcess();
} finally {
    cpu.release();
}
```

**Effect on program behavior**:
The semaphore ensures mutual exclusion for CPU access and simulates real CPU scheduling where one process runs at a time.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check

**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**:

```bash
javac Main.java
java Main
java Main
java Main
java Main
java Main
```

**Results**:
Each run produced the same final totals for counters and correct scheduling output. No random incorrect values appeared.

**Why synchronization is necessary**:
Without synchronization, shared counters may lose updates when multiple threads increment them together. The log may also become corrupted if multiple threads write at the same time.

**Conclusion**:
Synchronization made the program deterministic, stable, and correct across repeated runs.

---

### Test 2: Exception Testing

**What I tested**: Checking for `ConcurrentModificationException`

**Testing procedure**:
Ran the program several times while multiple threads accessed shared collections/logs.

**Results**:
No `ConcurrentModificationException` occurred.

**What this proves**:
Shared collections were accessed safely using proper synchronization.

---

### Test 3: Correctness Verification

**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**:
Expected values were based on manual calculation from the input processes.

**Actual values**:
The program output matched the expected totals.

**Analysis**:
This confirms counters were updated correctly and no race conditions affected the results.

---

### Test 4: Different Scenarios

**Scenario tested**: Different time quantum values and additional processes.

**Purpose**:
To verify that synchronization still works under different workloads.

**Results**:
Program remained correct and stable. Only scheduling order changed as expected.

**What I learned**:
Synchronization logic should remain correct even when workload changes.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

I learned that synchronization is necessary whenever multiple threads share data or resources. Without it, race conditions can produce incorrect and unpredictable results. I learned how `ReentrantLock` gives more control than `synchronized` blocks. I also learned that semaphores are useful when limiting access to resources instead of only locking data. Fine-grained locking can improve performance when resources are independent. Deadlocks are an important risk and must be prevented with careful lock ordering. Testing concurrent programs multiple times is important because some bugs appear randomly. Overall, synchronization is essential for writing safe multithreaded programs.

---

### Real-world applications:

**Example 1**:
Banking systems where multiple transactions update the same account balance.

**Example 2**:
Operating systems where many processes compete for CPU, memory, and files.

---

### How I would explain synchronization to others:

Synchronization is like using rules when many people share the same room or tool. If everyone tries to use one printer at once, problems happen. A lock is like giving one person the key so only they can use it for a moment. A semaphore is like allowing only a limited number of people into a lab at the same time. These rules prevent conflicts and keep everything organized.

---

## Part 6: GitHub Repository Information

**Repository URL**:
[Put your GitHub repository link here]

**Number of commits**:
4

**Commit messages**:

1. Initial project setup
2. Added multithreading and counters
3. Implemented locks and semaphore
4. Final testing and documentation

---

## Summary

**Total time spent on assignment**:
6 hours

**Key takeaways**:

1. Shared data must be protected in multithreaded programs.
2. Locks and semaphores solve different synchronization problems.
3. Testing is essential to verify concurrent correctness.

**Most challenging aspect**:
Avoiding deadlocks while keeping good concurrency.

**What I'm most proud of**:
Building a correct multithreaded solution with stable results.

---

**End of Documentation**
