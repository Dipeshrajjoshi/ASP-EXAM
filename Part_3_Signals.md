# 🏆 COMP8567 ULTIMATE EXAM GUIDE - PART 3/9
## SIGNALS & SIGNAL HANDLING
## Master Signals and Dominate Process Group Questions!

---

# 📚 PART 3: SIGNALS & SIGNAL HANDLING

## 🎯 WHAT YOU'LL MASTER IN THIS SECTION:
- ✅ What signals are and how they work
- ✅ Common signals (SIGINT, SIGALRM, SIGKILL, etc.)
- ✅ signal() and sigaction() for handler registration
- ✅ alarm() - THE ONE-SHOT TIMER (Exam Killer!)
- ✅ Process groups and signal delivery
- ✅ kill() for sending signals
- ✅ Signal masks and blocking
- ✅ 75+ True/False questions for guaranteed marks!

---

## 📖 SECTION 1: WHAT ARE SIGNALS?

### Definition
**Signals** are software interrupts that notify a process that an event has occurred.

**Key Points:**
- Asynchronous notification mechanism
- Can be sent by:
  - Kernel (e.g., divide by zero, illegal instruction)
  - Another process (via kill())
  - The process itself (via raise() or alarm())
  - Terminal (Ctrl-C, Ctrl-Z)

### Signal Lifecycle
```
1. Signal is GENERATED (sent)
2. Signal is PENDING (queued)
3. Signal is DELIVERED (process handles it)
4. Signal is HANDLED (default action, ignore, or custom handler)
```

---

## 📖 SECTION 2: COMMON SIGNALS - MEMORIZE THESE!

### Signal Table
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Signal    | Number | Default Action | Catchable? | Description
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SIGHUP    | 1      | Terminate      | YES        | Hangup
SIGINT    | 2      | Terminate      | YES        | Interrupt (Ctrl-C)
SIGQUIT   | 3      | Terminate+Core | YES        | Quit (Ctrl-\)
SIGILL    | 4      | Terminate+Core | YES        | Illegal instruction
SIGTRAP   | 5      | Terminate+Core | YES        | Trace trap
SIGABRT   | 6      | Terminate+Core | YES        | Abort
SIGKILL   | 9      | Terminate      | NO ❌      | Kill (uncatchable!)
SIGSEGV   | 11     | Terminate+Core | YES        | Segmentation fault
SIGPIPE   | 13     | Terminate      | YES        | Broken pipe
SIGALRM   | 14     | Terminate      | YES        | Alarm clock
SIGTERM   | 15     | Terminate      | YES        | Termination request
SIGCHLD   | 17     | Ignore         | YES        | Child stopped/terminated
SIGCONT   | 18     | Continue       | YES        | Continue if stopped
SIGSTOP   | 19     | Stop           | NO ❌      | Stop (uncatchable!)
SIGTSTP   | 20     | Stop           | YES        | Stop (Ctrl-Z)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### The Two Uncatchable Signals
**SIGKILL (9)** and **SIGSTOP (19)** CANNOT be caught, blocked, or ignored!

**Why?** System needs a guaranteed way to kill or stop processes.

---

## 📖 SECTION 3: SIGNAL() - REGISTERING HANDLERS

### Syntax
```c
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);

// Returns: previous handler, or SIG_ERR on error
```

### Handler Options
```c
SIG_DFL    // Default action
SIG_IGN    // Ignore the signal

// Or custom function:
void my_handler(int signo) {
    // Handle signal
}
```

### Example: Catching SIGINT (Ctrl-C)
```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void handler(int signo) {
    printf("\nCaught SIGINT! Can't stop me!\n");
    signal(SIGINT, handler);  // Re-register (for older systems)
}

int main() {
    signal(SIGINT, handler);
    
    while(1) {
        printf("Running... (Press Ctrl-C)\n");
        sleep(1);
    }
    
    return 0;
}
```

**Output when Ctrl-C pressed:**
```
Running...
Running...
^C
Caught SIGINT! Can't stop me!
Running...
Running...
```

---

## 📖 SECTION 4: ALARM() - THE EXAM KILLER! 🎯

### What is alarm()?
**alarm()** sets a ONE-SHOT timer. After N seconds, SIGALRM is sent to the process.

### Syntax
```c
unsigned int alarm(unsigned int seconds);

// Returns: seconds remaining from previous alarm, or 0
```

