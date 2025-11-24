# COMP8567 EXAM STUDY PLAN - REALISTIC VERSION
## Target: 70%+ on MCQ/T-F Exam (Nov 28, 4 PM)
## Strategy: CODE TRACING MASTERY

---

## 📅 OVERVIEW: Time to Exam

**Today:** Sunday, November 24, 2025  
**Exam:** Thursday, November 28, 2025 at 4:00 PM  

**Total Study Time Available:** 33 hours
- **Sunday (today):** 4 hours
- **Monday:** 4 hours  
- **Tuesday:** 5 hours
- **Wednesday:** 10 hours
- **Thursday:** 10 hours (exam day - morning study)

---

## 🎯 PRIORITY SYSTEM

**HIGH PRIORITY** (Most marks, most complex):
- ✅ FORK - Process Trees
- ✅ FILE I/O - Position Tracking
- ✅ SIGNALS - Handler Execution

**MEDIUM PRIORITY**:
- ✅ PIPES - Byte Flow
- ✅ THREADS - Shared Variables

**LOWER PRIORITY** (Easier to trace):
- ✅ SHELL SCRIPTS - Line by Line
- ✅ POINTERS - Memory Layout

---

## 📆 DAY-BY-DAY SCHEDULE

---

### **SUNDAY, NOV 24 (TODAY) - 4 HOURS**
**Focus: FORK - The Foundation of Everything**

#### **Your 4-Hour Study Block**

**HOUR 1: Fork Tracing Template (60 min)**

**Study Material:**
- Open Process_Control_F25.pptx
- Focus on fork() sections, return values, process trees

**Create FORK TRACING TEMPLATE:**
```
═══════════════════════════════════════════
           FORK TRACING TEMPLATE
═══════════════════════════════════════════

Step 1: Draw initial state
        [Parent PID]

Step 2: After each fork(), split the tree
        Parent ──fork()──> Parent (gets child PID)
                       └──> Child (gets 0)

Step 3: Label return values
        Parent: pid = positive number (child's PID)
        Child:  pid = 0

Step 4: Track which processes execute which code
        Use if/else conditions to determine paths

Step 5: Count processes OR track variable values
        Final answer: _______

QUICK FORMULAS:
  n sequential forks = 2^n processes
  for(i=0; i<n; i++) fork() = 2^n processes
```

**HOUR 2: Master the T1 Fork Question (60 min)**

**The Question:**
```c
int pid1, pid2, pid3;
pid1 = fork();  // Creates 2 processes
pid2 = fork();  // Both fork again = 4 processes  
pid3 = fork();  // All 4 fork again = 8 processes

if(pid1>0 && pid2>0 && pid3>0){  // Only ORIGINAL parent has all 3 > 0
    int child_pid;
    child_pid = waitpid(pid1, &status, 0);  // child_pid = pid1 value
    child_pid = waitpid(pid2, &status, 0);  // child_pid = pid2 value (OVERWRITES!)
    child_pid = waitpid(pid3, &status, 0);  // child_pid = pid3 value (OVERWRITES AGAIN!)
    printf("%d", child_pid);  // Prints pid3 (LAST assignment wins)
}
```

**TRACE IT STEP BY STEP:**
```
After pid1 = fork():
  Process A (parent): pid1 = 100
  Process B (child):  pid1 = 0

After pid2 = fork() [BOTH do this]:
  Process A: pid1=100, pid2=101
  Process B: pid1=0,   pid2=102
  Process C: pid1=100, pid2=0
  Process D: pid1=0,   pid2=0

After pid3 = fork() [ALL FOUR do this]:
  → 8 processes total

Only Process A has ALL three > 0:
  pid1=100, pid2=101, pid3=103

In Process A only:
  child_pid = waitpid(pid1, ...) → child_pid = 100
  child_pid = waitpid(pid2, ...) → child_pid = 101  ← OVERWRITES
  child_pid = waitpid(pid3, ...) → child_pid = 103  ← OVERWRITES AGAIN
  
  printf("%d", child_pid) → Prints 103

ANSWER: pid3 ✓
```

**🔑 KEY INSIGHT: Variable gets overwritten each time. LAST assignment is the final value!**

**Practice this question 5 times. Goal: Trace it in under 3 minutes.**

---

**HOUR 3: Fork Practice Problems (60 min)**

**Problem Set** (Create and solve these):

1. **Simple Count:**
   ```c
   fork(); fork(); fork();
   // How many processes total?
   ```
   Answer: 2³ = 8 processes

2. **Loop Fork:**
   ```c
   for(int i=0; i<4; i++)
       fork();
   // How many processes?
   ```
   Answer: 2⁴ = 16 processes

3. **Conditional Fork:**
   ```c
   if(fork())
       fork();
   // How many processes?
   ```
   Answer: 6 processes
   ```
   Parent ─fork()─> Parent (pid>0, true) ─fork()─> Parent
                                                └──> Child
                └──> Child (pid=0, false, doesn't fork)
   Total: 3 original + 3 more = 6? No, let me retrace...
   Actually: Parent + Parent's 2nd child + Child = 3? 
   
   Correct trace:
   Start: 1 process
   After fork(): 2 processes (parent got pid>0, child got 0)
   Parent enters if (true): forks again → 2 more processes
   Child skips if (false): no fork
   Total: 1 original parent + 1 child + 1 parent's new child = 3
   ```

4. **Variable Tracking:**
   ```c
   int x = 5;
   if(fork() == 0){
       x = 10;
   }
   printf("%d", x);
   // What do both processes print?
   ```
   Parent prints: 5
   Child prints: 10

