# 🏆 COMP8567 ULTIMATE EXAM GUIDE - PART 1/9
## PROCESS CONTROL & FORK
## Your Champion's Guide to Mastering Fork!

---

# 📚 PART 1: PROCESS CONTROL & FORK

## 🎯 WHAT YOU'LL MASTER IN THIS SECTION:
- ✅ How fork() works (return values, process creation)
- ✅ Counting processes (formulas you MUST know)
- ✅ Process trees and visualization
- ✅ waitpid() and process synchronization
- ✅ exec() family functions
- ✅ Process states and lifecycle
- ✅ 80+ True/False questions for EASY marks!

---

## 📖 SECTION 1: FORK FUNDAMENTALS

### What is fork()?
**fork()** is a system call that creates a NEW process by duplicating the calling process.

**Key Points:**
- Creates an EXACT copy of the calling process
- The NEW process is called the **CHILD**
- The ORIGINAL process is called the **PARENT**
- After fork(), BOTH processes exist and run concurrently
- Both processes continue from the line AFTER fork()

### The Golden Rule of fork() Return Values
```c
pid_t pid = fork();

// After this line, TWO processes are running!

// In PARENT process:
//   pid = child's PID (a POSITIVE number, e.g., 12345)

// In CHILD process:
//   pid = 0 (ZERO)

// On ERROR (no child created):
//   pid = -1
```

**🔑 CRITICAL INSIGHT:**
The return value is HOW you distinguish parent from child!

### Example 1: Basic Fork
```c
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();
    
    if (pid > 0) {
        // PARENT process (got child's PID)
        printf("I am parent, my child is %d\n", pid);
    } 
    else if (pid == 0) {
        // CHILD process (got 0)
        printf("I am child, my PID is %d\n", getpid());
    }
    else {
        // ERROR
        perror("fork failed");
    }
    
    return 0;
}
```

**Output (unpredictable order):**
```
I am parent, my child is 12345
I am child, my PID is 12345
```
OR
```
I am child, my PID is 12345
I am parent, my child is 12345
```

**Why unpredictable?** OS scheduler decides which runs first!

---

## 📖 SECTION 2: PROCESS COUNTING - THE EXAM'S FAVORITE!

### Formula #1: Sequential Forks
**n sequential fork() calls → 2^n processes**

```c
// Example 1: Three sequential forks
fork();    // 1 → 2 processes
fork();    // 2 → 4 processes  
fork();    // 4 → 8 processes

// Total: 2^3 = 8 processes
```

**Why?** Each fork() DOUBLES the number of processes:
- Start: 1 process
- After 1st fork: 2 processes (parent + child)
- After 2nd fork: 4 processes (all 2 fork again)
- After 3rd fork: 8 processes (all 4 fork again)

### Formula #2: Fork in a Loop
**for(i=0; i<n; i++) fork() → 2^n processes**

```c
// Example 2: Fork in a loop
for(int i = 0; i < 4; i++) {
    fork();
}
// Total: 2^4 = 16 processes
```

**Iteration breakdown:**
- i=0: fork() → 1 becomes 2
- i=1: fork() → 2 becomes 4
- i=2: fork() → 4 becomes 8
- i=3: fork() → 8 becomes 16

### Formula #3: Conditional Fork
**if(fork()) fork() → 3 processes**

```c
// Example 3: Conditional fork
if(fork()) {    // Parent enters (fork returns > 0)
    fork();     // Parent forks again
}
// Child skips if block (fork returns 0)
```

**Process tree:**
```
     [Parent]
      /    \
   fork()  fork()
    /        \
[Child1]    [Parent creates Child2]

Total: Parent + Child1 + Child2 = 3 processes
```

### Example 4: The T1 Question Pattern
```c
int pid1, pid2, pid3;
pid1 = fork();   // 2 processes
pid2 = fork();   // 4 processes (both fork)
pid3 = fork();   // 8 processes (all 4 fork)

// How many processes total? 8
// How many have all three > 0? ONLY original parent (1)
```