### CRITICAL RULES - MEMORIZE THESE!

**RULE 1:** alarm() is ONE-SHOT (fires once)
**RULE 2:** To repeat, handler must call alarm() again
**RULE 3:** Only ONE alarm active at a time
**RULE 4:** New alarm() cancels previous alarm
**RULE 5:** alarm(0) cancels the alarm

---

## 📖 SECTION 5: THE T1 ALARM QUESTION - STEP BY STEP

### The Question
```c
void handler1(int signo) {
    signal(SIGALRM, handler1);  // Re-register handler
    printf("\nIn the handler\n");
    // NOTE: Does NOT call alarm() again!
}

int main() {
    signal(SIGALRM, handler1);  // Register handler
    alarm(8);                    // Set alarm for 8 seconds
    
    for(int i = 0; i < 10; i++)
        sleep(1);  // Sleep 10 seconds total
    
    return 0;
}

// How many times does the handler execute?
```

### The Complete Trace

```
═══════════════════════════════════════════════════════════

TIME: 0 seconds
  Action: signal(SIGALRM, handler1) registered
  Action: alarm(8) starts countdown
  Status: Timer running, 8 seconds remaining

───────────────────────────────────────────────────────────

TIME: 1 second (i=0, sleep 1s)
  Status: Timer running, 7 seconds remaining

───────────────────────────────────────────────────────────

TIME: 2 seconds (i=1, sleep 1s)
  Status: Timer running, 6 seconds remaining

───────────────────────────────────────────────────────────

TIME: 3 seconds (i=2, sleep 1s)
  Status: Timer running, 5 seconds remaining

───────────────────────────────────────────────────────────

TIME: 4 seconds (i=3, sleep 1s)
  Status: Timer running, 4 seconds remaining

───────────────────────────────────────────────────────────

TIME: 5 seconds (i=4, sleep 1s)
  Status: Timer running, 3 seconds remaining

───────────────────────────────────────────────────────────

TIME: 6 seconds (i=5, sleep 1s)
  Status: Timer running, 2 seconds remaining

───────────────────────────────────────────────────────────

TIME: 7 seconds (i=6, sleep 1s)
  Status: Timer running, 1 second remaining

───────────────────────────────────────────────────────────

TIME: 8 seconds (i=7, sleep 1s)
  ⚡ SIGALRM DELIVERED!
  Action: handler1() executes
  Output: "\nIn the handler\n"
  Action: signal(SIGALRM, handler1) re-registers handler
  ⚠️ BUT: No alarm() call in handler!
  Status: NO TIMER SET!

───────────────────────────────────────────────────────────

TIME: 9 seconds (i=8, sleep 1s)
  Status: No timer, no signal

───────────────────────────────────────────────────────────

TIME: 10 seconds (i=9, sleep 1s)
  Status: No timer, no signal
  Action: Loop ends, program exits

═══════════════════════════════════════════════════════════

FINAL ANSWER: Handler executed ONCE (only 1 time)
```

### Why Only Once?

**The Trap:**
- Handler re-registers itself: `signal(SIGALRM, handler1);`
- But DOESN'T call `alarm()` again!
- alarm() is ONE-SHOT - fires once and stops!

**To make it repeat:**
```c
void handler1(int signo) {
    signal(SIGALRM, handler1);
    printf("\nIn the handler\n");
    alarm(8);  // ← ADD THIS for repeat!
}
```

---

## 📖 SECTION 6: PROCESS GROUPS - EXAM FAVORITE!

### What is a Process Group?
A **process group** is a collection of one or more processes.

**Key Points:**
- Every process belongs to a process group
- Process Group ID (PGID) identifies the group
- By default, child inherits parent's process group
- Can create new group with setpgid()

### Process Group Functions
```c
pid_t getpgrp(void);              // Get own PGID
int setpgid(pid_t pid, pid_t pgid);  // Set process group

// Common usage:
setpgid(0, 0);  // Create new group with self as leader
                // Same as: setpgid(getpid(), getpid());
```

### Why Process Groups Matter
**Ctrl-C (SIGINT) goes to FOREGROUND process group ONLY!**

This is the basis for MANY exam questions!

---

## 📖 SECTION 7: THE T2 PROCESS GROUP QUESTION

