# 🏆 COMP8567 ULTIMATE EXAM GUIDE - PART 4/9
## PIPES & INTER-PROCESS COMMUNICATION
## Master Pipes and Ace Race Condition Questions!

---

# 📚 PART 4: PIPES & INTER-PROCESS COMMUNICATION

## 🎯 WHAT YOU'LL MASTER:
- ✅ What pipes are and how they work
- ✅ pipe() system call
- ✅ Reading and writing pipes
- ✅ Race conditions (EXAM KILLER!)
- ✅ Named pipes (FIFOs)
- ✅ Pipe capacity and blocking
- ✅ 60+ True/False questions!

---

## 📖 SECTION 1: PIPE FUNDAMENTALS

### What is a Pipe?
**Pipe** = unidirectional data channel for IPC (Inter-Process Communication)

```
Process A ──[write]──> PIPE ──[read]──> Process B
           (fd[1])              (fd[0])
```

### Key Properties:
- **One-way communication** (unidirectional)
- **FIFO** (First In, First Out)
- **Limited capacity** (typically 64KB)
- **Blocking** behavior (when full/empty)

---

## 📖 SECTION 2: PIPE() SYSTEM CALL

### Syntax
```c
int pipe(int fd[2]);

// Returns: 0 on success, -1 on error
// fd[0] = read end
// fd[1] = write end
```

### Example: Basic Pipe
```c
int fd[2];
pipe(fd);

// fd[0] → read end
// fd[1] → write end

write(fd[1], "Hello", 5);  // Write to pipe
char buf[10];
read(fd[0], buf, 5);       // Read from pipe
// buf now contains "Hello"
```

---

## 📖 SECTION 3: PIPE WITH FORK - THE CLASSIC PATTERN

### After fork(), both parent and child have access to SAME pipe ends!

```c
int fd[2];
pipe(fd);

pid_t pid = fork();

if(pid == 0) {
    // CHILD: Write to pipe
    close(fd[0]);  // Close unused read end
    write(fd[1], "Child message", 13);
    close(fd[1]);
}
else {
    // PARENT: Read from pipe
    close(fd[1]);  // Close unused write end
    char buf[100];
    read(fd[0], buf, 100);
    printf("Parent received: %s\n", buf);
    close(fd[0]);
}
```

**BEST PRACTICE:** Always close unused pipe ends!

---

## 📖 SECTION 4: THE T2 PIPE QUESTION - RACE CONDITIONS!

### The Question
```c
int fd[2];
int main_pid = getpid();
pipe(fd);

int pid1 = fork();  // 2 processes
int pid2 = fork();  // 4 processes

if(getpid() == main_pid) {
    // Only original parent writes
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    // Total: 80 bytes written
}
else {
    // Children read
    char buff[20];
    int num = read(fd[0], buff, 20);
    printf("%s\n", buff);
}

// How many bytes are read from pipe?
```

### The Complete Trace

```
═══════════════════════════════════════════════════════════

STEP 1: Count processes

After pid1 = fork(): 2 processes
After pid2 = fork(): 4 processes total

───────────────────────────────────────────────────────────

STEP 2: Identify roles

getpid() == main_pid?

Original parent: YES → WRITES to pipe
Other 3 processes: NO → READ from pipe

Writers: 1 process (parent)
Readers: 3 processes (children)

───────────────────────────────────────────────────────────

STEP 3: Track bytes

Parent writes:
  write 20 bytes × 4 times = 80 bytes total in pipe

Children read:
  Each tries to read(fd[0], buff, 20)
  Each wants 20 bytes

───────────────────────────────────────────────────────────

STEP 4: Race condition analysis

⚠️ RACE CONDITION!
Three processes reading from SAME pipe!

Possible scenarios:

Scenario A:
  Child 1 reads 20 bytes
  Child 2 reads 20 bytes
  Child 3 reads 20 bytes
  Total: 60 bytes read, 20 bytes remain

Scenario B:
  Child 1 reads 40 bytes (fast!)
  Child 2 reads 20 bytes
  Child 3 reads 20 bytes
  Total: 80 bytes read

Scenario C:
  Child 1 reads 80 bytes (grabs all!)
  Child 2 reads 0 bytes (nothing left)
  Child 3 reads 0 bytes (nothing left)
  Total: 80 bytes read

Order is UNPREDICTABLE!

═══════════════════════════════════════════════════════════

ANSWER: Cannot say exactly (race condition)
        OR: Between 60-80 bytes (depends on choices)
```