**Process breakdown:**
```
After pid1 = fork():
  Parent: pid1 = child's PID (positive)
  Child:  pid1 = 0

After pid2 = fork() [BOTH do it]:
  Parent: pid1 > 0, pid2 > 0
  Child A: pid1 = 0, pid2 > 0  
  Child B: pid1 > 0, pid2 = 0
  Child C: pid1 = 0, pid2 = 0

After pid3 = fork() [ALL FOUR do it]:
  8 processes total

Only original parent has: pid1 > 0 AND pid2 > 0 AND pid3 > 0
```

---

## 📖 SECTION 3: PROCESS TREES - VISUALIZE TO WIN!

### How to Draw Process Trees

**Step 1:** Start with one process (parent)
```
[P]
```

**Step 2:** After first fork(), split into two
```
   [P] (parent, gets child PID)
    |
   [C1] (child, gets 0)
```

**Step 3:** After second fork(), BOTH split
```
   [P]              [C1]
   / \              / \
[P] [C2]        [C1] [C3]

Final: P, C1, C2, C3 = 4 processes
```

### Example: Tracking Variables Across Forks
```c
int x = 5;
int pid1 = fork();

// After fork, TWO processes:
// Parent: x=5, pid1=(child's PID, e.g., 100)
// Child:  x=5, pid1=0

if(pid1 == 0) {
    x = 10;  // ONLY child changes x
}

printf("x = %d\n", x);

// Parent prints: x = 5
// Child prints:  x = 10
```

**🔑 CRITICAL:** After fork(), parent and child have SEPARATE memory!
Changes in one do NOT affect the other!

---

## 📖 SECTION 4: WAITPID - THE EXAM KILLER TOPIC!

### What is waitpid()?
```c
pid_t waitpid(pid_t pid, int *status, int options);

// Parameters:
//   pid: which child to wait for
//   status: pointer to store exit status (can be NULL)
//   options: 0 (block) or WNOHANG (non-blocking)

// Returns: PID of terminated child, or -1 on error
```

### Common Uses:
```c
// Wait for specific child
waitpid(child_pid, NULL, 0);

// Wait for any child (like wait())
waitpid(-1, NULL, 0);

// Non-blocking wait
waitpid(pid, NULL, WNOHANG);
```

### THE VARIABLE OVERWRITING TRAP! 🚨
**THIS IS IN EVERY EXAM!**

```c
int result;
result = waitpid(pid1, NULL, 0);  // result = pid1
result = waitpid(pid2, NULL, 0);  // result = pid2 ← OVERWRITES!
result = waitpid(pid3, NULL, 0);  // result = pid3 ← OVERWRITES AGAIN!

printf("%d", result);  // Prints: pid3 (LAST assignment wins!)
```

**MEMORIZE THIS RULE:**
**When the same variable is assigned multiple times, ONLY the LAST value remains!**

### Example: The Exact T1 Question
```c
int main() {
    int pid1, pid2, pid3;
    
    pid1 = fork();  // 2 processes
    pid2 = fork();  // 4 processes
    pid3 = fork();  // 8 processes
    
    if(pid1 > 0 && pid2 > 0 && pid3 > 0) {
        // Only ORIGINAL parent enters here
        int child_pid;
        
        child_pid = waitpid(pid1, &status, 0);  // child_pid = pid1
        child_pid = waitpid(pid2, &status, 0);  // child_pid = pid2 ← overwrites
        child_pid = waitpid(pid3, &status, 0);  // child_pid = pid3 ← overwrites again
        
        printf("%d", child_pid);  // Prints: pid3
    }
    
    return 0;
}
```