### The Question
```c
void handler() { exit(0); }

int main() {
    signal(SIGINT, handler);
    
    for(int i = 1; i < 4; i++)
        fork();  // Creates 8 processes
    
    if((pid = fork()) == 0) {  // All 8 do this fork
        // Child from each fork
        setpgid(0, getpid());  // Creates NEW process group
    }
    
    // Ctrl-C pressed (SIGINT sent)
    
    // How many processes survive?
}
```

### The Complete Trace

```
═══════════════════════════════════════════════════════════

STEP 1: for(int i=1; i<4; i++) fork();

Iteration 1 (i=1): fork()
  Processes: 1 → 2

Iteration 2 (i=2): fork() [both processes do it]
  Processes: 2 → 4

Iteration 3 (i=3): fork() [all 4 do it]
  Processes: 4 → 8

Total after loop: 8 processes
All in SAME process group (parent's group)

───────────────────────────────────────────────────────────

STEP 2: if((pid=fork()) == 0) { setpgid(0, getpid()); }

ALL 8 processes execute this fork!

Each fork creates:
  - Parent: pid > 0, stays in original group
  - Child: pid = 0, executes setpgid(0, getpid())
           Creates NEW process group!

Result:
  8 parents in ORIGINAL group (foreground)
  8 children in NEW groups (each in own group)

Total: 16 processes

───────────────────────────────────────────────────────────

STEP 3: Ctrl-C pressed (SIGINT sent)

SIGINT is sent to FOREGROUND process group ONLY!

Foreground group = original group = 8 parents

8 parents:
  - Receive SIGINT
  - handler() executes
  - exit(0) called
  - DIE ☠️

8 children in different groups:
  - Do NOT receive SIGINT (not in foreground group)
  - SURVIVE ✓

═══════════════════════════════════════════════════════════

FINAL ANSWER: 8 processes survive
```

### Key Insight
**Ctrl-C (SIGINT) only affects FOREGROUND process group!**
Processes in other groups are NOT affected!

---

## 📖 SECTION 8: KILL() - SENDING SIGNALS

### Syntax
```c
int kill(pid_t pid, int sig);

// Returns: 0 on success, -1 on error
```

### pid Arguments
```c
pid > 0    // Send to specific process
pid == 0   // Send to all in same process group
pid == -1  // Send to all processes (broadcast)
pid < -1   // Send to process group |pid|
```

### Examples
```c
// Send SIGTERM to process 1234
kill(1234, SIGTERM);

// Send SIGINT to all in same group
kill(0, SIGINT);

// Send SIGKILL to process 5678 (can't be caught!)
kill(5678, SIGKILL);

// Check if process exists (signal 0 doesn't send anything)
if(kill(pid, 0) == 0)
    printf("Process exists\n");
```

### Job Control Signals
```c
kill(pid, SIGSTOP);   // Stop process (Ctrl-Z equivalent)
kill(pid, SIGCONT);   // Continue stopped process (fg equivalent)

// Stop yourself!
kill(getpid(), SIGSTOP);
// or
kill(0, SIGSTOP);  // Stop all in process group
```

---

## 📖 SECTION 9: SIGNAL DELIVERY AND HANDLING

### Default Actions
Each signal has a default action:
- **Term**: Terminate the process
- **Ign**: Ignore the signal
- **Core**: Terminate and dump core
- **Stop**: Stop the process
- **Cont**: Continue if stopped

### Disposition Options
```c
// 1. Default action
signal(SIGINT, SIG_DFL);

// 2. Ignore signal
signal(SIGINT, SIG_IGN);

// 3. Custom handler
void handler(int sig) { /* ... */ }
signal(SIGINT, handler);
```

### Handler Restrictions
**DO NOT do these in signal handlers:**
- Call non-reentrant functions (printf, malloc, etc.)
- Access global data without protection
- Call longjmp()

**SAFE to do:**
- Set a global flag (volatile sig_atomic_t)
- Call async-signal-safe functions

---

## 📖 SECTION 10: SIGNAL MASKS AND BLOCKING

### What is a Signal Mask?
Set of signals currently **blocked** from delivery.

### Blocking Functions
```c
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

// how can be:
SIG_BLOCK    // Add signals to mask
SIG_UNBLOCK  // Remove signals from mask
SIG_SETMASK  // Replace entire mask
```

### Signal Set Operations
```c
sigset_t set;

sigemptyset(&set);      // Clear set
sigfillset(&set);       // Add all signals
sigaddset(&set, SIGINT);   // Add SIGINT
sigdelset(&set, SIGINT);   // Remove SIGINT
sigismember(&set, SIGINT); // Check membership
```

