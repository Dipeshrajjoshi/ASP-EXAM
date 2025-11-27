# 🎯 COMP8567 - ULTIMATE EXAM SUCCESS GUIDE
## Target: 60%+ (All T/F Correct + 50% Code MCQs)
### Open Book • Print & Tab This Document • Based on Professor's Slides

---

## 📊 YOUR WINNING STRATEGY

**Exam Structure:**
- **~10 True/False** → Target: **10/10 = 100%** ✅ (EASY MARKS!)
- **~30 Code Output MCQs** → Target: **15/30 = 50%** ✅
- **TOTAL: 25/40 = 62.5% PASS!** 🎯

**Your Advantages:**
- ✅ OPEN BOOK (use this guide!)
- ✅ Testing UNDERSTANDING, not memorization
- ✅ Professor's actual code patterns included
- ✅ Step-by-step tracing methods

---

## 📚 TABLE OF CONTENTS (Tab These Sections!)

1. [**FORK & PROCESSES**](#fork) - Process counting, wait(), exec()
2. [**FILE I/O & lseek**](#fileio) - The 3 formulas, position vs size
3. [**SIGNALS**](#signals) - alarm() ONE-SHOT, process groups
4. [**PIPES**](#pipes) - Race conditions, multiple readers
5. [**THREADS**](#threads) - Shared memory, main() returns rule
6. [**SHELL**](#shell) - Variables, job control, loops
7. [**SOCKETS**](#sockets) - TCP/UDP, client/server flow
8. [**QUICK FORMULAS**](#formulas) - Emergency reference
9. [**EXAM TRAPS**](#traps) - Common mistakes to avoid

---

<a name="fork"></a>
# 1️⃣ FORK & PROCESS CONTROL

## 🔥 THE CRITICAL RULES

### Rule #1: Return Values
```c
pid_t pid = fork();

// In PARENT: pid = child's PID (positive number like 12345)
// In CHILD:  pid = 0 (always zero)
// On ERROR:  pid = -1
```

### Rule #2: Process Counting Formula
```
n sequential forks = 2^n TOTAL processes

Examples:
fork();           // 1 → 2 processes
fork(); fork();   // 1 → 2 → 4 processes  
fork(); fork(); fork();  // 1 → 2 → 4 → 8 processes
```

### Rule #3: Separate Memory
```c
int x = 5;
fork();

if (pid == 0) {
    x = 10;  // Child changes x
}
// Parent's x stays 5, Child's x becomes 10
// SEPARATE COPIES!
```

---

## 💻 PROFESSOR'S ACTUAL CODE PATTERNS

### Pattern A: Count Total Processes
```c
int main() {
    fork();  // 2 processes
    fork();  // 4 processes
    fork();  // 8 processes
    
    printf("X");  // Prints 8 times!
}
```

**How to solve:**
1. Count fork() calls: 3
2. Apply formula: 2^3 = 8
3. Each process executes printf once = 8 outputs

---

### Pattern B: Identify Original Parent
```c
int main() {
    int pid1 = fork();  // 2 processes
    int pid2 = fork();  // 4 processes
    
    if (pid1 > 0 && pid2 > 0) {
        // ONLY original parent enters here!
        printf("I'm the original parent\n");
    }
}
```

**Process Table:**
| Process | pid1 | pid2 | Original Parent? |
|---------|------|------|------------------|
| A       | >0   | >0   | ✅ YES           |
| B       | 0    | >0   | ❌ NO            |
| C       | >0   | 0    | ❌ NO            |
| D       | 0    | 0    | ❌ NO            |

---

### Pattern C: waitpid() Variable Overwriting (EXAM FAVORITE!)
```c
int main() {
    int pid1 = fork();
    int pid2 = fork();
    int pid3 = fork();
    
    if (pid1 > 0 && pid2 > 0 && pid3 > 0) {
        int result;
        result = waitpid(pid1, NULL, 0);  // result = pid1 (e.g., 100)
        result = waitpid(pid2, NULL, 0);  // result = pid2 (e.g., 200) ← OVERWRITES!
        result = waitpid(pid3, NULL, 0);  // result = pid3 (e.g., 300) ← FINAL VALUE!
        
        printf("%d", result);  // Prints 300 (LAST value only!)
    }
}
```

**🔥 EXAM TRAP:** Each assignment REPLACES previous value!
- Result contains ONLY the LAST waitpid() return value
- Like writing on a whiteboard - new text erases old

---

## 📝 TRUE/FALSE - FORK (Quick Reference)

**✅ TRUE:**
1. fork() creates new process by duplicating calling process
2. fork() returns child's PID to parent
3. Three sequential fork()s create 8 processes (2^3)
4. After fork(), child inherits open file descriptors
5. A zombie is terminated but parent hasn't called wait()
6. An orphan's parent has terminated
7. waitpid() blocks until specified child terminates
8. fork() can fail and return -1
9. After fork(), both continue from line AFTER fork()

**❌ FALSE:**
10. fork() returns 0 to parent (NO - returns child PID!)
11. After fork(), parent and child share same memory (NO - separate!)
12. Changes to variables in child affect parent (NO - separate!)
13. getpid() returns same value in parent and child (NO - unique!)
14. Zombie processes consume CPU (NO - only table entry!)
15. Multiple waitpid() calls store all values (NO - last one wins!)
16. exec() creates new process (NO - replaces current!)
17. Code after exec() executes (NO - never returns!)
18. Order after fork() is guaranteed (NO - unpredictable!)

---

## 🎯 FORK TRACING METHOD

**For any fork question:**

**Step 1:** Count fork() calls = n
**Step 2:** Calculate 2^n = total processes
**Step 3:** Draw process tree if needed
**Step 4:** Track which processes execute which code

**Example:**
```c
if (fork()) {
    fork();
}
printf("Hi");
```

**Solution:**
- 1st fork: 2 processes (parent + child1)
- Parent (true): does 2nd fork → creates child2
- child1 (false): skips 2nd fork
- Total: 3 processes (parent, child1, child2)
- Each prints "Hi" = 3 outputs

---

<a name="fileio"></a>
# 2️⃣ FILE I/O & lseek

## 🔥 THE THREE SACRED FORMULAS

### Formula 1: SEEK_SET
```
new_position = offset
```
**Example:** `lseek(fd, 100, SEEK_SET);` → position = 100

### Formula 2: SEEK_CUR
```
new_position = current_position + offset
```
**Example:** If pos=50, `lseek(fd, 20, SEEK_CUR);` → position = 70

### Formula 3: SEEK_END
```
new_position = file_size + offset
```
**Example:** If size=200, `lseek(fd, -10, SEEK_END);` → position = 190

---

## 🔥 THE CRITICAL RULES

### Rule #1: lseek NEVER Changes Size
```c
lseek(fd, 1000, SEEK_SET);  // Position = 1000
                            // Size = UNCHANGED!
```

### Rule #2: Failed lseek
```
If calculated position < 0 → lseek FAILS (returns -1)
Position DOES NOT CHANGE!
```

**Example:**
```c
// Current position = 50
lseek(fd, -100, SEEK_CUR);  // 50 + (-100) = -50 → NEGATIVE!
                            // lseek FAILS
                            // Position STAYS at 50!
```

### Rule #3: Only write() Changes Size
```c
// File size changes ONLY when writing beyond current end
write(fd, buf, n);  // Can extend file
```

---

## 💻 PROFESSOR'S T1 EXAM PATTERN

**The Classic File I/O Question:**

```c
int fd = open("check.txt", O_RDWR);  // size = 40, pos = 0

lseek(fd, 100, SEEK_END);    // pos = ?  size = ?
write(fd, buf, 9);           // pos = ?  size = ?
write(fd, buf, 9);           // pos = ?  size = ?
lseek(fd, -159, SEEK_CUR);   // pos = ?  size = ?
write(fd, buf, 9);           // pos = ?  size = ?
write(fd, buf, 9);           // pos = ?  size = ?

close(fd);
// Final size = ?
```

---

## 🎯 STEP-BY-STEP TRACING METHOD

**Use this table format:**

| Operation | Formula | Calculation | Position | Size |
|-----------|---------|-------------|----------|------|
| open | - | - | 0 | 40 |
| lseek END | size+100 | 40+100 | 140 | 40 |
| write 9 | pos+9 | 140+9 | 149 | 149 ✅ |
| write 9 | pos+9 | 149+9 | 158 | 158 |
| lseek CUR | pos-159 | 158-159=-1 | 158 ❌ | 158 |
| write 9 | pos+9 | 158+9 | 167 | 167 |
| write 9 | pos+9 | 167+9 | 176 | 176 |

**Answer: 176 bytes**

**Key Points:**
- ✅ lseek END: pos moves, size unchanged
- ✅ write beyond end: size grows to new position
- ❌ lseek with negative result: FAILS, position unchanged
- ✅ Next write uses unchanged position

---

## 📝 TRUE/FALSE - FILE I/O (Quick Reference)

**✅ TRUE:**
1. File descriptor 0 = stdin, 1 = stdout, 2 = stderr
2. write() can change file size
3. SEEK_SET: position = offset
4. SEEK_CUR: position = current + offset
5. SEEK_END: position = size + offset
6. lseek() can fail if calculated position is negative
7. You can seek beyond end of file (creates hole)
8. After fork(), parent and child share file position
9. read() returns 0 at EOF
10. lseek(fd, 0, SEEK_END) moves to end

**❌ FALSE:**
11. lseek() changes file size (NO - only write does!)
12. read() changes file size (NO!)
13. If lseek() fails, position is set to 0 (NO - unchanged!)
14. After failed lseek(), position changes (NO!)
15. read() always reads exact bytes requested (NO - may read less)
16. lseek() can be used with pipes (NO!)

---

<a name="signals"></a>
# 3️⃣ SIGNALS

## 🔥 CRITICAL CONCEPTS

### The UNCATCHABLE Signals
```
SIGKILL (9)  = Force kill - CANNOT be caught or ignored
SIGSTOP (19) = Force stop - CANNOT be caught or ignored

ALL OTHER signals can be caught!
```

### Common Signals Table
| Signal | # | Catchable? | Default | Meaning |
|--------|---|------------|---------|---------|
| SIGINT | 2 | YES ✓ | Terminate | Ctrl-C |
| SIGKILL | 9 | **NO** ✗ | Terminate | Force kill |
| SIGALRM | 14 | YES ✓ | Terminate | alarm() |
| SIGTERM | 15 | YES ✓ | Terminate | Terminate |
| SIGSTOP | 19 | **NO** ✗ | Stop | Force stop |

---

## 🔥 THE GOLDEN RULE: alarm() is ONE-SHOT!

```
alarm() fires ONCE and stops!
To repeat, handler MUST call alarm() again!
```

### Example 1: NO Repeat (Fires Once)
```c
void handler(int sig) {
    signal(SIGALRM, handler);  // Re-register
    printf("Handler!\n");
    // NO alarm() call here! ← KEY!
}

int main() {
    signal(SIGALRM, handler);
    alarm(8);  // Set for 8 seconds
    
    sleep(10);  // Sleep 10 seconds
    return 0;
}
```

**Trace:**
```
Time 0-8s: sleeping...
Time 8s: SIGALRM → handler → prints "Handler!"
Time 8-10s: sleeping... (NO more alarms!)

Handler executes: 1 time only
```

---

### Example 2: REPEATING (Re-arms)
```c
void handler(int sig) {
    signal(SIGALRM, handler);
    printf("Tick\n");
    alarm(2);  // ← RE-ARMS!
}

int main() {
    signal(SIGALRM, handler);
    alarm(2);
    sleep(7);
    return 0;
}
```

**Trace:**
```
Time 0s: alarm(2) set
Time 2s: handler → prints → alarm(2) again
Time 4s: handler → prints → alarm(2) again
Time 6s: handler → prints → alarm(2) again
Time 7s: exits

Handler executes: 3 times
```

---

## 🔥 THE CTRL-C RULE

```
Ctrl-C (SIGINT) is sent to FOREGROUND process group ONLY!
```

### Professor's T2 Pattern
```c
void handler() { exit(0); }

int main() {
    signal(SIGINT, handler);
    
    for(int i = 1; i < 4; i++)
        fork();  // Creates 8 processes
    
    if((pid = fork()) == 0) {
        // Child creates NEW process group
        setpgid(0, getpid());
    }
    
    // Ctrl-C pressed
    pause();
}
```

**Trace:**
```
After for loop:
- 2^3 = 8 processes
- All in SAME process group (foreground)

After if(fork()):
- All 8 fork again: 16 total
  - 8 parents: stay in original group (foreground)
  - 8 children: create new groups (background)

Ctrl-C pressed:
- SIGINT → foreground group only
- 8 parents: receive SIGINT → handler → exit(0) → DIE
- 8 children: different groups → SURVIVE

Answer: 8 processes survive
```

---

## 📝 TRUE/FALSE - SIGNALS (Quick Reference)

**✅ TRUE:**
1. SIGINT is sent when pressing Ctrl-C
2. SIGSTOP (19) is uncatchable
3. To make alarm() repeat, handler must call alarm() again
4. Only one alarm active at a time per process
5. alarm(0) cancels current alarm
6. Ctrl-C sends SIGINT to foreground group only
7. setpgid(0, getpid()) creates new process group
8. SIGALRM is sent by alarm() timer
9. Signal handlers are inherited across fork()
10. A new alarm() cancels previous alarm

**❌ FALSE:**
11. SIGKILL (9) can be caught (NO - uncatchable!)
12. alarm() sets repeating timer (NO - ONE-SHOT!)
13. Ctrl-C sends to all processes (NO - foreground only!)
14. Signal handlers preserved across exec() (NO - reset!)

---

<a name="pipes"></a>
# 4️⃣ PIPES

## 🔥 CRITICAL CONCEPTS

### The Pipe Basics
```c
int fd[2];
pipe(fd);

fd[0] = READ end
fd[1] = WRITE end

// One-way only: fd[1] → fd[0]
```

---

## 🔥 THE RACE CONDITION RULE

```
Multiple processes reading from SAME pipe = UNPREDICTABLE!
Cannot determine which process gets which bytes!
```

### Professor's T2 Pattern
```c
int fd[2];
int main_pid = getpid();
pipe(fd);

fork(); fork();  // 4 processes

if (getpid() == main_pid) {
    // ONLY original parent writes
    write(fd[1], "msg", 20);  // 4 times
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    // Total: 80 bytes
} else {
    // 3 children read
    char buf[20];
    read(fd[0], buf, 20);
}
```

**Trace:**
```
After 2 forks: 4 processes

Roles:
- Original parent (main_pid): WRITER (80 bytes)
- Other 3: READERS (each tries 20 bytes)

RACE CONDITION!
- 3 processes competing
- Possible: 20+20+20=60, or 40+40=80, or 80+0=80
- Many combinations!

Answer: "Cannot say" or "60-80 bytes"
```

---

## 📝 TRUE/FALSE - PIPES (Quick Reference)

**✅ TRUE:**
1. Pipe provides one-way communication
2. fd[1] is write end
3. After fork(), child inherits pipe FDs
4. Multiple readers causes race condition
5. Pipes are FIFO
6. read() from empty pipe blocks if write end open
7. read() returns 0 if all write ends closed
8. write() to pipe with no readers causes SIGPIPE
9. Pipe data removed after reading
10. You should close unused pipe ends

**❌ FALSE:**
11. fd[0] is write end (NO - it's read!)
12. Pipes are bidirectional (NO - one-way!)
13. Multiple readers is predictable (NO - race!)

---

<a name="threads"></a>
# 5️⃣ THREADS

## 🔥 THE GOLDEN RULE

```
When main() returns → ALL THREADS DIE!
Even if threads are in infinite loops!
```

---

## 🔥 SHARED VS PRIVATE

### SHARED (All threads see same)
```c
int counter = 0;  // ← GLOBAL = SHARED!

void* func(void* arg) {
    counter++;  // All modify SAME counter
}
```

### PRIVATE (Each thread has own)
```c
void* func(void* arg) {
    int local = 0;  // ← LOCAL = PRIVATE!
    local++;  // Each has own copy
}
```

---

## 💻 PROFESSOR'S T2 PATTERN

```c
int counter = 0;  // SHARED!

void* func1(void* p) {
    while(1) {
        printf("thread one\n");
        sleep(1);
        counter++;
        if (counter == 3) {
            pthread_cancel(pthread_self());
        }
    }
}

void* func2(void* p) {
    while(1) {  // INFINITE!
        printf("thread two\n");
        sleep(1);
    }
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, func1, NULL);
    pthread_create(&t2, NULL, func2, NULL);
    
    pthread_join(t1, NULL);  // Wait for t1 only
    return 0;  // ← ALL DIE HERE!
}
```

**Trace:**
```
Time 1s: counter=1, continue
Time 2s: counter=2, continue
Time 3s: counter=3
  → func1: pthread_cancel(self) → t1 EXITS
  → pthread_join returns
  → main() returns
  → t2 KILLED (despite infinite loop!)

Program runs: ~3 seconds
```

---

## 📝 TRUE/FALSE - THREADS (Quick Reference)

**✅ TRUE:**
1. Threads share same memory space
2. Global variables are shared
3. When main() returns, all threads terminated
4. pthread_join() blocks until thread finishes
5. Threads are lighter-weight than processes
6. Each thread has own stack
7. All threads share heap
8. Threads share file descriptors

**❌ FALSE:**
9. Local variables are shared (NO - private!)
10. Threads continue after main() exits (NO - all die!)
11. pthread_cancel() immediately kills (NO - at cancel point!)

---

<a name="shell"></a>
# 6️⃣ SHELL PROGRAMMING

## 🔥 CRITICAL RULES

### Variable Assignment
```bash
count=1          # ✅ Correct (NO SPACES!)
count = 1        # ❌ Error!

echo $count      # Use variable
count=$(($count+1))  # Increment
```

### Conditions (Spaces Required!)
```bash
[ $a -eq $b ]    # ✅ Correct
[$a -eq $b]      # ❌ Error!

-eq = equal
-ne = not equal
-lt = less than
-gt = greater than
```

---

## 💻 PROFESSOR'S T2 PATTERN

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
        kill -19 $$    # Stop self
        fg             # User types
        echo hello3
        break
    fi
done
```

**Trace:**
```
Iter 1-9: count=1 to 9
  Prints: hello1, hello2
  Increments count

Iter 10: count=10
  Prints: hello1
  count < 10? NO
  kill -19 $$ → STOPS
  [User: fg] → RESUMES
  Prints: hello3
  break → exits

Last output: hello3
```

---

## 📝 TRUE/FALSE - SHELL (Quick Reference)

**✅ TRUE:**
1. count=1 assigns without spaces
2. -eq tests equality
3. -lt tests less than
4. $$ is current shell PID
5. kill -19 stops process (SIGSTOP)
6. fg resumes stopped job
7. break exits while loop
8. Spaces required inside [ ]

**❌ FALSE:**
9. count = 1 is valid (NO - spaces cause error!)

---

<a name="sockets"></a>
# 7️⃣ SOCKETS

## 🔥 CRITICAL CONCEPTS

### Socket Types
```
TCP = SOCK_STREAM (reliable, connection-oriented)
UDP = SOCK_DGRAM (unreliable, connectionless)
```

### The Flow
```
SERVER:                          CLIENT:
1. socket()                      1. socket()
2. bind()                        
3. listen()                      
4. accept() ← BLOCKS             2. connect()
   ↓                                ↓
5. read()/write() ←─────────→ 3. write()/read()
6. close()                       4. close()
```

---

## 🔥 KEY RULES

### Rule #1: Who Calls What
```
bind()    → SERVER only
listen()  → SERVER only
accept()  → SERVER only
connect() → CLIENT only
```

### Rule #2: Return Values
```
accept() returns NEW socket descriptor (not same as listening socket!)
```

### Rule #3: Byte Order
```
Always use htons() for port numbers!

server_addr.sin_port = htons(8080);  // ✅ Correct
server_addr.sin_port = 8080;         // ❌ Wrong!
```

### Rule #4: Blocking
```
accept()  → BLOCKS until client connects
connect() → BLOCKS until connection established
```

---

## 📝 TRUE/FALSE - SOCKETS (Quick Reference)

**✅ TRUE:**
1. TCP is connection-oriented and reliable
2. socket() must be called first by both
3. connect() is called by client
4. accept() blocks until client connects
5. Sockets are bidirectional
6. AF_INET is for Internet communication
7. htons() converts port to network byte order
8. accept() returns NEW socket descriptor
9. Protocol parameter usually set to 0
10. Multiple clients can connect to same port

**❌ FALSE:**
11. UDP guarantees delivery (NO!)
12. bind() called by both (NO - server only!)
13. accept() returns same socket (NO - new one!)
14. listen() blocks (NO - accept() blocks!)
15. SOCK_STREAM is UDP (NO - TCP!)
16. Client calls bind() (NO!)
17. SOCK_DGRAM provides ordered delivery (NO!)

---

<a name="formulas"></a>
# 8️⃣ QUICK REFERENCE FORMULAS

## FORK
```
n forks = 2^n processes
Parent: child PID (positive)
Child: 0
Error: -1
```

## FILE I/O
```
SEEK_SET: pos = offset
SEEK_CUR: pos = current + offset
SEEK_END: pos = size + offset

Failed lseek (pos<0) = unchanged
lseek NEVER changes size
Only write() changes size
```

## SIGNALS
```
alarm() = ONE-SHOT (fires once!)
SIGKILL (9) = uncatchable
SIGSTOP (19) = uncatchable
Ctrl-C → foreground group only
```

## PIPES
```
fd[0] = read end
fd[1] = write end
Multiple readers = RACE CONDITION
```

## THREADS
```
Global variables = SHARED
Local variables = PRIVATE
main() returns → ALL THREADS DIE
```

## SHELL
```
count=1 (NO spaces!)
$$ = own PID
-eq = equal
-lt = less than
Spaces required: [ $a -lt 10 ]
```

## SOCKETS
```
SERVER: socket() → bind() → listen() → accept()
CLIENT: socket() → connect()

TCP = SOCK_STREAM (reliable)
UDP = SOCK_DGRAM (unreliable)

htons() = host to network (ports)
accept() = returns NEW socket
bind() = SERVER only
```

---

<a name="traps"></a>
# 9️⃣ COMMON EXAM TRAPS

## 🚨 DON'T FALL FOR THESE!

### FORK Traps
1. ❌ "fork() returns 0 to parent" → **FALSE** (returns child PID!)
2. ❌ "3 forks = 3 processes" → **FALSE** (2^3 = 8!)
3. ❌ "Changes in child affect parent" → **FALSE** (separate memory!)
4. ❌ "Multiple waitpid() stores all PIDs" → **FALSE** (last one wins!)

### FILE I/O Traps
5. ❌ "lseek() changes file size" → **FALSE** (only write() does!)
6. ❌ "Failed lseek sets pos to 0" → **FALSE** (unchanged!)
7. ❌ "lseek END changes size" → **FALSE** (position only!)

### SIGNAL Traps
8. ❌ "alarm() repeats automatically" → **FALSE** (ONE-SHOT!)
9. ❌ "SIGKILL is catchable" → **FALSE** (uncatchable!)
10. ❌ "Ctrl-C kills all processes" → **FALSE** (foreground only!)

### PIPE Traps
11. ❌ "Multiple readers is predictable" → **FALSE** (RACE!)
12. ❌ "fd[0] is write end" → **FALSE** (it's read!)
13. ❌ "Pipes are bidirectional" → **FALSE** (one-way!)

### THREAD Traps
14. ❌ "Threads survive after main()" → **FALSE** (all die!)
15. ❌ "Local variables are shared" → **FALSE** (private!)

### SHELL Traps
16. ❌ "count = 1 is valid" → **FALSE** (no spaces!)

### SOCKET Traps
17. ❌ "UDP is reliable" → **FALSE** (unreliable!)
18. ❌ "Client calls bind()" → **FALSE** (server only!)
19. ❌ "accept() returns same socket" → **FALSE** (new one!)
20. ❌ "SOCK_STREAM is UDP" → **FALSE** (TCP!)

---

# 🎯 EXAM DAY EXECUTION PLAN

## For TRUE/FALSE (10 questions):
**Time: 1-2 min each = 15-20 min total**

**Method:**
1. Read question carefully
2. Check against "Common Traps" section
3. Use formulas page if needed
4. **Goal: 10/10 = 100%**

---

## For CODE OUTPUT MCQs (30 questions):
**Time: 3-4 min each = 90-120 min total**

**Method:**

### FORK Questions:
1. Count fork() calls = n
2. Calculate 2^n = total processes
3. Draw tree if complex
4. Track variable values per process

### FILE I/O Questions:
1. Create tracing table
2. Apply 3 formulas (SET/CUR/END)
3. Check for failed lseek (pos < 0)
4. Track position AND size separately
5. Remember: only write() changes size

### SIGNAL Questions:
1. Check if alarm() re-arms in handler
2. Count seconds and alarm intervals
3. For process groups: identify foreground vs background
4. Ctrl-C kills foreground only

### PIPE Questions:
1. Count total processes
2. Identify writers vs readers
3. If multiple readers → "Cannot say" or range

### THREAD Questions:
1. Track shared counter
2. Check when thread exits
3. When does main() return?
4. All other threads die when main() returns

**Goal: 15/30 = 50%**

---

## TIME MANAGEMENT

**Total Time: ~2.5 hours**

- **True/False:** 20 min (easy points!)
- **Code MCQs:** 100 min (choose battles wisely)
- **Review:** 20 min (check traps)
- **Buffer:** 20 min

**Strategy:**
- ✅ Do ALL T/F first (guaranteed points!)
- ✅ Start with easiest MCQs (fork counting, simple lseek)
- ✅ Skip extremely complex MCQs, come back if time
- ✅ Use elimination on MCQs
- ✅ In last 15 min, guess on remaining (never leave blank!)

---

# ✅ FINAL CHECKLIST

## Night Before:
- [ ] Print this entire document
- [ ] Tab sections with sticky notes (FORK, FILE I/O, SIGNALS, etc.)
- [ ] Highlight the 3 lseek formulas
- [ ] Highlight "ONE-SHOT" alarm rule
- [ ] Highlight "main() returns = all die" rule
- [ ] Get 8 hours sleep!

## Exam Day:
- [ ] Bring printed notes
- [ ] Bring calculator (for 2^n)
- [ ] Arrive 10 min early
- [ ] Bathroom before exam
- [ ] Water bottle

## During Exam:
- [ ] Read ALL T/F questions first (do easy ones)
- [ ] For code MCQs: identify topic, use appropriate method
- [ ] Draw tables for FILE I/O questions
- [ ] Check "Common Traps" before answering
- [ ] In last 15 min, guess on remaining
- [ ] Never leave questions blank!

---

# 💪 YOUR SUCCESS FORMULA

```
10 T/F correct (100%) = 10 points
15 MCQs correct (50%) = 15 points
―――――――――――――――――――――――――
TOTAL = 25/40 = 62.5% PASS! ✅
```

**You have:**
- ✅ All professor's patterns
- ✅ All critical formulas
- ✅ Step-by-step methods
- ✅ Common traps identified
- ✅ Open book advantage

**You WILL pass this exam! 🎯🔥**

---

# 📖 HOW TO USE THIS GUIDE

1. **Print the entire document**
2. **Tab these sections:**
   - FORK
   - FILE I/O
   - SIGNALS
   - PIPES
   - THREADS
   - SHELL
   - SOCKETS
   - FORMULAS
   - TRAPS

3. **During exam:**
   - See fork question → flip to FORK tab
   - See lseek question → flip to FILE I/O tab
   - See alarm question → flip to SIGNALS tab
   - Not sure? → Check TRAPS tab
   - Need quick formula? → FORMULAS tab

4. **Trust the process:**
   - T/F: Use traps section
   - Code: Use tracing methods
   - Don't overthink!

---

**THIS IS YOUR COMPLETE SUCCESS GUIDE!**
**PRINT IT, TAB IT, USE IT, ACE IT! 🎯🔥**