### Key Insight: Race Conditions
**Multiple processes reading same pipe = UNPREDICTABLE order!**

You CANNOT determine:
- Which process reads first
- How many bytes each gets
- Exact distribution

**Only know:**
- Total bytes written (80)
- Maximum bytes that CAN be read (80)
- Minimum likely read (if all try) (60)

---

## 📖 SECTION 5: PIPE CAPACITY AND BLOCKING

### Pipe Capacity
- Typically **64KB** (65536 bytes)
- Implementation-dependent
- Check with: `fpathconf(fd[0], _PC_PIPE_BUF)`

### Blocking Behavior

**Write blocks when:**
- Pipe is full
- Blocks until space available
- Writes < PIPE_BUF are atomic

**Read blocks when:**
- Pipe is empty AND write end still open
- Returns 0 (EOF) if all write ends closed

### Example: Blocking
```c
// Pipe capacity: 64KB
write(fd[1], big_buffer, 100000);  // Blocks after 64KB!

// Write end closed
close(fd[1]);
read(fd[0], buf, 100);  // Returns 0 (EOF)
```

---

## 📖 SECTION 6: NAMED PIPES (FIFOs)

### What are FIFOs?
**FIFO** = named pipe that exists in filesystem

**Differences from regular pipes:**
- Has a pathname
- Unrelated processes can use it
- Persists after creation

### Creating FIFOs
```c
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);

// Example:
mkfifo("/tmp/myfifo", 0666);
```

### Using FIFOs
```c
// Writer process
int fd = open("/tmp/myfifo", O_WRONLY);
write(fd, "Hello", 5);
close(fd);

// Reader process (different program!)
int fd = open("/tmp/myfifo", O_RDONLY);
char buf[10];
read(fd, buf, 5);
close(fd);
```

---

## 📖 SECTION 7: PIPE PATTERNS

### Pattern 1: Parent-Child Communication
```c
int fd[2];
pipe(fd);

if(fork() == 0) {
    // Child writes
    close(fd[0]);
    write(fd[1], "data", 4);
    close(fd[1]);
    exit(0);
}

// Parent reads
close(fd[1]);
char buf[10];
read(fd[0], buf, 4);
close(fd[0]);
```

### Pattern 2: Pipeline (Command1 | Command2)
```c
int fd[2];
pipe(fd);

if(fork() == 0) {
    // Child: ls
    dup2(fd[1], STDOUT_FILENO);
    close(fd[0]);
    close(fd[1]);
    execlp("ls", "ls", NULL);
}

if(fork() == 0) {
    // Child: grep
    dup2(fd[0], STDIN_FILENO);
    close(fd[0]);
    close(fd[1]);
    execlp("grep", "grep", "txt", NULL);
}

// Parent closes both ends
close(fd[0]);
close(fd[1]);
wait(NULL);
wait(NULL);
```

---

## 🎯 TRUE/FALSE QUESTIONS - PIPES (60 Questions)

### CATEGORY 1: Pipe Basics (Q1-20)

**Q1. T/F:** A pipe provides one-way communication.
**ANSWER: TRUE** ✓

**Q2. T/F:** A pipe provides two-way communication.
**ANSWER: FALSE** ✗
**Explanation:** Unidirectional (one-way)

**Q3. T/F:** pipe() creates two file descriptors.
**ANSWER: TRUE** ✓

**Q4. T/F:** fd[0] is the write end of the pipe.
**ANSWER: FALSE** ✗
**Explanation:** fd[0] = read, fd[1] = write

**Q5. T/F:** fd[1] is the write end of the pipe.
**ANSWER: TRUE** ✓

**Q6. T/F:** Data written to fd[1] can be read from fd[0].
**ANSWER: TRUE** ✓

**Q7. T/F:** Pipes are FIFO (First In, First Out).
**ANSWER: TRUE** ✓