### Example: Blocking SIGINT
```c
sigset_t set;
sigemptyset(&set);
sigaddset(&set, SIGINT);

// Block SIGINT
sigprocmask(SIG_BLOCK, &set, NULL);

// ... do critical work ...

// Unblock SIGINT
sigprocmask(SIG_UNBLOCK, &set, NULL);
```

---

## 🎯 TRUE/FALSE QUESTIONS - SIGNALS
### 75 Questions to Master Signals!

---

### CATEGORY 1: Signal Basics (Q1-20)

**Q1. T/F:** Signals are software interrupts.
**ANSWER: TRUE** ✓

**Q2. T/F:** Signals are delivered synchronously.
**ANSWER: FALSE** ✗
**Explanation:** Asynchronous (can arrive anytime)

**Q3. T/F:** SIGINT is sent when user presses Ctrl-C.
**ANSWER: TRUE** ✓

**Q4. T/F:** SIGINT has signal number 2.
**ANSWER: TRUE** ✓

**Q5. T/F:** SIGKILL can be caught or ignored.
**ANSWER: FALSE** ✗
**Explanation:** SIGKILL (9) is uncatchable!

**Q6. T/F:** SIGSTOP can be caught or ignored.
**ANSWER: FALSE** ✗
**Explanation:** SIGSTOP (19) is uncatchable!

**Q7. T/F:** SIGTERM is catchable.
**ANSWER: TRUE** ✓

**Q8. T/F:** SIGALRM is sent by alarm() timer.
**ANSWER: TRUE** ✓

**Q9. T/F:** SIGCHLD is sent when a child process terminates.
**ANSWER: TRUE** ✓

**Q10. T/F:** SIGSEGV is sent on segmentation fault.
**ANSWER: TRUE** ✓

**Q11. T/F:** The default action for SIGINT is to terminate the process.
**ANSWER: TRUE** ✓

**Q12. T/F:** The default action for SIGCHLD is to ignore it.
**ANSWER: TRUE** ✓

**Q13. T/F:** A process can send a signal to itself.
**ANSWER: TRUE** ✓
**Explanation:** Using raise() or kill(getpid(), sig)

**Q14. T/F:** Only the kernel can send signals.
**ANSWER: FALSE** ✗
**Explanation:** Processes can send signals too (kill)

**Q15. T/F:** A signal can be pending (queued) before delivery.
**ANSWER: TRUE** ✓

**Q16. T/F:** Multiple identical signals can be queued.
**ANSWER: FALSE** ✗
**Explanation:** Standard signals don't queue (real-time do)

**Q17. T/F:** SIGKILL has signal number 9.
**ANSWER: TRUE** ✓

**Q18. T/F:** SIGSTOP has signal number 19.
**ANSWER: TRUE** ✓

**Q19. T/F:** SIGALRM has signal number 14.
**ANSWER: TRUE** ✓

**Q20. T/F:** Pressing Ctrl-Z sends SIGSTOP.
**ANSWER: FALSE** ✗
**Explanation:** Ctrl-Z sends SIGTSTP (20), which IS catchable

---

### CATEGORY 2: signal() Function (Q21-35)

**Q21. T/F:** signal() registers a signal handler.
**ANSWER: TRUE** ✓

**Q22. T/F:** signal() returns the previous handler.
**ANSWER: TRUE** ✓

**Q23. T/F:** SIG_DFL specifies the default action.
**ANSWER: TRUE** ✓

**Q24. T/F:** SIG_IGN makes the process ignore the signal.
**ANSWER: TRUE** ✓

**Q25. T/F:** A handler function must have signature: void handler(int).
**ANSWER: TRUE** ✓

**Q26. T/F:** signal(SIGKILL, handler) successfully registers a handler.
**ANSWER: FALSE** ✗
**Explanation:** Can't catch SIGKILL!

**Q27. T/F:** signal(SIGSTOP, handler) successfully registers a handler.
**ANSWER: FALSE** ✗
**Explanation:** Can't catch SIGSTOP!

**Q28. T/F:** After catching a signal, the handler may need re-registration on some systems.
**ANSWER: TRUE** ✓
**Explanation:** Old Unix behavior (not POSIX)

**Q29. T/F:** A handler can call signal() to re-register itself.
**ANSWER: TRUE** ✓