**ANSWER:** The program prints **pid3** (the last waitpid's return value)

---

## 📖 SECTION 5: PROCESS STATES & LIFECYCLE

### The 5 Process States

1. **NEW** - Process is being created
2. **READY** - Process is ready to run, waiting for CPU
3. **RUNNING** - Process is currently executing
4. **WAITING/BLOCKED** - Process waiting for I/O or event
5. **TERMINATED** - Process has finished execution

### Special States

**ZOMBIE Process:**
- Child has terminated (called exit())
- Parent has NOT called wait() yet
- Process table entry still exists
- Consumes NO CPU or memory (only table entry)
- Appears as `<defunct>` in ps output

**ORPHAN Process:**
- Parent terminated before child
- Child is "adopted" by init process (PID 1)
- init automatically reaps orphans (calls wait())

### Example: Creating a Zombie
```c
int main() {
    pid_t pid = fork();
    
    if(pid == 0) {
        // Child exits immediately
        exit(0);  // Child becomes ZOMBIE
    }
    else {
        // Parent does NOT call wait()
        sleep(30);  // Child is zombie for 30 seconds!
    }
    return 0;
}
```

### Example: Creating an Orphan
```c
int main() {
    pid_t pid = fork();
    
    if(pid == 0) {
        // Child sleeps
        sleep(30);
        printf("Child's parent PID: %d\n", getppid());  // Will be 1 (init)
    }
    else {
        // Parent exits immediately
        exit(0);  // Child becomes ORPHAN
    }
    return 0;
}
```

---

## 📖 SECTION 6: EXEC FAMILY - PROCESS REPLACEMENT

### What is exec()?
**exec()** replaces the current process's memory with a NEW program.

**Key Points:**
- Does NOT create a new process (same PID)
- Loads a new program into current process
- If successful, NEVER returns (old program is gone!)
- If fails, returns -1 and continues

### Common exec() Variants
```c
execl("/bin/ls", "ls", "-l", NULL);      // List arguments
execv("/bin/ls", argv);                   // Array of arguments
execvp("ls", argv);                       // Search in PATH
```

### Example: Fork + Exec Pattern
```c
int main() {
    pid_t pid = fork();
    
    if(pid == 0) {
        // CHILD: Replace with new program
        execl("/bin/ls", "ls", "-l", NULL);
        
        // If we reach here, exec FAILED!
        perror("exec failed");
        exit(1);
    }
    else {
        // PARENT: Wait for child
        wait(NULL);
        printf("Child finished\n");
    }
    
    return 0;
}
```

### What Happens After exec()?
```
BEFORE exec:
  PID: 12345
  Program: ./myprogram
  Memory: [myprogram's code and data]

AFTER exec("/bin/ls", ...):
  PID: 12345  ← SAME PID!
  Program: /bin/ls
  Memory: [ls's code and data]  ← COMPLETELY REPLACED!
```

**🔑 CRITICAL:**
- PID stays the same
- Open file descriptors are inherited (unless FD_CLOEXEC)
- Signal handlers are reset to default
- All threads except calling thread are destroyed

---

## 📖 SECTION 7: IMPORTANT SYSTEM CALLS

### getpid() and getppid()
```c
pid_t getpid();   // Returns current process's PID
pid_t getppid();  // Returns parent's PID

// Example:
printf("My PID: %d\n", getpid());
printf("My parent's PID: %d\n", getppid());
```

### exit() vs _exit()
```c
exit(0);    // Normal termination
            // - Flushes I/O buffers
            // - Calls atexit() handlers
            // - Closes file descriptors

_exit(0);   // Immediate termination
            // - NO buffer flushing
            // - NO atexit() handlers
            // - Direct system call
```

**When to use _exit():**
- In child after fork() if not doing exec()
- Prevents double-flushing of parent's buffers

---

## 🎯 TRUE/FALSE QUESTIONS - FORK & PROCESS CONTROL
### 80 Questions to Grab Easy Marks!

---

### CATEGORY 1: Basic Fork Concepts (Q1-20)

**Q1. T/F:** fork() creates a new process by duplicating the calling process.
**ANSWER: TRUE** ✓
**Explanation:** fork() makes an exact copy of the calling process.

**Q2. T/F:** After fork(), both parent and child start executing from the beginning of main().
**ANSWER: FALSE** ✗
**Explanation:** Both continue from the line AFTER fork().

**Q3. T/F:** fork() returns the same value in both parent and child.
**ANSWER: FALSE** ✗
**Explanation:** Returns child PID to parent, 0 to child.

**Q4. T/F:** In the parent process, fork() returns 0.
**ANSWER: FALSE** ✗
**Explanation:** Parent gets child's PID (positive number).

**Q5. T/F:** In the child process, fork() returns 0.
**ANSWER: TRUE** ✓
**Explanation:** Child always gets 0 from fork().

**Q6. T/F:** If fork() fails, it returns -1.
**ANSWER: TRUE** ✓
**Explanation:** -1 indicates error (no child created).

**Q7. T/F:** After fork(), both parent and child have the same PID.
**ANSWER: FALSE** ✗
**Explanation:** Child gets a NEW, unique PID.

**Q8. T/F:** getpid() returns the same value in parent and child after fork().
**ANSWER: FALSE** ✗
**Explanation:** Each has its own PID.

**Q9. T/F:** getppid() in child returns the parent's PID.
**ANSWER: TRUE** ✓
**Explanation:** getppid() returns parent's process ID.

**Q10. T/F:** After fork(), parent and child share the same memory space.
**ANSWER: FALSE** ✗
**Explanation:** Child gets a COPY, not shared memory.

**Q11. T/F:** Changes to variables in child affect parent's variables.
**ANSWER: FALSE** ✗
**Explanation:** Separate memory spaces (copy-on-write).

**Q12. T/F:** Both parent and child execute concurrently after fork().
**ANSWER: TRUE** ✓
**Explanation:** OS schedules both independently.

**Q13. T/F:** The order of execution between parent and child is guaranteed.
**ANSWER: FALSE** ✗
**Explanation:** Order is unpredictable (OS scheduler).

**Q14. T/F:** fork() is a system call.
**ANSWER: TRUE** ✓
**Explanation:** It's a kernel operation.

**Q15. T/F:** fork() creates a new thread, not a new process.
**ANSWER: FALSE** ✗
**Explanation:** Creates a full process, not just a thread.

**Q16. T/F:** A process can call fork() multiple times.
**ANSWER: TRUE** ✓
**Explanation:** No limit on fork() calls (except system resources).

**Q17. T/F:** After fork(), the child inherits open file descriptors from parent.
**ANSWER: TRUE** ✓
**Explanation:** File descriptors are inherited.

**Q18. T/F:** After fork(), both processes point to the same file offset.
**ANSWER: TRUE** ✓
**Explanation:** They share the open file table entry.

**Q19. T/F:** fork() duplicates only the calling thread in a multi-threaded program.
**ANSWER: TRUE** ✓
**Explanation:** Other threads are not copied to child.

**Q20. T/F:** fork() is faster than creating a new thread.
**ANSWER: FALSE** ✗
**Explanation:** fork() is heavier (copies entire process).

---

### CATEGORY 2: Process Counting (Q21-35)

**Q21. T/F:** Three sequential fork() calls create 8 processes total.
**ANSWER: TRUE** ✓
**Explanation:** 2^3 = 8 processes.

**Q22. T/F:** A loop with 5 fork() calls creates 5 processes.
**ANSWER: FALSE** ✗
**Explanation:** Creates 2^5 = 32 processes.

**Q23. T/F:** fork(); fork(); creates 3 processes.
**ANSWER: FALSE** ✗
**Explanation:** Creates 2^2 = 4 processes.

**Q24. T/F:** for(i=0; i<4; i++) fork(); creates 16 processes.
**ANSWER: TRUE** ✓
**Explanation:** 2^4 = 16 processes.

**Q25. T/F:** if(fork()) fork(); creates 6 processes.
**ANSWER: FALSE** ✗
**Explanation:** Creates 3 processes (parent + child + parent's 2nd child).

**Q26. T/F:** In if(fork()) fork();, the child from the first fork also forks.
**ANSWER: FALSE** ✗
**Explanation:** Child gets 0, so if condition is false, child doesn't fork.

**Q27. T/F:** Four sequential forks create 16 processes.
**ANSWER: TRUE** ✓
**Explanation:** 2^4 = 16.

**Q28. T/F:** In a loop, if one process exits before forking again, it doesn't affect the total count.
**ANSWER: FALSE** ✗
**Explanation:** Dead processes don't fork, reducing total.

**Q29. T/F:** fork(); fork(); fork(); if(fork()) fork(); creates 32 processes.
**ANSWER: FALSE** ✗
**Explanation:** 8 * 3 = 24 processes (8 from first 3, then conditional adds 16 more... actually complex, but answer is not 32).

**Q30. T/F:** The formula for n sequential forks is 2^n.
**ANSWER: TRUE** ✓
**Explanation:** Each fork doubles the processes.

**Q31. T/F:** After 10 sequential forks, there are 1024 processes.
**ANSWER: TRUE** ✓
**Explanation:** 2^10 = 1024.

**Q32. T/F:** In for(i=0; i<n; i++) fork();, exactly n new processes are created.
**ANSWER: FALSE** ✗
**Explanation:** 2^n total processes, so 2^n - 1 new processes.

**Q33. T/F:** The original parent process can be identified by checking if all fork() return values are > 0.
**ANSWER: TRUE** ✓
**Explanation:** Only original parent has all positive returns.

**Q34. T/F:** In three sequential forks, 7 new processes are created.
**ANSWER: TRUE** ✓
**Explanation:** 2^3 = 8 total, minus 1 original = 7 new.

**Q35. T/F:** if(fork() && fork()) fork(); creates more processes than if(fork() || fork()) fork();
**ANSWER: FALSE** ✗
**Explanation:** && and || short-circuit differently, complex calculation needed.

---

### CATEGORY 3: waitpid() and Synchronization (Q36-50)

**Q36. T/F:** waitpid() blocks until a specific child terminates.
**ANSWER: TRUE** ✓
**Explanation:** It waits for the specified PID.

**Q37. T/F:** waitpid() returns the child's PID when child terminates.
**ANSWER: TRUE** ✓
**Explanation:** Returns PID of terminated child.

**Q38. T/F:** wait() and waitpid(-1, NULL, 0) are equivalent.
**ANSWER: TRUE** ✓
**Explanation:** Both wait for any child.

**Q39. T/F:** If waitpid is called multiple times on the same variable, only the last return value is stored.
**ANSWER: TRUE** ✓
**Explanation:** Variable overwriting - EXAM FAVORITE!

**Q40. T/F:** waitpid() can wait for a grandchild process.
**ANSWER: FALSE** ✗
**Explanation:** Only direct children.

**Q41. T/F:** If a parent doesn't call wait(), the child becomes a zombie.
**ANSWER: TRUE** ✓
**Explanation:** Zombie until parent reaps it.

**Q42. T/F:** Zombie processes consume CPU resources.
**ANSWER: FALSE** ✗
**Explanation:** Only occupy process table entry.

**Q43. T/F:** Zombie processes consume memory.
**ANSWER: FALSE** ✗
**Explanation:** Memory is freed, only table entry remains.

**Q44. T/F:** Orphan processes are adopted by init (PID 1).
**ANSWER: TRUE** ✓
**Explanation:** init becomes the new parent.

**Q45. T/F:** A parent can call waitpid() on an already-terminated child.
**ANSWER: TRUE** ✓
**Explanation:** Reaps zombie immediately.

**Q46. T/F:** WNOHANG option makes waitpid() non-blocking.
**ANSWER: TRUE** ✓
**Explanation:** Returns immediately if child hasn't terminated.

**Q47. T/F:** waitpid(0, NULL, 0) waits for any child in the same process group.
**ANSWER: TRUE** ✓
**Explanation:** pid=0 means same group.

**Q48. T/F:** Multiple children can be reaped with a single wait() call.
**ANSWER: FALSE** ✗
**Explanation:** One wait() reaps one child.

**Q49. T/F:** If all children terminate before parent calls wait(), wait() blocks forever.
**ANSWER: FALSE** ✗
**Explanation:** Returns immediately for zombies.

**Q50. T/F:** Zombie processes can be killed with SIGKILL.
**ANSWER: FALSE** ✗
**Explanation:** Already dead, need parent to call wait().

---

### CATEGORY 4: exec() Family (Q51-60)

**Q51. T/F:** exec() creates a new process.
**ANSWER: FALSE** ✗
**Explanation:** Replaces current process image, same PID.

**Q52. T/F:** After successful exec(), code following it executes.
**ANSWER: FALSE** ✗
**Explanation:** exec() never returns on success.

**Q53. T/F:** If exec() fails, it returns -1.
**ANSWER: TRUE** ✓
**Explanation:** Returns -1 and continues with old program.

**Q54. T/F:** exec() is commonly used after fork() in the child process.
**ANSWER: TRUE** ✓
**Explanation:** Classic fork-exec pattern.

**Q55. T/F:** The PID changes after exec().
**ANSWER: FALSE** ✗
**Explanation:** Same PID, different program.

**Q56. T/F:** Open file descriptors are closed after exec().
**ANSWER: FALSE** ✗
**Explanation:** Inherited unless FD_CLOEXEC set.

**Q57. T/F:** execvp() searches for the program in PATH environment variable.
**ANSWER: TRUE** ✓
**Explanation:** The 'p' means PATH search.

**Q58. T/F:** Signal handlers are preserved after exec().
**ANSWER: FALSE** ✗
**Explanation:** Reset to default.

**Q59. T/F:** execl() and execv() differ only in how arguments are passed.
**ANSWER: TRUE** ✓
**Explanation:** 'l' = list, 'v' = vector (array).

**Q60. T/F:** A program can exec() itself, creating an infinite loop.
**ANSWER: TRUE** ✓
**Explanation:** Possible but usually unintended.

---

### CATEGORY 5: Process States & Advanced Concepts (Q61-80)

**Q61. T/F:** A zombie process is one that has terminated but hasn't been reaped.
**ANSWER: TRUE** ✓
**Explanation:** Definition of zombie.

**Q62. T/F:** Orphan processes have no parent.
**ANSWER: FALSE** ✗
**Explanation:** Adopted by init, so they have a parent (init).

**Q63. T/F:** init process has PID 1.
**ANSWER: TRUE** ✓
**Explanation:** First process started by kernel.

**Q64. T/F:** exit() and _exit() are identical.
**ANSWER: FALSE** ✗
**Explanation:** exit() flushes buffers, _exit() doesn't.

**Q65. T/F:** _exit() should be used in child after fork() if not calling exec().
**ANSWER: TRUE** ✓
**Explanation:** Prevents double-flushing parent's buffers.

**Q66. T/F:** A process can have multiple parents.
**ANSWER: FALSE** ✗
**Explanation:** Each process has exactly one parent.

**Q67. T/F:** getppid() always returns a positive number.
**ANSWER: TRUE** ✓
**Explanation:** Parent PID is always positive (at least 1 for init).

**Q68. T/F:** After parent dies, child's getppid() returns 1.
**ANSWER: TRUE** ✓
**Explanation:** Orphans are adopted by init (PID 1).

**Q69. T/F:** A process can change its own PID.
**ANSWER: FALSE** ✗
**Explanation:** PIDs are assigned by kernel, immutable.

**Q70. T/F:** fork() can fail if system runs out of resources.
**ANSWER: TRUE** ✓
**Explanation:** Returns -1 if insufficient memory or process limit reached.

**Q71. T/F:** The maximum number of processes is unlimited.
**ANSWER: FALSE** ✗
**Explanation:** Limited by system resources and limits.

**Q72. T/F:** A child process can outlive its parent.
**ANSWER: TRUE** ✓
**Explanation:** Becomes orphan, adopted by init.

**Q73. T/F:** init process can never terminate.
**ANSWER: TRUE** ✓
**Explanation:** System would crash if init dies.

**Q74. T/F:** All processes are descendants of init.
**ANSWER: TRUE** ✓
**Explanation:** Init is the ancestor of all user processes.

**Q75. T/F:** A running process is always consuming CPU.
**ANSWER: FALSE** ✗
**Explanation:** May be blocked/waiting for I/O.

**Q76. T/F:** A blocked process cannot receive signals.
**ANSWER: FALSE** ✗
**Explanation:** Can receive signals even when blocked.

**Q77. T/F:** vfork() is safer than fork().
**ANSWER: FALSE** ✗
**Explanation:** vfork() is more dangerous (shares memory with parent).

**Q78. T/F:** In a process tree, all leaf nodes are the most recently created processes.
**ANSWER: FALSE** ✗
**Explanation:** Leaf nodes could be old processes that didn't fork.

**Q79. T/F:** The shell uses fork() and exec() to run commands.
**ANSWER: TRUE** ✓
**Explanation:** Shell forks, child execs the command.

**Q80. T/F:** A process ID (PID) can be reused after a process terminates.
**ANSWER: TRUE** ✓
**Explanation:** PIDs are recycled by the system.

---

## 🔥 EXAM STRATEGY FOR FORK QUESTIONS

### Step 1: Identify the Pattern
- **Sequential forks?** → Use 2^n formula
- **Loop forks?** → Use 2^n formula  
- **Conditional forks?** → Draw process tree
- **waitpid() chain?** → Remember last value wins!

### Step 2: Draw Process Tree (if needed)
- Start with 1 process
- After each fork, split branches
- Label return values (parent: PID, child: 0)

### Step 3: Track Variables
- Check which processes enter which if/else blocks
- Remember: separate memory after fork

### Step 4: Apply Variable Overwriting Rule
- Multiple assignments to same variable?
- LAST assignment is the final value!

---

## 📝 QUICK REFERENCE - MEMORIZE THIS!

```
FORK RETURN VALUES:
Parent: child PID (positive)
Child:  0
Error:  -1

PROCESS COUNTING:
n sequential forks = 2^n processes
fork in loop n times = 2^n processes

WAITPID PATTERN:
result = waitpid(pid1, ...);
result = waitpid(pid2, ...);  ← OVERWRITES
result = waitpid(pid3, ...);  ← OVERWRITES AGAIN
// result has LAST value (pid3)

PROCESS STATES:
New → Ready → Running → Waiting → Terminated
Zombie: terminated but not reaped
Orphan: parent died, adopted by init

EXEC:
- Replaces process image
- Same PID
- Never returns on success
- Returns -1 on failure
```

---

## ✅ CHECKLIST - DID YOU MASTER THIS?

- □ I can calculate process count from any fork pattern
- □ I understand fork return values (parent vs child)
- □ I know the variable overwriting rule for waitpid
- □ I can draw process trees
- □ I know difference between zombie and orphan
- □ I understand exec replaces, not creates
- □ I got 75%+ on the True/False questions

---

## 🎯 NEXT STEP

**Move to PART 2: FILE I/O & FILE DESCRIPTORS**

You've mastered Fork! Time to conquer File I/O!

---

**END OF PART 1/9**

**Progress: [████░░░░░] 11% Complete**

**Keep going! You're building a champion's knowledge! 💪🔥**