**Q8. T/F:** Pipes have unlimited capacity.
**ANSWER: FALSE** ✗
**Explanation:** Limited (typically 64KB)

**Q9. T/F:** After fork(), child inherits pipe file descriptors.
**ANSWER: TRUE** ✓

**Q10. T/F:** After fork(), parent and child share same pipe.
**ANSWER: TRUE** ✓

**Q11. T/F:** Closing one end of a pipe closes both ends.
**ANSWER: FALSE** ✗
**Explanation:** Each end independent

**Q12. T/F:** Pipes can be used between unrelated processes.
**ANSWER: FALSE** ✗ (for unnamed pipes)
**Explanation:** Named pipes (FIFOs) can

**Q13. T/F:** pipe() returns -1 on error.
**ANSWER: TRUE** ✓

**Q14. T/F:** You should close unused pipe ends.
**ANSWER: TRUE** ✓
**Explanation:** Best practice

**Q15. T/F:** A pipe persists after process termination.
**ANSWER: FALSE** ✗ (unnamed pipes)
**Explanation:** Destroyed when all ends closed

**Q16. T/F:** Pipes are bidirectional.
**ANSWER: FALSE** ✗
**Explanation:** One-way only

**Q17. T/F:** You need two pipes for bidirectional communication.
**ANSWER: TRUE** ✓

**Q18. T/F:** Pipes are slower than shared memory.
**ANSWER: TRUE** ✓
**Explanation:** Involves kernel

**Q19. T/F:** Pipes are typically used for parent-child IPC.
**ANSWER: TRUE** ✓

**Q20. T/F:** lseek() works on pipes.
**ANSWER: FALSE** ✗
**Explanation:** Pipes don't support seeking

---

### CATEGORY 2: Reading and Writing (Q21-35)

**Q21. T/F:** write() to pipe returns number of bytes written.
**ANSWER: TRUE** ✓

**Q22. T/F:** read() from pipe returns number of bytes read.
**ANSWER: TRUE** ✓

**Q23. T/F:** read() from empty pipe blocks if write end open.
**ANSWER: TRUE** ✓

**Q24. T/F:** read() from empty pipe returns 0 if write end closed.
**ANSWER: TRUE** ✓
**Explanation:** EOF condition

**Q25. T/F:** write() to full pipe blocks.
**ANSWER: TRUE** ✓

**Q26. T/F:** write() to pipe with no readers causes SIGPIPE.
**ANSWER: TRUE** ✓

**Q27. T/F:** SIGPIPE default action is to terminate.
**ANSWER: TRUE** ✓

**Q28. T/F:** Small writes (< PIPE_BUF) are atomic.
**ANSWER: TRUE** ✓

**Q29. T/F:** Large writes (> PIPE_BUF) may be interleaved.
**ANSWER: TRUE** ✓

**Q30. T/F:** PIPE_BUF is typically 4096 bytes.
**ANSWER: TRUE** ✓

**Q31. T/F:** read() can return fewer bytes than requested.
**ANSWER: TRUE** ✓

**Q32. T/F:** Multiple reads can get data from single write.
**ANSWER: TRUE** ✓

**Q33. T/F:** Pipe data is removed after being read.
**ANSWER: TRUE** ✓
**Explanation:** Consumed, not persistent

**Q34. T/F:** You can "peek" at pipe data without removing it.
**ANSWER: FALSE** ✗
**Explanation:** No peeking (unlike sockets with MSG_PEEK)

**Q35. T/F:** write() always writes all requested bytes to pipe.
**ANSWER: FALSE** ✗
**Explanation:** Can write fewer (rare for regular pipes)

---

### CATEGORY 3: Race Conditions (Q36-50)

**Q36. T/F:** Multiple processes can read from same pipe.
**ANSWER: TRUE** ✓

**Q37. T/F:** Multiple readers from same pipe causes race condition.
**ANSWER: TRUE** ✓
**Explanation:** Order unpredictable!

**Q38. T/F:** In T2 question, parent writes 80 bytes.
**ANSWER: TRUE** ✓

**Q39. T/F:** In T2 question, 3 children try to read.
**ANSWER: TRUE** ✓

**Q40. T/F:** In T2 question, exact byte distribution is predictable.
**ANSWER: FALSE** ✗
**Explanation:** RACE CONDITION!