**Q30. T/F:** It's safe to call printf() in a signal handler.
**ANSWER: FALSE** ✗
**Explanation:** printf is not async-signal-safe!

**Q31. T/F:** A signal handler should be kept short and simple.
**ANSWER: TRUE** ✓

**Q32. T/F:** signal() can be used to ignore a signal.
**ANSWER: TRUE** ✓
**Explanation:** signal(SIG, SIG_IGN)

**Q33. T/F:** signal() can restore default behavior.
**ANSWER: TRUE** ✓
**Explanation:** signal(SIG, SIG_DFL)

**Q34. T/F:** Multiple signals can share the same handler function.
**ANSWER: TRUE** ✓

**Q35. T/F:** The handler parameter receives the signal number.
**ANSWER: TRUE** ✓

---

### CATEGORY 3: alarm() - THE KILLER TOPIC! (Q36-50)

**Q36. T/F:** alarm() sets a timer that sends SIGALRM.
**ANSWER: TRUE** ✓

**Q37. T/F:** alarm() is a repeating timer.
**ANSWER: FALSE** ✗
**Explanation:** ONE-SHOT! Fires once then stops!

**Q38. T/F:** To repeat alarm, handler must call alarm() again.
**ANSWER: TRUE** ✓
**Explanation:** CRITICAL! This is THE exam trap!

**Q39. T/F:** Only one alarm can be active at a time per process.
**ANSWER: TRUE** ✓

**Q40. T/F:** alarm(0) cancels the current alarm.
**ANSWER: TRUE** ✓

**Q41. T/F:** A new alarm() call cancels the previous alarm.
**ANSWER: TRUE** ✓

**Q42. T/F:** alarm() returns the seconds remaining from previous alarm.
**ANSWER: TRUE** ✓

**Q43. T/F:** In the T1 question, handler executes only once.
**ANSWER: TRUE** ✓
**Explanation:** Handler doesn't call alarm() again!

**Q44. T/F:** In the T1 question, handler re-registers itself.
**ANSWER: TRUE** ✓
**Explanation:** But doesn't set new alarm!

**Q45. T/F:** alarm(5) followed immediately by alarm(10) results in alarm after 15 seconds.
**ANSWER: FALSE** ✗
**Explanation:** Second alarm REPLACES first (10 seconds total)

**Q46. T/F:** If a process exits before alarm expires, no SIGALRM is sent.
**ANSWER: TRUE** ✓

**Q47. T/F:** alarm() can be used for precise timing.
**ANSWER: FALSE** ✗
**Explanation:** Only second granularity, not precise

**Q48. T/F:** sleep() and alarm() can interfere with each other.
**ANSWER: TRUE** ✓
**Explanation:** Both may use SIGALRM internally

**Q49. T/F:** alarm() is inherited by child after fork().
**ANSWER: FALSE** ✗
**Explanation:** Child does NOT inherit pending alarms

**Q50. T/F:** alarm() timer continues across exec().
**ANSWER: FALSE** ✗
**Explanation:** exec() resets alarms

---

### CATEGORY 4: Process Groups (Q51-65)

**Q51. T/F:** Every process belongs to a process group.
**ANSWER: TRUE** ✓

**Q52. T/F:** By default, child inherits parent's process group.
**ANSWER: TRUE** ✓

**Q53. T/F:** setpgid(0, getpid()) creates a new process group.
**ANSWER: TRUE** ✓

**Q54. T/F:** setpgid(0, 0) is equivalent to setpgid(getpid(), getpid()).
**ANSWER: TRUE** ✓

**Q55. T/F:** Ctrl-C sends SIGINT to all processes in the system.
**ANSWER: FALSE** ✗
**Explanation:** Only foreground process group!

**Q56. T/F:** Ctrl-C sends SIGINT to the foreground process group only.
**ANSWER: TRUE** ✓
**Explanation:** KEY CONCEPT for T2 question!

**Q57. T/F:** Processes in different process groups don't receive Ctrl-C.
**ANSWER: TRUE** ✓
**Explanation:** If not in foreground group

**Q58. T/F:** In T2 question, children create new process groups.
**ANSWER: TRUE** ✓

**Q59. T/F:** In T2 question, children survive Ctrl-C.
**ANSWER: TRUE** ✓
**Explanation:** They're in different groups!

**Q60. T/F:** In T2 question, parents die from Ctrl-C.
**ANSWER: TRUE** ✓
**Explanation:** They're in foreground group