5. **Multiple waitpid:**
   ```c
   int a = fork();
   int b = fork();
   if(a>0 && b>0){
       int result;
       result = waitpid(a, NULL, 0);
       result = waitpid(b, NULL, 0);
       printf("%d", result);
   }
   // What prints?
   ```
   Answer: b (last waitpid's return value)

6-8: **Create 3 more variations** of the T1 question (change number of forks, variable names)

---

**HOUR 4: Create Fork Reference Sheet + PRINT (60 min)**

**Create this document:**
```
═══════════════════════════════════════════════════════
                   FORK REFERENCE SHEET
═══════════════════════════════════════════════════════

FORK RETURN VALUES:
  Parent process: Returns child's PID (positive number)
  Child process:  Returns 0
  Error:          Returns -1

PROCESS COUNTING FORMULAS:
  Sequential forks:
    fork(); fork(); fork(); → 2³ = 8 processes
    
  Loop forks:
    for(i=0; i<n; i++) fork(); → 2ⁿ processes
    
  Conditional fork:
    if(fork()) fork(); → 3 processes
    (parent forks twice, child doesn't enter if)

WAITPID FUNCTION:
  waitpid(pid, &status, 0)
  - Blocks until specific child (with that PID) exits
  - Returns: PID of the child that terminated
  - Parent uses this to wait for children

TRACING CHECKLIST:
  □ Step 1: Draw process tree
  □ Step 2: Label fork() return value for EACH process
  □ Step 3: Identify which processes execute which code blocks
  □ Step 4: Track variable values (watch for overwrites!)
  □ Step 5: Count processes OR calculate final value

CRITICAL RULES TO REMEMBER:
  ✓ Each fork() DOUBLES the number of processes
  ✓ Child always gets 0, parent gets child's PID
  ✓ Each process has its own copy of variables (after fork)
  ✓ When same variable assigned multiple times:
    LAST assignment is the final value!
    
    Example:
    x = 100;
    x = 200;  // x is now 200
    x = 300;  // x is now 300 ← THIS is the final value

COMMON MISTAKES TO AVOID:
  ✗ Forgetting that ALL processes execute code after fork
  ✗ Not tracking which process enters which if/else branch
  ✗ Forgetting variable overwriting rule
  ✗ Miscounting processes (use formula!)
```

**🖨️ PRINT THIS SHEET IMMEDIATELY**
**ORGANIZE:** Put this in a folder/binder for exam day

---

### **MONDAY, NOV 25 - 4 HOURS**
**Focus: FILE I/O - Position Tracking Mastery**

#### **Your 4-Hour Study Block**

**HOUR 1: File I/O Tracing Template (60 min)**

**Study Material:**
- Open Unix_File_Input_Output_F25.pptx
- Focus on: lseek(), read(), write(), file descriptors

**Create FILE I/O TRACING TEMPLATE:**
```
═══════════════════════════════════════════
        FILE I/O TRACING TEMPLATE
═══════════════════════════════════════════

Initial state:
  File size: _____ bytes
  Position: 0

Operation 1: _____________________
  Calculation: _____________________
  New Position: _____ 
  New File Size: _____

Operation 2: _____________________
  Calculation: _____________________
  New Position: _____ 
  New File Size: _____

[Continue for all operations...]

Final File Size: _____ bytes

THE THREE LSEEK FORMULAS:
  SEEK_SET (0): new_pos = offset
  SEEK_CUR (1): new_pos = current_pos + offset
  SEEK_END (2): new_pos = file_size + offset

⚠️ CRITICAL: If new_pos < 0, lseek FAILS!
   Failed lseek → position DOESN'T change
```

**HOUR 2: Master the T1 File I/O Question (60 min)**

**The Question:**
```c
int fd = open("check.txt", O_RDWR);  // Initial file size = 40 bytes
lseek(fd, 100, SEEK_END);
write(fd, buf, 9);
write(fd, buf, 9);
lseek(fd, -159, SEEK_CUR);
write(fd, buf, 9);
write(fd, buf, 9);
close(fd);
// What is the final file size?
```

**TRACE IT STEP BY STEP:**
```
═══════════════════════════════════════════

Initial State:
  File size: 40 bytes
  Position: 0

─────────────────────────────────────────────

Operation 1: lseek(fd, 100, SEEK_END)
  Formula: SEEK_END → new_pos = file_size + offset
  Calculation: new_pos = 40 + 100 = 140
  New Position: 140
  New File Size: 40 (lseek doesn't change size!)

─────────────────────────────────────────────

Operation 2: write(fd, buf, 9)
  Writes 9 bytes at position 140
  New Position: 140 + 9 = 149
  New File Size: 149 (file grew! write beyond end)

─────────────────────────────────────────────

Operation 3: write(fd, buf, 9)
  Writes 9 bytes at position 149
  New Position: 149 + 9 = 158
  New File Size: 158

─────────────────────────────────────────────

Operation 4: lseek(fd, -159, SEEK_CUR)
  Formula: SEEK_CUR → new_pos = current_pos + offset
  Calculation: new_pos = 158 + (-159) = -1
  ⚠️ NEGATIVE POSITION! lseek FAILS!
  Failed lseek returns -1
  New Position: 158 (DOESN'T CHANGE!)
  New File Size: 158 (no change)

─────────────────────────────────────────────

Operation 5: write(fd, buf, 9)
  Writes 9 bytes at position 158 (still there!)
  New Position: 158 + 9 = 167
  New File Size: 167

─────────────────────────────────────────────

Operation 6: write(fd, buf, 9)
  Writes 9 bytes at position 167
  New Position: 167 + 9 = 176
  New File Size: 176

═══════════════════════════════════════════

FINAL FILE SIZE: 176 bytes ✓
```

**🔑 KEY INSIGHTS:**
1. **lseek doesn't change file size**, only position
2. **write changes both** position and size (if writing beyond end)
3. **Failed lseek (negative position) doesn't change position**
4. **File size = furthest byte written**

**Practice this question 4 times. Goal: Trace it in under 4 minutes.**

---

**HOUR 3: File I/O Practice Problems (60 min)**

**Problem Set:**

1. **Simple SEEK_SET:**
   ```c
   fd = open("file.txt", O_RDWR);  // size = 50
   lseek(fd, 10, SEEK_SET);
   write(fd, buf, 5);
   // Final size? Position?
   ```
   Position: 10 + 5 = 15
   Size: 50 (wrote within file)

2. **SEEK_END Extension:**
   ```c
   fd = open("file.txt", O_RDWR);  // size = 30
   lseek(fd, 20, SEEK_END);
   write(fd, buf, 10);
   // Final size?
   ```
   Position: 30 + 20 = 50
   After write: 50 + 10 = 60
   Size: 60

3. **Failed lseek:**
   ```c
   fd = open("file.txt", O_RDWR);  // size = 100
   lseek(fd, 50, SEEK_SET);
   lseek(fd, -100, SEEK_CUR);  // FAILS!
   write(fd, buf, 5);
   // Where does write happen?
   ```
   After 1st lseek: pos = 50
   2nd lseek: 50 + (-100) = -50 → FAILS, pos stays 50
   Write at 50

4. **Multiple Operations:**
   ```c
   fd = open("file.txt", O_RDWR);  // size = 20
   write(fd, buf, 10);
   lseek(fd, 5, SEEK_SET);
   write(fd, buf, 3);
   lseek(fd, 0, SEEK_END);
   write(fd, buf, 5);
   // Final size?
   ```
   After write 10: pos=10, size=20
   After lseek SEEK_SET: pos=5
   After write 3: pos=8, size=20 (within file)
   After lseek SEEK_END: pos=20
   After write 5: pos=25, size=25

5. **Read Operations:**
   ```c
   fd = open("file.txt", O_RDONLY);  // size = 100
   char buf[20];
   read(fd, buf, 10);
   read(fd, buf, 15);
   lseek(fd, 0, SEEK_CUR);  // Where are we?
   ```
   After 1st read: pos = 10
   After 2nd read: pos = 10 + 15 = 25
   lseek returns: 25

6. **Complex Sequence - Create your own using T1 as template**

---

**HOUR 4: Create File I/O Reference Sheet + PRINT (60 min)**

**Create this document:**
```
═══════════════════════════════════════════════════════
               FILE I/O REFERENCE SHEET
═══════════════════════════════════════════════════════

LSEEK - CHANGE FILE POSITION:
  Syntax: lseek(fd, offset, whence)
  Returns: new position, OR -1 if failed

  THE THREE FORMULAS:
    SEEK_SET (0): new_position = offset
    SEEK_CUR (1): new_position = current_position + offset
    SEEK_END (2): new_position = file_size + offset

  ⚠️ LSEEK CAN FAIL:
    If calculated position < 0 → lseek fails
    Failed lseek → position DOESN'T change
    Example: pos=10, lseek(fd, -20, SEEK_CUR) → fails!

READ - READ FROM FILE:
  Syntax: read(fd, buffer, n)
  - Reads UP TO n bytes from current position
  - Position moves forward by bytes_actually_read
  - Returns: bytes read (0 at EOF, -1 on error)
  - Does NOT change file size

WRITE - WRITE TO FILE:
  Syntax: write(fd, buffer, n)
  - Writes n bytes at current position
  - Position moves forward by n
  - If position > current_size → file GROWS
  - Returns: bytes written (-1 on error)

FILE SIZE RULES:
  ✓ File size = furthest byte ever written
  ✓ lseek does NOT change file size
  ✓ read does NOT change file size
  ✓ write CAN change file size (if writing beyond end)

TRACING STEPS:
  □ Step 1: Note initial size and position
  □ Step 2: For EACH operation:
      • Calculate new position using formula
      • Check if lseek failed (negative position)
      • Update file size if write beyond end
  □ Step 3: Final size = max byte position reached

TRACING TABLE FORMAT:
  | Operation | Formula | New Pos | New Size |
  |-----------|---------|---------|----------|
  | Initial   | -       | 0       | ??       |
  | lseek...  | ...     | ??      | ??       |
  | write...  | ...     | ??      | ??       |

COMMON MISTAKES TO AVOID:
  ✗ Thinking lseek changes file size
  ✗ Forgetting that failed lseek doesn't change position
  ✗ Not tracking position after each operation
  ✗ Confusing SEEK_SET, SEEK_CUR, SEEK_END formulas

MEMORY TRICKS:
  • SEEK_SET = "Set to absolute position"
  • SEEK_CUR = "Current + offset"
  • SEEK_END = "End + offset"
```

**🖨️ PRINT THIS SHEET IMMEDIATELY**

---

### **TUESDAY, NOV 26 - 5 HOURS**
**Focus: SIGNALS + Start PIPES**

#### **Your 5-Hour Study Block**

**HOUR 1: Signals Tracing Template (60 min)**

**Study Material:**
- Open Signals_F25.pptx
- Focus on: signal(), alarm(), process groups, kill

**Create SIGNAL TRACING TEMPLATE:**
```
═══════════════════════════════════════════
          SIGNAL TRACING TEMPLATE
═══════════════════════════════════════════

Step 1: Count total processes
  After all forks: _____ processes

Step 2: Identify process groups
  Default group (pgid): _____ (all start in parent's group)
  New groups created (setpgid): _____

Step 3: Signal delivery analysis
  Signal type: _______
  Sent to which group: _______
  Which processes receive it: _______
  Which have handlers registered: _______

Step 4: Calculate result
  Handler executions: _____
  OR Processes that survive: _____
  OR Final output: _____

COMMON SIGNALS:
  SIGINT (2): Ctrl-C (catchable)
  SIGALRM (14): alarm() timer
  SIGTERM (15): termination request
  SIGKILL (9): force kill (NOT catchable)
  SIGSTOP (19): stop process (NOT catchable)
```

**HOUR 2: Master Signal Questions (60 min)**

**Example 1 - T1 Alarm Question:**
```c
void handler1(int signo){
    signal(SIGALRM, handler1);  // Re-register handler
    printf("\nIn the handler\n");
    // NOTE: Does NOT call alarm() again!
}

int main(){
    signal(SIGALRM, handler1);  // Register handler
    alarm(8);  // Set timer for 8 seconds
    
    for(int i=0; i<10; i++)
        sleep(1);  // Sleep total 10 seconds
}
```

**TRACE:**
```
Time 0s: 
  - signal(SIGALRM, handler1) registered
  - alarm(8) starts countdown

Time 1-7s: 
  - Sleeping...

Time 8s: 
  - SIGALRM delivered
  - handler1() executes
  - Prints "\nIn the handler\n"
  - Re-registers handler (signal(SIGALRM, handler1))
  - BUT: No new alarm() called!
  - Returns from handler

Time 9-10s:
  - Continue sleeping
  - No more alarms (only one alarm was set!)

Program exits after 10 seconds.

ANSWER: Handler called ONCE only
```

**🔑 KEY INSIGHT: alarm() is ONE-SHOT!**
- alarm() sets a timer for N seconds
- After N seconds, SIGALRM delivered ONCE
- If handler wants another alarm, must call alarm() again

---

**Example 2 - Process Groups:**
```c
void handler(){ exit(0); }

int main(){
    signal(SIGINT, handler);
    
    for(int i=1; i<4; i++)
        fork();  // 8 processes total
    
    if((pid=fork()) == 0){  // All 8 processes do this
        // Child from each fork
        setpgid(0, getpid());  // Create new process group
    }
    
    // Ctrl-C pressed (sends SIGINT)
}
```

**TRACE:**
```
After for loop:
  fork() 3 times → 2³ = 8 processes
  All in same process group (parent's group)

After if(fork()):
  All 8 processes fork → 16 processes total
  
  For each of the 8 forks:
    Parent: stays in original group
    Child: executes setpgid(0, getpid()) → creates NEW group
  
  Result:
    8 parents in ORIGINAL group (foreground)
    8 children in NEW groups (each in own group)

When Ctrl-C pressed:
  SIGINT sent to FOREGROUND group only
  
  8 parents receive SIGINT → handler() → exit(0) → DIE
  8 children in different groups → DON'T receive signal → SURVIVE

ANSWER: 8 processes survive
```

**🔑 KEY INSIGHT: Ctrl-C (SIGINT) only kills foreground group!**

---

**HOUR 3: Signal Practice + Reference Sheet (60 min)**

**Practice Problems:**

1. **Repeated Alarms:**
   ```c
   void handler(int sig){
       signal(SIGALRM, handler);
       alarm(2);  // Re-arm!
       printf("Tick\n");
   }
   
   int main(){
       signal(SIGALRM, handler);
       alarm(2);
       sleep(7);
   }
   // How many "Tick" prints?
   ```
   Answer: 3 times (at 2s, 4s, 6s)

2. **No Handler:**
   ```c
   int main(){
       alarm(5);
       sleep(10);
   }
   // What happens?
   ```
   Answer: Program terminates at 5s (default SIGALRM action = terminate)

3. **Process Groups:**
   Count survivors in various setpgid scenarios

**Create SIGNAL REFERENCE SHEET:**
```
═══════════════════════════════════════════════════════
              SIGNAL REFERENCE SHEET
═══════════════════════════════════════════════════════

SIGNAL FUNCTION:
  signal(SIG, handler_function)
  - Registers handler for that signal
  - When signal arrives, handler executes
  - After handler completes, may need to re-register

ALARM FUNCTION:
  alarm(seconds)
  - Sets ONE-SHOT timer for N seconds
  - After N seconds, sends SIGALRM to calling process
  - Only ONE alarm can be active at a time
  - New alarm() call cancels previous alarm
  
  ⚠️ CRITICAL: alarm() is ONE-SHOT!
     Handler must call alarm() again for repeat

KILL FUNCTION:
  kill(pid, signal)
  - Sends signal to process with that PID
  - kill(-pgid, signal) sends to process group

PROCESS GROUPS:
  setpgid(0, getpid())
  - Creates new process group
  - First arg 0 = calling process
  - Second arg = new group ID (usually own PID)
  
  ⚠️ Ctrl-C (SIGINT) sends to FOREGROUND group only!
     Processes in other groups survive

COMMON SIGNALS:
  Number | Name    | Default Action | Catchable?
  -------|---------|----------------|------------
  2      | SIGINT  | Terminate      | YES
  9      | SIGKILL | Terminate      | NO
  14     | SIGALRM | Terminate      | YES
  15     | SIGTERM | Terminate      | YES
  19     | SIGSTOP | Stop           | NO
  18     | SIGCONT | Continue       | YES

TRACING STEPS:
  □ Step 1: Count processes after all forks
  □ Step 2: Identify process groups
  □ Step 3: When signal sent:
      • Which group receives it?
      • Which processes have handlers?
  □ Step 4: Count handler executions OR survivors

COMMON PATTERNS:
  Pattern 1: One-shot alarm
    alarm(N) → handler runs once after N seconds
  
  Pattern 2: Repeating alarm
    Handler calls alarm(N) again → repeats
  
  Pattern 3: Process group isolation
    setpgid() → creates new group → survives Ctrl-C
```

**🖨️ PRINT THIS SHEET**

---

**HOURS 4-5: PIPES - Byte Flow Basics (120 min)**

**Study Material:**
- Open Pipes_F25.pptx
- Focus on: pipe(), read/write, multiple processes

**Create PIPE TRACING TEMPLATE:**
```
═══════════════════════════════════════════
           PIPE TRACING TEMPLATE
═══════════════════════════════════════════

Step 1: Count processes
  After all forks: _____ processes

Step 2: Identify roles
  Writers (who calls write): _____
  Readers (who calls read): _____

Step 3: Track bytes
  Total bytes written to pipe: _____ bytes
  Number of processes reading: _____
  Bytes per read() call: _____ bytes

Step 4: Race condition analysis
  Multiple readers? YES / NO
  If YES → Cannot predict exact distribution
  
Step 5: Final answer
  Total bytes that CAN be read: _____
  OR "Cannot say" (race condition)

PIPE RULES:
  pipe(fd) creates: fd[0] = read end
                    fd[1] = write end
  Data flows: write end → read end (ONE direction)
```

**T2 Pipe Question:**
```c
int fd[2];
int main_pid = getpid();
pipe(fd);

int pid1 = fork();  // 2 processes
int pid2 = fork();  // 4 processes

if(getpid() == main_pid){  // Only original parent
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    // Total: 80 bytes written
} else {  // Children
    char buff[20];
    int num = read(fd[0], buff, 20);
    printf("%s\n", buff);
}
```

**TRACE:**
```
After fork() twice:
  4 processes total

Identify roles:
  Original parent: getpid() == main_pid → TRUE → WRITER
  Other 3 processes: getpid() == main_pid → FALSE → READERS

Bytes written:
  Parent writes 4 times × 20 bytes = 80 bytes total

Bytes reading:
  3 processes each try to read(fd[0], buff, 20)
  Each tries to read 20 bytes

RACE CONDITION:
  All 3 readers compete for data in pipe
  Possibilities:
    - Child 1 reads 20, Child 2 reads 20, Child 3 reads 20 = 60 total
    - Child 1 reads 40, Child 2 reads 40, Child 3 reads 0 = 80 total
    - Child 1 reads 80, Child 2 reads 0, Child 3 reads 0 = 80 total
    - Many other combinations...

Order is UNPREDICTABLE!

ANSWER: "Cannot say" (race condition) OR 60-80 bytes depending on options
```

**🔑 KEY INSIGHT: Multiple readers = RACE CONDITION = Unpredictable!**

**Create PIPE REFERENCE:**
```
═══════════════════════════════════════════════════════
                PIPE REFERENCE SHEET
═══════════════════════════════════════════════════════

PIPE CREATION:
  pipe(int fd[2])
  - Creates a pipe with two file descriptors
  - fd[0] = read end
  - fd[1] = write end
  - Data flows ONE direction: write → read

WRITE TO PIPE:
  write(fd[1], buffer, n)
  - Puts n bytes into the pipe
  - Returns: n (bytes written) or -1 (error)
  - Data stays in pipe until read

READ FROM PIPE:
  read(fd[0], buffer, n)
  - Reads UP TO n bytes from pipe
  - Returns: bytes actually read, 0 (EOF), or -1 (error)
  - Removes data from pipe
  - Blocks if pipe is empty (and write end still open)

MULTIPLE PROCESSES:
  After fork(), both parent and child have access to:
    - Same fd[0] (read end)
    - Same fd[1] (write end)

RACE CONDITION:
  When multiple processes read from same pipe:
    - Order of reads is UNPREDICTABLE
    - Cannot determine which process gets which bytes
    - Total bytes read ≤ total bytes written
    - Answer: "Cannot say" or range of possibilities

PREDICTABLE CASES:
  - Only ONE reader, ONE writer → predictable
  - After closing one end in each process → predictable

TRACING STEPS:
  □ Count total processes
  □ Identify writers (how many bytes total)
  □ Identify readers (how many processes)
  □ Check for race condition
  □ Calculate total bytes OR say "Cannot say"

COMMON PATTERNS:
  Pattern 1: Parent writes, children read
    → Race condition if multiple children
  
  Pattern 2: Children write, parent reads
    → Parent might get mixed data
  
  Pattern 3: Pipeline (one-to-one)
    → Close unused ends → predictable
```

**🖨️ PRINT THIS SHEET**

---

### **WEDNESDAY, NOV 27 - 10 HOURS**
**Focus: THREADS + SHELL + POINTERS + Intensive Review**

#### **Morning Block: 4 hours (8 AM - 12 PM)**

**HOURS 1-2: THREADS - Shared Variable Tracking (120 min)**

**Study Material:**
- Open Threads_F25.pptx
- Focus on: pthread_create, pthread_join, shared vs local variables

**Create THREAD TRACING TEMPLATE:**
```
═══════════════════════════════════════════
          THREAD TRACING TEMPLATE
═══════════════════════════════════════════

Shared variables (global): _________________

Thread 1 (func: _______):
  What it does: _______________________
  Ends when: __________________________
  
Thread 2 (func: _______):
  What it does: _______________________
  Ends when: __________________________

Main thread:
  Waits for: __________________________
  Continues when: ______________________
  Returns when: ________________________

Timeline:
  Time 0: _____________________________
  Time 1s: ____________________________
  Time 2s: ____________________________
  [etc...]

Result: __________________________________

CRITICAL RULE:
  When main() returns → ALL threads DIE!
```

**T2 Thread Question:**
```c
int counter = 0;  // SHARED GLOBAL VARIABLE!

void* func1(void* p) {
    while(1) {
        printf("thread one\n");
        sleep(1);
        counter++;  // Increments shared counter
        if(counter == 3) {
            pthread_cancel(pthread_self());  // Cancel self
        }
    }
}

void* func2(void* p) {
    while(1) {
        printf("thread two\n");
        sleep(1);
    }
}

void* func3(void* p) {
    while(1) {
        printf("thread three\n");
        sleep(1);
    }
}

int main() {
    pthread_t thread_one, thread_two, thread_three;
    
    pthread_create(&thread_one, NULL, func1, NULL);
    pthread_create(&thread_two, NULL, func2, NULL);
    pthread_create(&thread_three, NULL, func3, NULL);
    
    pthread_join(thread_one, NULL);  // Wait for thread_one to finish
    // Main returns after this!
}
```

**TRACE:**
```
Time 0:
  - Main creates 3 threads
  - All 3 start running IMMEDIATELY
  - Main blocks at pthread_join(thread_one)

Time 0-1s:
  - thread_one: prints "thread one", sleeps
  - thread_two: prints "thread two", sleeps
  - thread_three: prints "thread three", sleeps

Time 1s:
  - thread_one wakes: counter++ → counter = 1
  - Checks: counter == 3? NO, continues loop
  - All 3 print again

Time 2s:
  - thread_one wakes: counter++ → counter = 2
  - Checks: counter == 3? NO, continues loop
  - All 3 print again

Time 3s:
  - thread_one wakes: counter++ → counter = 3
  - Checks: counter == 3? YES!
  - pthread_cancel(pthread_self()) → thread_one EXITS
  - pthread_join returns (thread_one finished)
  - main() RETURNS
  - ALL THREADS DIE (thread_two and thread_three killed!)

Program terminates after approximately 3 seconds.

ANSWER: Program runs for about 3 seconds then terminates.
        All threads die when main() returns.
```

**🔑 KEY INSIGHTS:**
1. **Global variables are SHARED** by all threads
2. **pthread_join blocks** until specified thread finishes
3. **When main() returns, ALL threads are killed** (even if in infinite loops!)

**Practice:** Create 3 variations of this question with different counter values, different threads

**Create THREAD REFERENCE:**
```
═══════════════════════════════════════════════════════
               THREAD REFERENCE SHEET
═══════════════════════════════════════════════════════

PTHREAD_CREATE:
  pthread_create(&thread, NULL, function, arg)
  - Creates new thread running function
  - Thread starts IMMEDIATELY
  - Returns 0 on success, error code on failure

PTHREAD_JOIN:
  pthread_join(thread, NULL)
  - Blocks calling thread until specified thread finishes
  - Like waitpid() for threads
  - Returns 0 on success

PTHREAD_CANCEL:
  pthread_cancel(thread)
  - Requests thread to terminate
  - Thread exits at next cancellation point

PTHREAD_SELF:
  pthread_self()
  - Returns calling thread's ID
  - Used like: pthread_cancel(pthread_self())

MEMORY MODEL:
  Global variables: SHARED by ALL threads
    int counter = 0;  ← All threads see same counter
  
  Local variables: PRIVATE to each thread
    void* func(void* p) {
        int local = 0;  ← Each thread has own copy
    }

EXECUTION MODEL:
  - Threads run CONCURRENTLY (pseudo-parallel)
  - Order of execution is UNPREDICTABLE
  - Must use synchronization for shared data
  - sleep() causes thread to pause

CRITICAL RULE - PROGRAM TERMINATION:
  ⚠️ When main() returns → ALL threads are killed!
     Even if threads are in infinite loops!
     
  Example:
    Thread 1: infinite loop
    Thread 2: infinite loop
    Main: pthread_join(thread_one)
    
    When thread_one exits → join returns
    → main() returns
    → thread_two DIES even though it's looping!

TRACING STEPS:
  □ Identify shared variables (global)
  □ Track each thread's execution
  □ Note when pthread_join blocks
  □ Determine when thread exits
  □ Remember: main returns → all threads die

COMMON PATTERNS:
  Pattern 1: Thread counts to N then exits
    Check counter in loop
  
  Pattern 2: Main waits for one thread
    Other threads die when main returns
  
  Pattern 3: Race conditions
    Multiple threads modify shared variable
    Order unpredictable
```

**🖨️ PRINT THIS SHEET**

---

**HOURS 3-4: SHELL SCRIPTS - Line by Line (120 min)**

**Study Material:**
- Open Shell_Programming_F25.pptx
- Focus on: variables, loops, conditions, job control

**Create SHELL TRACING TEMPLATE:**
```
═══════════════════════════════════════════
         SHELL SCRIPT TRACING TEMPLATE
═══════════════════════════════════════════

Create a table to track state:

| Line | count | Condition | Action | Output |
|------|-------|-----------|--------|--------|
| 1    |       |           |        |        |
| 2    |       |           |        |        |
...

Track:
- Variable values after each line
- Loop iterations
- Condition results (true/false)
- What gets printed
- When script stops/continues
```

**T2 Shell Question:**
```bash
#!/bin/bash
count=1
while [ $count -ne 15 ]
do
    echo hello1
    if [ $count -lt 10 ]; then
        echo hello2
        count=$(($count+1))
        sleep 1
    else
        kill -19 $$  # Stop myself (SIGSTOP)
        fg           # Resume (user types this)
        echo hello3
        break
    fi
done
```

**TRACE:**
```
| Iter | count | count<10? | count≠15? | Action | Output |
|------|-------|-----------|-----------|--------|---------|
| 1 | 1 | YES | YES | echo hello1, hello2, count=2, sleep 1s | hello1 \n hello2 |
| 2 | 2 | YES | YES | echo hello1, hello2, count=3, sleep 1s | hello1 \n hello2 |
| 3 | 3 | YES | YES | echo hello1, hello2, count=4, sleep 1s | hello1 \n hello2 |
| 4 | 4 | YES | YES | echo hello1, hello2, count=5, sleep 1s | hello1 \n hello2 |
| 5 | 5 | YES | YES | echo hello1, hello2, count=6, sleep 1s | hello1 \n hello2 |
| 6 | 6 | YES | YES | echo hello1, hello2, count=7, sleep 1s | hello1 \n hello2 |
| 7 | 7 | YES | YES | echo hello1, hello2, count=8, sleep 1s | hello1 \n hello2 |
| 8 | 8 | YES | YES | echo hello1, hello2, count=9, sleep 1s | hello1 \n hello2 |
| 9 | 9 | YES | YES | echo hello1, hello2, count=10, sleep 1s| hello1 \n hello2 |
| 10 | 10 | NO | YES | echo hello1, else branch | hello1 |
| | | | | kill -19 $$ (SIGSTOP sent to self) | [STOPPED] |
| | | | | User types: fg | [RESUMED] |
| | | | | echo hello3 | hello3 |
| | | | | break (exit while loop) | |

Script ends.

LAST OUTPUT: hello3

ANSWER: hello3 ✓
```

**🔑 KEY INSIGHTS:**
1. Track variables line by line in a table
2. `count=$(($count+1))` is arithmetic (increment)
3. `$$` = current shell PID
4. `kill -19 $$` = stop myself (SIGSTOP)
5. `fg` = resume stopped job
6. `break` = exit loop

**Create SHELL REFERENCE:**
```
═══════════════════════════════════════════════════════
            SHELL SCRIPT REFERENCE SHEET
═══════════════════════════════════════════════════════

VARIABLES:
  Assignment: count=1        (NO SPACES around =)
  Use: $count or ${count}
  Arithmetic: count=$(($count+1))  or count=$((count + 1))

CONDITIONS:
  [ expression ]  ← Note: SPACES required inside brackets!
  
  Numeric comparisons:
    -eq : equal          [ $a -eq $b ]
    -ne : not equal      [ $a -ne $b ]
    -lt : less than      [ $a -lt $b ]
    -gt : greater than   [ $a -gt $b ]
    -le : less or equal  [ $a -le $b ]
    -ge : greater or equal [ $a -ge $b ]
  
  String comparisons:
    = : equal            [ "$a" = "$b" ]
    != : not equal       [ "$a" != "$b" ]

LOOPS:
  while [ condition ]
  do
      commands
  done
  
  for variable in list
  do
      commands
  done

CONTROL:
  break : exit loop
  continue : skip to next iteration

JOB CONTROL:
  $$       : PID of current shell
  $!       : PID of last background process
  kill -19 PID : Send SIGSTOP (stop process)
  kill -18 PID : Send SIGCONT (continue process)
  fg       : Bring stopped job to foreground
  bg       : Continue stopped job in background

SPECIAL SIGNALS FOR KILL:
  -19 : SIGSTOP (stop/pause process)
  -18 : SIGCONT (continue process)
  -9  : SIGKILL (force kill)
  -15 : SIGTERM (terminate gracefully)

TRACING METHOD:
  1. Create table with columns:
     | Iteration | Variables | Condition | Action | Output |
  2. Fill in row by row
  3. Update variable values after each iteration
  4. Track what gets printed
  5. Last echo statement = typical answer

COMMON PATTERNS:
  Pattern 1: Count until condition
    while [ $count -lt 10 ]
    do
        count=$(($count+1))
    done
  
  Pattern 2: Self-stop and resume
    kill -19 $$  # Stop myself
    # (user types fg to resume)
  
  Pattern 3: Break on condition
    if [ condition ]; then
        break
    fi

SYNTAX REMINDERS:
  ✓ count=1          (correct)
  ✗ count = 1        (wrong - no spaces!)
  ✓ [ $count -lt 10 ] (correct - spaces inside)
  ✗ [$count -lt 10]   (wrong - no spaces!)
```

**🖨️ PRINT THIS SHEET**

---

#### **Lunch Break: 12 PM - 1 PM**

---

#### **Afternoon Block: 6 hours (1 PM - 7 PM)**

**HOUR 5: POINTERS Quick Review (60 min)**

**Study Material:**
- Open Advanced_C_Programming_Techniques_F25.pptx
- Focus on: pointer basics, dereferencing, pointer arithmetic

**Create POINTER TRACING TEMPLATE:**
```
═══════════════════════════════════════════
          POINTER TRACING TEMPLATE
═══════════════════════════════════════════

Memory layout:

| Address | Variable | Value | Type |
|---------|----------|-------|------|
| 1000    | x        | 5     | int  |
| 1004    | y        | 10    | int  |
| 1008    | ptr      | 1000  | int* |

After each operation:
- Update values in table
- Track what ptr points to
- Calculate results
```

**Simple Example:**
```c
int x = 5;
int *ptr = &x;
*ptr = 10;
printf("%d", x);  // What prints?
```

**TRACE:**
```
Memory initially:
  Address 1000: x = 5
  Address 1004: ptr = ?

After int *ptr = &x:
  ptr = 1000 (address of x)

After *ptr = 10:
  Go to address 1000 (where ptr points)
  Change value there to 10
  x is now 10

printf("%d", x) → Prints 10
```

**Pointer Arithmetic:**
```c
int arr[] = {10, 20, 30, 40};
int *ptr = arr;  // Points to arr[0]
ptr++;           // Move to next int
printf("%d", *ptr);  // What prints?
```

**TRACE:**
```
arr[0] at 1000: 10
arr[1] at 1004: 20
arr[2] at 1008: 30
arr[3] at 1012: 40

ptr = arr → ptr = 1000
ptr++ → ptr = 1004 (moved to next int)
*ptr → value at 1004 → 20

Prints: 20
```

**Create POINTER REFERENCE:**
```
═══════════════════════════════════════════════════════
              POINTER REFERENCE SHEET
═══════════════════════════════════════════════════════

POINTER BASICS:
  int *ptr;      // ptr is a pointer (holds an address)
  ptr = &x;      // ptr now holds address of x
  *ptr           // Value AT the address ptr points to
  &var           // Address OF variable var

OPERATIONS:
  Declare:    int *ptr;
  Assign address:  ptr = &x;
  Dereference:     value = *ptr;  (get value at address)
  Modify through pointer:  *ptr = 10;  (change value at address)

POINTER ARITHMETIC:
  ptr++       // Move to next element (adds sizeof(type))
  ptr--       // Move to previous element
  ptr + n     // Move n elements forward
  ptr - n     // Move n elements backward

ARRAYS AND POINTERS:
  int arr[4] = {10, 20, 30, 40};
  int *ptr = arr;  // ptr points to arr[0]
  
  Equivalences:
    arr[0]  = *arr      = *ptr
    arr[1]  = *(arr+1)  = *(ptr+1)
    arr[i]  = *(arr+i)  = *(ptr+i)
  
  &arr[i] = arr + i = ptr + i

TRACING METHOD:
  1. Draw memory layout (addresses and values)
  2. Track what each pointer points to
  3. When dereferencing (*ptr), go to that address
  4. Update values at addresses when assigning through pointers
  5. For arithmetic, calculate new address

COMMON OPERATIONS:
  *ptr = 10;       // Change value at ptr's address
  ptr = ptr + 1;   // Move ptr to next element
  x = *ptr;        // Read value at ptr's address
  ptr = &y;        // Make ptr point to y

MEMORY TRICK:
  • & = "address of"
  • * = "value at"
```

**🖨️ PRINT THIS SHEET**

---

**HOURS 6-7: ALL T1 QUESTIONS REVIEW (120 min)**

**Mission:** Go through EVERY question from Test 1

**Method:**
1. For each question:
   - Identify topic (5 seconds)
   - Pull out correct template (5 seconds)
   - Trace on scratch paper (3-4 minutes)
   - Check answer
2. Keep track of:
   - Questions you got RIGHT ✓
   - Questions you got WRONG ✗
3. For wrong answers:
   - Re-trace step by step
   - Identify your mistake
   - Practice that pattern again

**Topics to cover:**
- Fork questions
- File I/O questions
- Signal questions
- Any others from T1

**Time per question: ~8 minutes** (trace + review)
**Total questions in T1: ~15?** (estimate)

---

**HOURS 8-9: ALL T2 SAMPLE QUESTIONS REVIEW (120 min)**

**Mission:** Go through EVERY question from Test 2 samples

**Same method as T1 review**

**Focus on:**
- Questions with code tracing
- Questions you're unsure about
- Questions that appear frequently

**Speed goal: Can you trace each in under 4 minutes?**

---

**HOUR 10: WEAK TOPIC DEEP DIVE (60 min)**

**Identify your weakest topic:**
- Which topic took longest?
- Which had most mistakes?
- Which felt least confident?

**Spend this hour ONLY on that topic:**
- Re-read that template
- Review that PPT section
- Do 5-10 more practice problems
- Master the pattern

**Common weak topics:**
- Signals (process groups, alarm behavior)
- File I/O (failed lseek, size calculations)
- Pipes (race conditions)

---

### **THURSDAY, NOV 28 (EXAM DAY) - 10 HOURS**

#### **Morning Block: 6 AM - 12 PM (6 hours)**

**6:00 - 7:00 AM: Wake Up & Light Review (60 min)**
- Light breakfast
- Don't study new material
- Skim through your 7 printed reference sheets
- Stay calm and positive

**7:00 - 9:00 AM: SPEED DRILLS (120 min)**

**Practice Set 1:** 10 mixed questions (60 min)
- 2 Fork
- 2 File I/O
- 2 Signals
- 1 Pipe
- 1 Thread
- 1 Shell
- 1 Pointer

**Time limit: 5-6 minutes per question**
**Use your templates!**

**Practice Set 2:** 10 different mixed questions (60 min)
- Same distribution
- Focus on speed AND accuracy
- Build confidence

**9:00 - 10:00 AM: TEMPLATE MASTERY (60 min)**

**Exercise: Template Speed Test**
- Lay out all 7 templates
- Practice identifying topic quickly:
  - See fork code → grab Fork template (under 5 seconds)
  - See lseek → grab File I/O template (under 5 seconds)
  - See signal() → grab Signal template (under 5 seconds)

**Create "topic identification" flash cards:**
```
Front: Code snippet
Back: Topic name

fork() { ... }     → FORK
lseek(fd, ...)     → FILE I/O
signal(SIGINT, ..) → SIGNALS
pipe(fd)           → PIPES
pthread_create...  → THREADS
while [ ]          → SHELL
int *ptr           → POINTERS
```

**10:00 - 11:00 AM: FINAL PRACTICE (60 min)**

**Simulation:** Full exam mode
- 10 random questions
- Time yourself strictly: 4-5 min per question
- No looking at notes
- Only use your printed templates
- Check answers after

**Identify any remaining weak areas**

**11:00 AM - 12:00 PM: ORGANIZATION & LIGHT REVIEW (60 min)**

**Organize your exam materials:**

**Create TABS for quick access:**
- Tab 1: 📄 FORK
- Tab 2: 📄 FILE I/O
- Tab 3: 📄 SIGNALS
- Tab 4: 📄 PIPES
- Tab 5: 📄 THREADS
- Tab 6: 📄 SHELL
- Tab 7: 📄 POINTERS

**Use sticky notes or colored paper clips**

**Pack your exam bag:**
- ✓ 7 printed reference sheets (with tabs)
- ✓ Blank scratch paper
- ✓ Multiple pens (blue/black)
- ✓ Multiple pencils + eraser
- ✓ Calculator (if allowed)
- ✓ Water bottle
- ✓ Student ID
- ✓ Watch (for time management)

**Final review:**
- Read each template once (don't practice)
- Review key formulas:
  - Fork: 2^n processes
  - File I/O: SEEK formulas
  - Signal: alarm() is one-shot
  - Threads: main returns → all die

---

#### **12:00 - 1:00 PM: LUNCH (60 min)**

**EAT LIGHT:**
- Not too heavy (don't want to be sleepy)
- Protein + carbs for energy
- Stay hydrated

**NO STUDYING during lunch**
- Relax
- Listen to music
- Take a short walk
- Stay positive

---

#### **1:00 - 3:30 PM: PRE-EXAM PREPARATION (150 min)**

**1:00 - 2:00 PM: Final Light Review (60 min)**
- Read through templates ONE MORE TIME
- Don't do problems
- Just refresh memory
- Stay fresh, don't burn out

**2:00 - 2:30 PM: Mental Preparation (30 min)**
- Deep breathing exercises
- Positive affirmations:
  - "I have prepared well"
  - "I can trace any code"
  - "I will get 70%+"
- Visualize success
- Stay calm

**2:30 - 3:00 PM: Pack & Prepare (30 min)**
- Double-check exam bag
- Use restroom
- Put phone on silent
- Review exam location/room number

**3:00 - 3:30 PM: Travel to Exam (30 min)**
- Arrive early (by 3:30 for 4 PM exam)
- Find your seat
- Set up materials
- Deep breaths

---

#### **3:30 - 4:00 PM: AT EXAM LOCATION (30 min)**

**Setup:**
- Lay out your 7 reference sheets with tabs visible
- Have scratch paper ready
- Pens/pencils ready
- Water bottle accessible

**Final mental prep:**
- Deep breaths
- "I've got this"
- Stay calm and focused

**When exam is handed out:**
- Write your name/ID immediately
- Read instructions carefully
- Glance through exam to see question types
- Start!

---

## 🎯 EXAM STRATEGY (4:00 - 6:00 PM)

### **The 3-Minute Method:**

**For EACH question:**

**⏱️ Step 1: Identify Topic (10 seconds)**
- Fork? File I/O? Signals? Pipes? Threads? Shell? Pointer?
- Look for keywords: fork(), lseek(), signal(), pipe(), pthread_, bash, *ptr

**⏱️ Step 2: Grab Template (5 seconds)**
- Use your tabs to find correct sheet instantly
- Place it next to your scratch paper

**⏱️ Step 3: Trace on Scratch Paper (2-3 minutes)**
- Follow template steps exactly
- Write everything down (don't do it in your head!)
- Don't skip steps
- Be systematic

**⏱️ Step 4: Find Answer (30 seconds)**
- Check your traced result
- Match to options (a, b, c, d)
- Select answer confidently

**⏱️ Step 5: Quick Check (15 seconds)**
- Does answer make sense?
- Any obvious calculation errors?
- Move on!

**Total time per question: ~3-4 minutes**
**40 questions × 3.5 min = 140 minutes = 2 hours 20 min**
**Perfect! You'll have 20 min buffer for harder questions**

---

### **Time Management Strategy:**

**First Pass (90 minutes):**
- Do ALL easy/medium questions
- Skip hard questions (mark them)
- Goal: Answer ~30 questions

**Second Pass (20 minutes):**
- Return to marked hard questions
- Spend more time on these
- Use templates carefully

**Final Check (10 minutes):**
- Review any uncertain answers
- Make sure all questions answered
- Check for silly mistakes

---

### **True/False Strategy:**

**For T/F questions:**
- If you can trace quickly (under 2 min) → trace it
- If purely conceptual → use reference sheets
- If completely unsure → educated guess (DON'T leave blank!)

**Common T/F patterns:**
- "alarm() fires repeatedly" → FALSE (one-shot)
- "Failed lseek changes position" → FALSE
- "Global variables shared by threads" → TRUE
- "fork() returns 0 to parent" → FALSE (returns child PID)

---

### **Staying Calm During Exam:**

**If you get stuck:**
1. Skip it, mark it
2. Move to next question
3. Come back later
4. Don't panic!

**If running short on time:**
1. Answer all remaining questions with best guess
2. Better to guess than leave blank
3. Use templates even if rushed

**Trust your preparation:**
- You've practiced 33 hours
- You have perfect templates
- You can trace ANY code
- You've got this! 🎯

---

## 📋 FINAL CHECKLIST

### **By End of Sunday:**
- □ Fork template created and printed
- □ Fork reference sheet printed
- □ Practiced T1 fork question 5+ times

### **By End of Monday:**
- □ File I/O template created and printed
- □ File I/O reference sheet printed
- □ Practiced T1 file I/O question 4+ times

### **By End of Tuesday:**
- □ Signal template and reference printed
- □ Pipe template and reference printed
- □ Practiced both T1 and T2 signal questions

### **By End of Wednesday:**
- □ Thread, Shell, Pointer references printed
- □ All 7 sheets organized with tabs
- □ Reviewed all T1 and T2 questions
- □ Identified weak topic and practiced more

### **Thursday Morning:**
- □ All materials packed
- □ Tab system working
- □ Speed drills completed
- □ Confident and ready!

### **At Exam:**
- □ All 7 reference sheets with tabs
- □ Scratch paper
- □ Multiple pens/pencils
- □ Water
- □ Student ID
- □ Calm mindset
- □ CONFIDENCE! 💪

---

## 🚀 MOTIVATIONAL REMINDERS

**Remember:**
- Every MCQ is just a tracing problem
- You have templates for EVERYTHING
- You've practiced 33+ hours
- You know the patterns
- You can do this!

**Your Goal: 70%+**
- 40 questions total
- Need 28+ correct (70%)
- With your templates: VERY achievable!

**The Secret:**
1. Identify topic (5 sec)
2. Use template (3 min)
3. Find answer (30 sec)
4. Move on!

**Trust the process.**
**Trust your practice.**
**Trust yourself.**

---

## 💪 FINAL WORDS

You have a solid plan.  
You have powerful templates.  
You have 33 hours to prepare.

**Stay focused.**
**Stay disciplined.**
**Stay confident.**

Every hour counts!
Every practice problem matters!

When you walk into that exam on Thursday,
you'll have your 7 reference sheets,
you'll have your templates,
and you'll have the skills to trace ANY code.

**You're going to ace this exam!** 🎯🔥

**NOW GO EXECUTE THIS PLAN!**
**START WITH HOUR 1: FORK TEMPLATE!**
**SET YOUR TIMER AND BEGIN!** ⏰

---

# YOU'VE GOT THIS! 🎯💪🔥

Good luck! You're going to do GREAT! 🌟