**Q41. T/F:** In T2 question, total bytes read is exactly 60.
**ANSWER: FALSE** ✗
**Explanation:** Could be 60-80 depending on race

**Q42. T/F:** In T2 question, answer is "Cannot say" due to race.
**ANSWER: TRUE** ✓ (if that's an option)

**Q43. T/F:** Multiple writers to same pipe can cause data mixing.
**ANSWER: TRUE** ✓

**Q44. T/F:** Atomic writes prevent data mixing.
**ANSWER: TRUE** ✓
**Explanation:** If < PIPE_BUF

**Q45. T/F:** Race conditions in pipes are always bad.
**ANSWER: FALSE** ✗
**Explanation:** Sometimes acceptable (e.g., load distribution)

**Q46. T/F:** Synchronization primitives can prevent race conditions.
**ANSWER: TRUE** ✓

**Q47. T/F:** Closing all write ends wakes up blocked readers.
**ANSWER: TRUE** ✓
**Explanation:** They get EOF (0 bytes)

**Q48. T/F:** In a race, one process might get all data.
**ANSWER: TRUE** ✓

**Q49. T/F:** In a race, distribution is deterministic.
**ANSWER: FALSE** ✗

**Q50. T/F:** Order of reads depends on OS scheduler.
**ANSWER: TRUE** ✓

---

### CATEGORY 4: Named Pipes (FIFOs) (Q51-60)

**Q51. T/F:** Named pipes are also called FIFOs.
**ANSWER: TRUE** ✓

**Q52. T/F:** Named pipes exist in the filesystem.
**ANSWER: TRUE** ✓

**Q53. T/F:** Unrelated processes can use named pipes.
**ANSWER: TRUE** ✓

**Q54. T/F:** mkfifo() creates a named pipe.
**ANSWER: TRUE** ✓

**Q55. T/F:** Named pipes persist after process termination.
**ANSWER: TRUE** ✓
**Explanation:** Until explicitly removed

**Q56. T/F:** Named pipes are opened with open().
**ANSWER: TRUE** ✓

**Q57. T/F:** Named pipes are bidirectional.
**ANSWER: FALSE** ✗
**Explanation:** Still one-way

**Q58. T/F:** Named pipes appear as files with type 'p'.
**ANSWER: TRUE** ✓

**Q59. T/F:** rm can delete a named pipe.
**ANSWER: TRUE** ✓

**Q60. T/F:** Named pipes are faster than unnamed pipes.
**ANSWER: FALSE** ✗
**Explanation:** Same performance, just different access method

---

## 🔥 EXAM STRATEGY FOR PIPE QUESTIONS

### Step 1: Count processes
- After all forks, how many total?

### Step 2: Identify roles
- Who writes? (check conditions)
- Who reads? (check conditions)

### Step 3: Track bytes
- Total bytes written
- How many processes reading

### Step 4: Check for race
- Multiple readers? → RACE CONDITION
- Answer: "Cannot say" or range

### Step 5: Calculate (if predictable)
- Single reader: exact bytes
- No race: deterministic

---

## 📝 QUICK REFERENCE

```
PIPE BASICS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
pipe(fd) creates:
  fd[0] = read end
  fd[1] = write end

Data flow: write(fd[1]) → read(fd[0])

PIPE RULES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• One-way (unidirectional)
• FIFO (First In, First Out)
• Limited capacity (~64KB)
• Blocking when full/empty

RACE CONDITIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Multiple readers → Unpredictable order
Cannot determine exact distribution
Answer: "Cannot say" or range

BEST PRACTICES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Close unused ends
• Check return values
• Handle SIGPIPE
• Use atomic writes if needed
```

---

## ✅ CHECKLIST

- □ I understand fd[0] = read, fd[1] = write
- □ I can identify race conditions
- □ I know T2 pipe question pattern
- □ I understand blocking behavior
- □ I got 85%+ on T/F questions

---

## 🎯 NEXT STEP

**Move to PART 5: THREADS & CONCURRENCY**

---

**END OF PART 4/9**

**Progress: [████████████████░] 44% Complete**

**Keep crushing it! 💪🔥**