**Q61. T/F:** kill(0, SIGINT) sends signal to all processes in same group.
**ANSWER: TRUE** ✓

**Q62. T/F:** kill(-1, SIGINT) broadcasts to all processes.
**ANSWER: TRUE** ✓

**Q63. T/F:** A process can change another process's group.
**ANSWER: TRUE** ✓
**Explanation:** With setpgid(pid, pgid) if allowed

**Q64. T/F:** getpgrp() returns the process group ID.
**ANSWER: TRUE** ✓

**Q65. T/F:** Process group ID is always equal to process ID.
**ANSWER: FALSE** ✗
**Explanation:** PGID = PID only for group leader

---

### CATEGORY 5: kill() and Advanced Topics (Q66-75)

**Q66. T/F:** kill() can send any signal.
**ANSWER: TRUE** ✓

**Q67. T/F:** kill(pid, 0) checks if process exists without sending signal.
**ANSWER: TRUE** ✓

**Q68. T/F:** kill(getpid(), SIGINT) sends SIGINT to self.
**ANSWER: TRUE** ✓

**Q69. T/F:** raise(sig) is equivalent to kill(getpid(), sig).
**ANSWER: TRUE** ✓

**Q70. T/F:** A process can kill any other process.
**ANSWER: FALSE** ✗
**Explanation:** Needs permissions (same user or root)

**Q71. T/F:** Signal handlers are inherited across fork().
**ANSWER: TRUE** ✓

**Q72. T/F:** Signal handlers are preserved across exec().
**ANSWER: FALSE** ✗
**Explanation:** Reset to default after exec()

**Q73. T/F:** Blocked signals are inherited across fork().
**ANSWER: TRUE** ✓

**Q74. T/F:** A signal can interrupt a system call.
**ANSWER: TRUE** ✓
**Explanation:** System call may return EINTR

**Q75. T/F:** sigaction() is more reliable than signal().
**ANSWER: TRUE** ✓
**Explanation:** POSIX-compliant, more control

---

## 🔥 EXAM STRATEGY FOR SIGNAL QUESTIONS

### For alarm() Questions:
1. **Check if handler calls alarm() again**
   - If YES → repeating timer
   - If NO → fires once only (T1 pattern!)

2. **Count seconds carefully**
   - When does alarm fire?
   - How many times can it fire?

3. **Remember: alarm() is ONE-SHOT!**

### For Process Group Questions:
1. **Count total processes after all forks**
   - Use 2^n formula

2. **Identify which create new groups**
   - Look for setpgid(0, getpid())

3. **Determine foreground group**
   - Original parent's group usually

4. **Apply Ctrl-C rule**
   - SIGINT to foreground group only
   - Other groups survive!

---

## 📝 QUICK REFERENCE - MEMORIZE THIS!

```
COMMON SIGNALS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SIGINT (2)   - Ctrl-C (catchable)
SIGKILL (9)  - Force kill (NOT catchable!)
SIGALRM (14) - alarm() timer
SIGTERM (15) - Termination request
SIGCHLD (17) - Child terminated
SIGCONT (18) - Continue process
SIGSTOP (19) - Stop (NOT catchable!)

ALARM RULES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. alarm() is ONE-SHOT (fires once)
2. Handler must call alarm() again to repeat
3. Only one alarm active at a time
4. alarm(0) cancels alarm

PROCESS GROUPS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Ctrl-C → foreground group ONLY
• setpgid(0, getpid()) → new group
• Different groups survive Ctrl-C

SIGNAL FUNCTIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
signal(sig, handler) - Register handler
alarm(seconds)       - Set timer
kill(pid, sig)       - Send signal
raise(sig)           - Signal self
```

---

## ✅ CHECKLIST - DID YOU MASTER THIS?

- □ I know alarm() is ONE-SHOT
- □ I can trace alarm questions (like T1)
- □ I understand process groups
- □ I know Ctrl-C affects foreground group only
- □ I can solve T2 process group question
- □ I know SIGKILL and SIGSTOP are uncatchable
- □ I got 85%+ on the True/False questions

---

## 🎯 NEXT STEP

**Move to PART 4: PIPES & INTER-PROCESS COMMUNICATION**

You've mastered Signals! Time to conquer Pipes!

---

**END OF PART 3/9**

**Progress: [████████████░] 33% Complete**

**You're on fire! Keep pushing! 💪🔥**
