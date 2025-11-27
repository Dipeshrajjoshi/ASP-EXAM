# 🎯 COMP8567 - ULTIMATE 14-HOUR EXAM CRASH COURSE
## Complete Syllabus | T/F Mastery | Code Output Tracing

---

## 📊 EXAM BREAKDOWN & STRATEGY

**Total: 40 Questions**
- **~10 True/False** (Goal: 10/10 = 100%) ← EASY MARKS!
- **~30 Code Output MCQs** (Goal: 15/30 = 50%)
- **Total Target: 25/40 = 62.5% SOLID PASS**

**Your Advantage:**
- ✅ OPEN BOOK (use notes for reference)
- ✅ Testing UNDERSTANDING, not memorization
- ✅ Focus on CODE TRACING skills

---

## 📅 YOUR 14-HOUR BATTLE PLAN

### Thursday, Nov 28
- **1:00-3:00 AM (2h):** FORK complete mastery
- **12:30-1:50 PM (1.3h):** FILE I/O deep dive
- **4:00-7:00 PM (3h):** SIGNALS + PIPES
- **8:00 PM-2:00 AM (6h):** THREADS + SHELL + Practice

### Friday, Nov 29
- **3:00-5:00 PM (2h):** Final review + T/F + MCQ drills
- **After 5:00 PM:** EXAM TIME! 🔥

---

# 📚 TOPIC 1: FORK & PROCESS CONTROL

## ⏰ Thursday 1:00-3:00 AM (2 HOURS)

---

## 🎓 CONCEPT: What is fork()?

**Simple Analogy:**
You're playing a video game. You press "Save Game" and create a save file. Now you have:
- **Original game (PARENT):** Continues playing
- **Saved game (CHILD):** Exact copy at that moment

Both exist independently. Changes in one don't affect the other.

```c
pid_t pid = fork();
// After this: TWO processes exist!
// Both continue from here
```

---

## 🔑 CRITICAL CONCEPT #1: Return Values

**The Golden Rule:**
```c
pid_t pid = fork();

// In PARENT process:
//   pid = child's PID (positive number, like 12345)

// In CHILD process:
//   pid = 0 (always zero)

// On ERROR:
//   pid = -1 (no child created)
```

**Visual Example:**
```c
int main() {
    printf("Before fork: One process\n");
    
    pid_t pid = fork();
    // ← MAGIC HAPPENS HERE: 1 becomes 2!
    
    printf("After fork: This prints TWICE!\n");
    
    if (pid > 0) {
        printf("I'm PARENT, child ID = %d\n", pid);
    }
    else if (pid == 0) {
        printf("I'm CHILD, my ID = %d\n", getpid());
    }
    
    return 0;
}
```

**Output:**
```
Before fork: One process
After fork: This prints TWICE!
I'm PARENT, child ID = 12345
After fork: This prints TWICE!
I'm CHILD, my ID = 12346
```

---

## 🔑 CRITICAL CONCEPT #2: Process Counting

**Rule:** Each fork() **DOUBLES** the number of processes.

**Formula:** n sequential forks = 2^n processes

**Example 1: Sequential Forks**
```c
fork();  // 1 → 2 processes
fork();  // 2 → 4 processes
fork();  // 4 → 8 processes

// Total: 2^3 = 8 processes
```

**Visual Tree:**
```
                    [P]
                     |
          ┌────────fork()────────┐
          |                      |
        [P]                    [C1]
          |                      |
    ┌───fork()───┐         ┌──fork()──┐
    |            |         |          |
  [P]          [C2]      [C1]       [C3]
    |            |         |          |
  fork()      fork()    fork()     fork()
    |            |         |          |
  [P][C4]    [C2][C5]  [C1][C6]   [C3][C7]

Final: 8 processes
```

**Example 2: Loop Forks**
```c
for(int i = 0; i < 4; i++) {
    fork();
}

// i=0: 1 → 2
// i=1: 2 → 4
// i=2: 4 → 8
// i=3: 8 → 16

// Total: 2^4 = 16 processes
```

**Example 3: Conditional Fork**
```c
if(fork()) {  // Parent: true (pid > 0), Child: false (pid = 0)
    fork();   // Only parent does this
}

// Process tree:
// Start: 1
// After 1st fork: 2 (parent + child)
// Parent does 2nd fork: +1
// Total: 3 processes
```

---

## 🔑 CRITICAL CONCEPT #3: Separate Memory

**After fork(), each process has its OWN COPY of memory.**

```c
int x = 5;
pid_t pid = fork();

if (pid == 0) {
    // CHILD modifies its copy
    x = 10;
    printf("Child: x = %d\n", x);  // 10
}
else {
    sleep(1);  // Wait for child to print
    printf("Parent: x = %d\n", x); // 5 (unchanged!)
}
```

**Analogy:** You and your twin each get a copy of a document. If twin writes on their copy, YOUR copy stays clean.

---

## 🔑 CRITICAL CONCEPT #4: waitpid() & Variable Overwriting

**waitpid() waits for a specific child to finish:**

```c
pid_t waitpid(pid_t pid, int *status, int options);
// Returns: PID of terminated child
```

**THE EXAM TRAP: Variable Overwriting**

```c
int main() {
    int pid1 = fork();
    int pid2 = fork();
    int pid3 = fork();
    
    if (pid1 > 0 && pid2 > 0 && pid3 > 0) {
        // Only original parent enters here
        
        int result;
        result = waitpid(pid1, NULL, 0);  // result = pid1 (e.g., 100)
        result = waitpid(pid2, NULL, 0);  // result = pid2 (e.g., 200) ← OVERWRITES!
        result = waitpid(pid3, NULL, 0);  // result = pid3 (e.g., 300) ← OVERWRITES AGAIN!
        
        printf("%d", result);  // Prints 300 (LAST value)
    }
    
    return 0;
}
```

**Why?** Each assignment REPLACES the previous value. Like writing on a whiteboard - new text erases old text.

---

## 📝 TRUE/FALSE QUESTIONS - FORK (20 Questions)

### T/F #1
**Q:** fork() creates a new process by duplicating the calling process.
**A:** TRUE ✓
**Why:** That's exactly what fork does - makes an exact copy.

---

### T/F #2
**Q:** fork() returns 0 to the parent process.
**A:** FALSE ✗
**Why:** Parent gets child's PID (positive). Child gets 0.

---

### T/F #3
**Q:** fork() returns the child's PID to the parent.
**A:** TRUE ✓
**Why:** Parent receives positive PID so it knows which child it created.

---

### T/F #4
**Q:** After fork(), both parent and child execute from the beginning of main().
**A:** FALSE ✗
**Why:** Both continue from the line AFTER fork().

---

### T/F #5
**Q:** Three sequential fork() calls create 8 processes total.
**A:** TRUE ✓
**Why:** 2^3 = 8

---

### T/F #6
**Q:** After fork(), parent and child share the same memory space.
**A:** FALSE ✗
**Why:** Each has separate memory (copy-on-write).

---

### T/F #7
**Q:** Changes to variables in child affect parent's variables.
**A:** FALSE ✗
**Why:** Separate memory - changes are independent.

---

### T/F #8
**Q:** getpid() returns the same value in parent and child.
**A:** FALSE ✗
**Why:** Each has unique PID.

---

### T/F #9
**Q:** getppid() in child returns the parent's PID.
**A:** TRUE ✓
**Why:** getppid() = get parent PID.

---

### T/F #10
**Q:** for(i=0; i<5; i++) fork() creates 32 processes.
**A:** TRUE ✓
**Why:** 2^5 = 32

---

### T/F #11
**Q:** A zombie process is one that has terminated but parent hasn't called wait().
**A:** TRUE ✓
**Why:** Definition of zombie - terminated but not reaped.

---

### T/F #12
**Q:** An orphan process is one whose parent has terminated.
**A:** TRUE ✓
**Why:** Parent died; child adopted by init.

---

### T/F #13
**Q:** Zombie processes consume CPU resources.
**A:** FALSE ✗
**Why:** Only occupy process table entry, no CPU/memory.

---

### T/F #14
**Q:** waitpid() blocks until the specified child terminates.
**A:** TRUE ✓
**Why:** That's its purpose - wait for specific child.

---

### T/F #15
**Q:** After fork(), child inherits open file descriptors.
**A:** TRUE ✓
**Why:** File descriptors are inherited.

---

### T/F #16
**Q:** Multiple waitpid() calls on the same variable store all return values.
**A:** FALSE ✗
**Why:** Variable overwriting - only LAST value remains.

---

### T/F #17
**Q:** fork() can fail and return -1.
**A:** TRUE ✓
**Why:** Fails if insufficient resources.

---

### T/F #18
**Q:** exec() creates a new process.
**A:** FALSE ✗
**Why:** Replaces current process image; same PID.

---

### T/F #19
**Q:** After exec(), the code following it executes.
**A:** FALSE ✗
**Why:** exec() never returns on success.

---

### T/F #20
**Q:** The order of execution between parent and child after fork() is guaranteed.
**A:** FALSE ✗
**Why:** OS scheduler decides - unpredictable.

---

## 💻 CODE OUTPUT MCQs - FORK (15 Questions)

### MCQ #1 - Basic Fork
```c
int main() {
    fork();
    printf("Hello");
    return 0;
}
```
**Q:** How many times is "Hello" printed?
- A) 1
- B) 2
- C) 4
- D) 8

**Answer:** B ✓
**Trace:**
- Start: 1 process
- After fork(): 2 processes
- Each prints "Hello" once
- Total: 2 times

---

### MCQ #2 - Return Value
```c
int main() {
    int pid = fork();
    
    if (pid == 0) {
        printf("A");
    } else {
        printf("B");
    }
    
    return 0;
}
```
**Q:** What is printed?
- A) A
- B) B
- C) AB or BA
- D) Nothing

**Answer:** C ✓
**Trace:**
- Child (pid=0): prints "A"
- Parent (pid>0): prints "B"
- Order unpredictable: "AB" or "BA"

---

### MCQ #3 - Counting
```c
int main() {
    fork();
    fork();
    printf("X");
    return 0;
}
```
**Q:** How many times is "X" printed?
- A) 2
- B) 3
- C) 4
- D) 8

**Answer:** C ✓
**Trace:**
- After 1st fork: 2 processes
- After 2nd fork: 4 processes
- Each prints "X" once
- Total: 4 times

---

### MCQ #4 - Variable Changes
```c
int main() {
    int x = 10;
    
    if (fork() == 0) {
        x = 20;
    }
    
    printf("%d ", x);
    return 0;
}
```
**Q:** What is printed?
- A) 10 10
- B) 20 20
- C) 10 20 or 20 10
- D) 30

**Answer:** C ✓
**Trace:**
- Parent: x stays 10, prints "10"
- Child: x becomes 20, prints "20"
- Order unpredictable: "10 20" or "20 10"

---

### MCQ #5 - Loop Fork
```c
int main() {
    int i;
    for (i = 0; i < 3; i++) {
        fork();
    }
    printf("Done");
    return 0;
}
```
**Q:** How many processes print "Done"?
- A) 3
- B) 6
- C) 8
- D) 16

**Answer:** C ✓
**Trace:**
- i=0: 1 → 2
- i=1: 2 → 4
- i=2: 4 → 8
- Total: 2^3 = 8 processes

---

### MCQ #6 - Conditional Fork
```c
int main() {
    if (fork()) {
        fork();
    }
    printf("Hi");
    return 0;
}
```
**Q:** How many times is "Hi" printed?
- A) 2
- B) 3
- C) 4
- D) 6

**Answer:** B ✓
**Trace:**
- 1st fork: 2 processes (parent + child1)
- Parent (pid>0): enters if, does 2nd fork → creates child2
- child1 (pid=0): skips if
- Final: parent, child1, child2 = 3
- Each prints "Hi"

---

### MCQ #7 - getpid()
```c
int main() {
    int pid = fork();
    
    if (pid > 0) {
        printf("%d ", getpid());
    } else {
        printf("%d ", getpid());
    }
    
    return 0;
}
```
**Q:** What is printed?
- A) Same PID twice
- B) Two different PIDs
- C) 0 and a positive number
- D) Two zeros

**Answer:** B ✓
**Trace:**
- Both processes print their own PID
- Each process has unique PID
- Two different PIDs printed

---

### MCQ #8 - Multiple Forks
```c
int main() {
    fork();
    fork();
    fork();
    return 0;
}
```
**Q:** How many processes exist after all forks?
- A) 3
- B) 6
- C) 8
- D) 16

**Answer:** C ✓
**Trace:**
- 2^3 = 8 processes

---

### MCQ #9 - waitpid() Overwriting
```c
int main() {
    int p1 = fork();
    int p2 = fork();
    
    if (p1 > 0 && p2 > 0) {
        int result;
        result = waitpid(p1, NULL, 0);
        result = waitpid(p2, NULL, 0);
        printf("%d", (result == p2));
    }
    
    return 0;
}
```
**Q:** What is printed?
- A) 0
- B) 1
- C) 2
- D) Nothing

**Answer:** B ✓
**Trace:**
- result = waitpid(p1) → result = p1
- result = waitpid(p2) → result = p2 (OVERWRITES!)
- result == p2 → TRUE → 1

---

### MCQ #10 - Complex Fork
```c
int main() {
    int count = 0;
    
    if (fork() == 0) {
        count++;
    }
    
    if (fork() == 0) {
        count++;
    }
    
    printf("%d ", count);
    return 0;
}
```
**Q:** What is the sum of all printed values?
- A) 0
- B) 2
- C) 4
- D) 6

**Answer:** C ✓
**Trace:**
After 1st fork (2 processes):
- Parent: count=0
- Child1: count=1

After 2nd fork (4 processes):
- Parent (from 1st): count=0, 2nd fork creates child → child increments → prints 1
- Parent (from 1st): count=0 → prints 0
- Child1: count=1, 2nd fork creates child → child increments → prints 2
- Child1: count=1 → prints 1

Printed: 0, 1, 1, 2
Sum: 0+1+1+2 = 4

---

### MCQ #11-15: Practice Problems

**MCQ #11:**
```c
fork() || fork();
```
How many processes?
**Answer:** 3
**Why:** Short-circuit evaluation: parent forks (true), child forks again

**MCQ #12:**
```c
fork() && fork();
```
How many processes?
**Answer:** 3
**Why:** Parent sees true AND does fork, child sees false AND stops

**MCQ #13:**
```c
int x = 0;
if (!fork()) x = 1;
printf("%d", x);
```
What's printed?
**Answer:** "0 1" or "1 0"
**Why:** Parent prints 0, child prints 1

**MCQ #14:**
```c
for(int i=0; i<2; i++)
    fork();
fork();
```
How many processes?
**Answer:** 8
**Why:** 2^2 = 4, then all 4 fork → 8

**MCQ #15:**
```c
if (fork() == 0)
    exit(0);
printf("X");
```
How many "X" printed?
**Answer:** 1 (only parent)
**Why:** Child exits immediately

---

# 📚 TOPIC 2: FILE I/O & lseek()

## ⏰ Thursday 12:30-1:50 PM (1 HOUR 20 MIN)

---

## 🎓 CONCEPT: File Descriptors

**Simple Analogy:** File descriptors are like "handles" or "remote controls" for files.

```
File Descriptor = Small integer pointing to an open file

Pre-opened:
- 0 = stdin (keyboard input)
- 1 = stdout (screen output)
- 2 = stderr (error messages)

Your files start at 3, 4, 5...
```

---

## 🔑 CRITICAL CONCEPT #1: open(), read(), write()

### open() - Opening Files
```c
int fd = open("file.txt", O_RDONLY);  // Read only
int fd = open("file.txt", O_WRONLY);  // Write only
int fd = open("file.txt", O_RDWR);    // Read and write

// Creating file:
int fd = open("file.txt", O_WRONLY | O_CREAT, 0644);

// Returns: file descriptor (positive) or -1 on error
```

### read() - Reading
```c
char buffer[100];
ssize_t n = read(fd, buffer, 100);
// Reads UP TO 100 bytes
// Returns: bytes actually read, or 0 at EOF, or -1 on error
// Position moves forward by n bytes
```

### write() - Writing
```c
char *msg = "Hello";
ssize_t n = write(fd, msg, 5);
// Writes 5 bytes
// Returns: bytes written or -1 on error
// Position moves forward by n bytes
```

---

## 🔑 CRITICAL CONCEPT #2: The THREE lseek() Formulas

**lseek() changes file position (NOT size!)**

```c
off_t lseek(int fd, off_t offset, int whence);
// Returns: new position, or -1 on error
```

### FORMULA 1: SEEK_SET (whence = 0)
```
new_position = offset
```
**Example:**
```c
lseek(fd, 50, SEEK_SET);  // Position = 50 (absolute)
```

### FORMULA 2: SEEK_CUR (whence = 1)
```
new_position = current_position + offset
```
**Example:**
```c
// Current position = 100
lseek(fd, 20, SEEK_CUR);   // Position = 100 + 20 = 120
lseek(fd, -30, SEEK_CUR);  // Position = 120 + (-30) = 90
```

### FORMULA 3: SEEK_END (whence = 2)
```
new_position = file_size + offset
```
**Example:**
```c
// File size = 200
lseek(fd, 0, SEEK_END);    // Position = 200 + 0 = 200 (end)
lseek(fd, -10, SEEK_END);  // Position = 200 + (-10) = 190
lseek(fd, 50, SEEK_END);   // Position = 200 + 50 = 250 (beyond end!)
```

---

## 🔑 CRITICAL CONCEPT #3: Failed lseek() Rule

**If calculated position < 0, lseek() FAILS!**

```c
// Current position = 50
lseek(fd, -100, SEEK_CUR);  // 50 + (-100) = -50 → NEGATIVE!
// lseek() returns -1 (failure)
// Position DOES NOT CHANGE (stays at 50)
```

**This is THE MOST COMMON EXAM TRAP!**

---

## 🔑 CRITICAL CONCEPT #4: Position vs Size

**TWO SEPARATE THINGS:**

| Operation | Changes Position? | Changes Size? |
|-----------|-------------------|---------------|
| lseek()   | YES ✓            | NO ✗         |
| read()    | YES ✓            | NO ✗         |
| write()   | YES ✓            | YES ✓ (if beyond end) |

**File Size = Furthest byte ever written**

---

## 🔑 CRITICAL CONCEPT #5: The T1 Question Pattern

```c
int fd = open("file.txt", O_RDWR);  // Initial size = 40, position = 0

lseek(fd, 100, SEEK_END);           // Position = 40 + 100 = 140
                                    // Size = 40 (unchanged!)

write(fd, buf, 9);                  // Position = 140 + 9 = 149
                                    // Size = 149 (grew!)

write(fd, buf, 9);                  // Position = 149 + 9 = 158
                                    // Size = 158

lseek(fd, -159, SEEK_CUR);          // 158 + (-159) = -1 → NEGATIVE!
                                    // lseek FAILS!
                                    // Position = 158 (unchanged!)
                                    // Size = 158 (unchanged!)

write(fd, buf, 9);                  // Position = 158 + 9 = 167
                                    // Size = 167

write(fd, buf, 9);                  // Position = 167 + 9 = 176
                                    // Size = 176

close(fd);

// Final size: 176 bytes
```

**Tracing Table:**
| Operation | Formula | Position | Size |
|-----------|---------|----------|------|
| open      | -       | 0        | 40   |
| lseek END | 40+100  | 140      | 40   |
| write 9   | 140+9   | 149      | 149  |
| write 9   | 149+9   | 158      | 158  |
| lseek CUR | FAIL    | 158      | 158  |
| write 9   | 158+9   | 167      | 167  |
| write 9   | 167+9   | 176      | 176  |

---

## 📝 TRUE/FALSE QUESTIONS - FILE I/O (20 Questions)

### T/F #1
**Q:** File descriptor 0 represents standard input (stdin).
**A:** TRUE ✓
**Why:** 0=stdin, 1=stdout, 2=stderr

---

### T/F #2
**Q:** lseek() changes the file size.
**A:** FALSE ✗
**Why:** lseek NEVER changes size, only position!

---

### T/F #3
**Q:** write() can change the file size.
**A:** TRUE ✓
**Why:** If writing beyond current end, file grows.

---

### T/F #4
**Q:** read() changes the file size.
**A:** FALSE ✗
**Why:** Reading doesn't modify file size.

---

### T/F #5
**Q:** SEEK_SET sets position relative to beginning of file.
**A:** TRUE ✓
**Why:** SEEK_SET: position = offset

---

### T/F #6
**Q:** SEEK_CUR sets position relative to current position.
**A:** TRUE ✓
**Why:** SEEK_CUR: position = current + offset

---

### T/F #7
**Q:** SEEK_END sets position relative to end of file.
**A:** TRUE ✓
**Why:** SEEK_END: position = size + offset

---

### T/F #8
**Q:** If lseek() fails, the file position is set to 0.
**A:** FALSE ✗
**Why:** Position doesn't change on failure!

---

### T/F #9
**Q:** lseek() can fail if the calculated position is negative.
**A:** TRUE ✓
**Why:** Negative position is invalid.

---

### T/F #10
**Q:** You can seek beyond the end of a file.
**A:** TRUE ✓
**Why:** Creates a "hole" (sparse file).

---

### T/F #11
**Q:** After fork(), parent and child share the same file position.
**A:** TRUE ✓
**Why:** They point to same open file table entry.

---

### T/F #12
**Q:** open() returns -1 on error.
**A:** TRUE ✓
**Why:** Standard error return value.

---

### T/F #13
**Q:** read() always reads the exact number of bytes requested.
**A:** FALSE ✗
**Why:** May read fewer (e.g., at EOF).

---

### T/F #14
**Q:** read() returns 0 when EOF is reached.
**A:** TRUE ✓
**Why:** 0 indicates end of file.

---

### T/F #15
**Q:** lseek(fd, 0, SEEK_SET) moves to beginning of file.
**A:** TRUE ✓
**Why:** Position = 0

---

### T/F #16
**Q:** lseek(fd, 0, SEEK_END) moves to end of file.
**A:** TRUE ✓
**Why:** Position = size + 0 = size

---

### T/F #17
**Q:** After a failed lseek(), subsequent write() uses the original position.
**A:** TRUE ✓
**Why:** Position unchanged after failed lseek.

---

### T/F #18
**Q:** O_APPEND makes all writes go to the end of file.
**A:** TRUE ✓
**Why:** Append mode automatically seeks to end.

---

### T/F #19
**Q:** close() releases the file descriptor.
**A:** TRUE ✓
**Why:** Frees the descriptor for reuse.

---

### T/F #20
**Q:** lseek() can be used with pipes.
**A:** FALSE ✗
**Why:** Pipes don't support seeking.

---

## 💻 CODE OUTPUT MCQs - FILE I/O (15 Questions)

### MCQ #1 - Basic lseek
```c
int fd = open("file.txt", O_RDWR);  // size = 100
lseek(fd, 50, SEEK_SET);
write(fd, "X", 1);
close(fd);
```
**Q:** What is the final file size?
- A) 50
- B) 51
- C) 100
- D) 101

**Answer:** C ✓
**Trace:**
- Initial: pos=0, size=100
- lseek: pos=50, size=100 (no change)
- write: pos=51, size=100 (wrote within file)
- Final size: 100

---

### MCQ #2 - SEEK_END
```c
int fd = open("file.txt", O_RDWR);  // size = 50
lseek(fd, 30, SEEK_END);
write(fd, "Y", 1);
close(fd);
```
**Q:** What is the final file size?
- A) 50
- B) 51
- C) 80
- D) 81

**Answer:** D ✓
**Trace:**
- Initial: pos=0, size=50
- lseek: pos=50+30=80, size=50
- write: pos=81, size=81 (beyond end!)
- Final size: 81

---

### MCQ #3 - Failed lseek
```c
int fd = open("file.txt", O_RDWR);  // size = 100
lseek(fd, 50, SEEK_SET);
lseek(fd, -100, SEEK_CUR);
write(fd, "Z", 1);
close(fd);
```
**Q:** At what position does write() occur?
- A) 0
- B) 50
- C) -50
- D) 51

**Answer:** B ✓
**Trace:**
- lseek SET: pos=50
- lseek CUR: 50+(-100)=-50 → FAILS! pos stays 50
- write at pos=50

---

### MCQ #4 - Multiple Operations
```c
int fd = open("file.txt", O_RDWR);  // size = 20
write(fd, "ABCD", 4);
lseek(fd, 10, SEEK_SET);
write(fd, "XY", 2);
close(fd);
```
**Q:** What is the final file size?
- A) 4
- B) 12
- C) 20
- D) 24

**Answer:** C ✓
**Trace:**
- write: pos=4, size=20 (within file)
- lseek: pos=10, size=20
- write: pos=12, size=20 (within file)
- Final: 20

---

### MCQ #5 - SEEK_CUR
```c
int fd = open("file.txt", O_RDWR);  // size = 100
lseek(fd, 40, SEEK_SET);
lseek(fd, 20, SEEK_CUR);
lseek(fd, -30, SEEK_CUR);
write(fd, "M", 1);
close(fd);
```
**Q:** At what position does write() occur?
- A) 30
- B) 40
- C) 50
- D) 60

**Answer:** A ✓
**Trace:**
- lseek SET: pos=40
- lseek CUR: pos=40+20=60
- lseek CUR: pos=60-30=30
- write at pos=30

---

### MCQ #6 - The T1 Pattern
```c
int fd = open("file.txt", O_RDWR);  // size = 40
lseek(fd, 100, SEEK_END);
write(fd, "A", 1);
write(fd, "B", 1);
lseek(fd, -143, SEEK_CUR);
write(fd, "C", 1);
close(fd);
```
**Q:** What is the final file size?
- A) 40
- B) 142
- C) 143
- D) 141

**Answer:** B ✓
**Trace:**
- lseek END: pos=40+100=140, size=40
- write A: pos=141, size=141
- write B: pos=142, size=142
- lseek CUR: 142+(-143)=-1 → FAILS! pos=142
- write C: pos=143, size=143... wait let me recalculate
- Actually after write B: pos=142, size=142
- lseek: 142-143=-1 FAILS, pos stays 142
- write C at 142: pos=143 but size stays 142? No, write extends!
- Final size: 143

Actually Answer should be C: 143

---

### MCQ #7 - read() Effect
```c
int fd = open("file.txt", O_RDONLY);  // size = 100
char buf[20];
read(fd, buf, 10);
read(fd, buf, 15);
off_t pos = lseek(fd, 0, SEEK_CUR);
close(fd);
```
**Q:** What is the value of pos?
- A) 0
- B) 10
- C) 15
- D) 25

**Answer:** D ✓
**Trace:**
- read 10: pos=10
- read 15: pos=25
- lseek(0,CUR) returns current position = 25

---

### MCQ #8 - Sparse File
```c
int fd = open("file.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
write(fd, "A", 1);
lseek(fd, 1000, SEEK_SET);
write(fd, "B", 1);
close(fd);
```
**Q:** What is the file size?
- A) 2
- B) 1000
- C) 1001
- D) 1002

**Answer:** C ✓
**Trace:**
- write A: pos=1, size=1
- lseek: pos=1000, size=1
- write B: pos=1001, size=1001
- Final: 1001

---

### MCQ #9 - Complex Trace
```c
int fd = open("file.txt", O_RDWR);  // size = 60
lseek(fd, -20, SEEK_END);
write(fd, "XY", 2);
lseek(fd, 100, SEEK_SET);
write(fd, "Z", 1);
close(fd);
```
**Q:** What is the final file size?
- A) 60
- B) 62
- C) 100
- D) 101

**Answer:** D ✓
**Trace:**
- lseek END: pos=60-20=40, size=60
- write XY: pos=42, size=60 (within)
- lseek SET: pos=100, size=60
- write Z: pos=101, size=101 (extended!)
- Final: 101

---

### MCQ #10-15: Quick Problems

**MCQ #10:**
```c
lseek(fd, 0, SEEK_END);
off_t size = lseek(fd, 0, SEEK_CUR);
```
What does size contain?
**Answer:** File size (lseek returns new position)

**MCQ #11:**
```c
// size = 50
lseek(fd, 10, SEEK_SET);
lseek(fd, 5, SEEK_CUR);
```
Final position?
**Answer:** 15 (10+5)

**MCQ #12:**
```c
// size = 80
lseek(fd, -10, SEEK_END);
```
Final position?
**Answer:** 70 (80-10)

**MCQ #13:**
```c
// pos = 30
lseek(fd, -50, SEEK_CUR);
```
What happens?
**Answer:** Fails (30-50=-20), position stays 30

**MCQ #14:**
```c
write(fd, "12345", 5);
lseek(fd, -2, SEEK_CUR);
write(fd, "XY", 2);
```
What's in file?
**Answer:** "123XY" (overwrites last 2)

**MCQ #15:**
O_APPEND mode - does lseek() matter?
**Answer:** NO - writes always go to end

---

# 📚 TOPIC 3: SIGNALS

## ⏰ Thursday 4:00-5:00 PM (1 HOUR)

---

## 🎓 CONCEPT: What are Signals?

**Simple Analogy:** Signals are like "notifications" or "alerts" sent to processes.

Examples:
- Ctrl-C → sends SIGINT (interrupt signal)
- alarm() timer → sends SIGALRM
- Segmentation fault → sends SIGSEGV

---

## 🔑 CRITICAL CONCEPT #1: Common Signals

| Signal | Number | Catchable? | Meaning |
|--------|--------|------------|---------|
| SIGINT | 2 | YES ✓ | Ctrl-C (interrupt) |
| SIGKILL | 9 | **NO** ✗ | Force kill (uncatchable!) |
| SIGALRM | 14 | YES ✓ | alarm() timer |
| SIGTERM | 15 | YES ✓ | Terminate request |
| SIGCHLD | 17 | YES ✓ | Child stopped/terminated |
| SIGCONT | 18 | YES ✓ | Continue process |
| SIGSTOP | 19 | **NO** ✗ | Stop (uncatchable!) |

**KEY POINT:** SIGKILL (9) and SIGSTOP (19) are the ONLY uncatchable signals!

---

## 🔑 CRITICAL CONCEPT #2: signal() - Registering Handlers

```c
void handler(int signo) {
    printf("Caught signal %d!\n", signo);
    signal(signo, handler);  // Re-register (some systems)
}

int main() {
    signal(SIGINT, handler);  // Register handler for Ctrl-C
    
    while(1) {
        sleep(1);
    }
}
```

**When Ctrl-C pressed:**
- SIGINT sent to process
- handler() executes
- Process continues running

---

## 🔑 CRITICAL CONCEPT #3: alarm() - THE ONE-SHOT TIMER

**MOST IMPORTANT EXAM RULE:**

```
alarm() is ONE-SHOT!
It fires ONCE and stops!
To repeat, handler MUST call alarm() again!
```

### Example 1: ONE-SHOT (No Repeat)
```c
void handler(int sig) {
    signal(SIGALRM, handler);  // Re-register ✓
    printf("Handler!\n");
    // NO alarm() call here! ← TRAP!
}

int main() {
    signal(SIGALRM, handler);
    alarm(8);  // Set alarm for 8 seconds
    
    for(int i = 0; i < 10; i++)
        sleep(1);  // Sleep 10 seconds total
    
    return 0;
}
```

**Trace:**
```
Time 0-8s: sleeping...
Time 8s: SIGALRM delivered → handler executes → prints "Handler!"
Time 8-10s: sleeping... (NO MORE ALARMS!)

Result: Handler called ONCE only
```

---

### Example 2: REPEATING (Calls alarm() Again)
```c
void handler(int sig) {
    signal(SIGALRM, handler);  
    printf("Handler!\n");
    alarm(2);  // ← ADDS THIS to repeat!
}

int main() {
    signal(SIGALRM, handler);
    alarm(2);  // First alarm
    
    sleep(7);  // Sleep 7 seconds
    
    return 0;
}
```

**Trace:**
```
Time 0s: alarm(2) set
Time 2s: SIGALRM → handler → prints → alarm(2) set again
Time 4s: SIGALRM → handler → prints → alarm(2) set again
Time 6s: SIGALRM → handler → prints → alarm(2) set again
Time 7s: program exits

Result: Handler called 3 times (at 2s, 4s, 6s)
```

---

## 🔑 CRITICAL CONCEPT #4: Process Groups & Ctrl-C

**THE RULE:**
```
Ctrl-C (SIGINT) is sent to FOREGROUND process group ONLY!
```

### Example: T2 Question Pattern
```c
void handler() { 
    exit(0); 
}

int main() {
    signal(SIGINT, handler);
    
    for(int i = 1; i < 4; i++)
        fork();  // 8 processes
    
    if((pid = fork()) == 0) {
        // Child creates NEW process group
        setpgid(0, getpid());
    }
    
    // Ctrl-C pressed
}
```

**Trace:**
```
After for loop:
- 2^3 = 8 processes
- All in SAME process group (parent's group)

After if(fork()):
- All 8 processes fork
- Total: 16 processes
  - 8 parents: stay in original group (foreground)
  - 8 children: create new groups (background)

When Ctrl-C pressed:
- SIGINT sent to foreground group only
- 8 parents: receive SIGINT → handler() → exit(0) → DIE
- 8 children: in different groups → DON'T receive SIGINT → SURVIVE

Answer: 8 processes survive
```

---

## 🔑 CRITICAL CONCEPT #5: kill() - Sending Signals

```c
int kill(pid_t pid, int sig);

// Examples:
kill(child_pid, SIGTERM);     // Send SIGTERM to specific process
kill(0, SIGINT);              // Send to all in same group
kill(getpid(), SIGALRM);      // Send to self

// Job control:
kill(pid, SIGSTOP);           // Stop process (like Ctrl-Z)
kill(pid, SIGCONT);           // Continue process (like fg)

// Special:
kill(getpid(), SIGSTOP);      // Stop yourself!
```

---

## 📝 TRUE/FALSE QUESTIONS - SIGNALS (20 Questions)

### T/F #1
**Q:** SIGINT is sent when user presses Ctrl-C.
**A:** TRUE ✓
**Why:** Ctrl-C sends SIGINT (signal 2).

### T/F #2
**Q:** SIGKILL can be caught or ignored.
**A:** FALSE ✗
**Why:** SIGKILL (9) is uncatchable!

### T/F #3
**Q:** SIGSTOP can be caught or ignored.
**A:** FALSE ✗
**Why:** SIGSTOP (19) is uncatchable!

### T/F #4
**Q:** alarm() sets a repeating timer.
**A:** FALSE ✗
**Why:** ONE-SHOT! Fires once and stops!

### T/F #5
**Q:** To make alarm() repeat, handler must call alarm() again.
**A:** TRUE ✓
**Why:** alarm() is one-shot by design.

### T/F #6
**Q:** Only one alarm can be active at a time per process.
**A:** TRUE ✓
**Why:** New alarm() cancels previous.

### T/F #7
**Q:** alarm(0) cancels the current alarm.
**A:** TRUE ✓
**Why:** alarm(0) is the cancel command.

### T/F #8
**Q:** Ctrl-C sends SIGINT to all processes in the system.
**A:** FALSE ✗
**Why:** Only to foreground process group!

### T/F #9
**Q:** Ctrl-C sends SIGINT to the foreground process group only.
**A:** TRUE ✓
**Why:** This is the key rule for T2 questions.

### T/F #10
**Q:** setpgid(0, getpid()) creates a new process group.
**A:** TRUE ✓
**Why:** Makes process a group leader.

### T/F #11
**Q:** Processes in different process groups don't receive Ctrl-C.
**A:** TRUE ✓
**Why:** If not in foreground group.

### T/F #12
**Q:** SIGALRM is sent by the alarm() timer.
**A:** TRUE ✓
**Why:** alarm() sends SIGALRM after N seconds.

### T/F #13
**Q:** The default action for SIGINT is to terminate the process.
**A:** TRUE ✓
**Why:** Unless caught by handler.

### T/F #14
**Q:** A signal handler must have signature: void handler(int).
**A:** TRUE ✓
**Why:** Required signature.

### T/F #15
**Q:** Signal handlers are inherited across fork().
**A:** TRUE ✓
**Why:** Child inherits parent's handlers.

### T/F #16
**Q:** Signal handlers are preserved across exec().
**A:** FALSE ✗
**Why:** Reset to default after exec().

### T/F #17
**Q:** alarm() returns the seconds remaining from previous alarm.
**A:** TRUE ✓
**Why:** Returns remaining time or 0.

### T/F #18
**Q:** A new alarm() call cancels the previous alarm.
**A:** TRUE ✓
**Why:** Only one alarm active at a time.

### T/F #19
**Q:** kill(0, sig) sends signal to all processes in same group.
**A:** TRUE ✓
**Why:** pid=0 means same group.

### T/F #20
**Q:** raise(sig) is equivalent to kill(getpid(), sig).
**A:** TRUE ✓
**Why:** Both send signal to self.

---

## 💻 CODE OUTPUT MCQs - SIGNALS (15 Questions)

### MCQ #1 - Basic alarm()
```c
void handler(int sig) {
    signal(SIGALRM, handler);
    printf("Alarm!\n");
}

int main() {
    signal(SIGALRM, handler);
    alarm(3);
    sleep(5);
    return 0;
}
```
**Q:** How many times does handler execute?
- A) 0
- B) 1
- C) 2
- D) Infinite

**Answer:** B ✓
**Trace:**
- alarm(3) set
- At 3s: SIGALRM → handler → prints (no new alarm!)
- Sleep continues for 2 more seconds
- Total: 1 time

---

### MCQ #2 - Repeating alarm()
```c
void handler(int sig) {
    signal(SIGALRM, handler);
    printf("Tick\n");
    alarm(2);  // Re-arms!
}

int main() {
    signal(SIGALRM, handler);
    alarm(2);
    sleep(7);
    return 0;
}
```
**Q:** How many "Tick" are printed?
- A) 1
- B) 2
- C) 3
- D) 4

**Answer:** C ✓
**Trace:**
- At 2s: handler → prints → alarm(2)
- At 4s: handler → prints → alarm(2)
- At 6s: handler → prints → alarm(2)
- At 7s: program exits
- Total: 3 times

---

### MCQ #3 - No Handler
```c
int main() {
    alarm(5);
    sleep(10);
    printf("Done\n");
    return 0;
}
```
**Q:** What happens?
- A) Prints "Done"
- B) Program terminates at 5s
- C) Infinite loop
- D) Segmentation fault

**Answer:** B ✓
**Trace:**
- No handler registered
- At 5s: SIGALRM → default action = terminate
- "Done" never printed

---

### MCQ #4 - Process Groups
```c
void handler() { exit(0); }

int main() {
    signal(SIGINT, handler);
    
    fork();  // 2 processes
    fork();  // 4 processes
    
    if(fork() == 0) {
        setpgid(0, getpid());  // Child creates new group
    }
    
    // Ctrl-C pressed
    pause();  // Wait for signal
}
```
**Q:** How many processes survive Ctrl-C?
- A) 0
- B) 4
- C) 8
- D) 16

**Answer:** B ✓
**Trace:**
- After 3 forks: 8 processes
- All 8 fork again: 16 total
- 8 parents: original group → DIE from SIGINT
- 8 children: new groups → SURVIVE
- Wait, let me recalculate...
- After fork() fork(): 4 processes
- All 4 do if(fork()): 8 total
- 4 parents: original group → DIE
- 4 children: new groups → SURVIVE
- Answer: 4

---

### MCQ #5 - alarm() Cancellation
```c
int main() {
    alarm(10);
    alarm(5);
    sleep(7);
    printf("Done\n");
    return 0;
}
```
**Q:** When does program terminate?
- A) 5 seconds
- B) 7 seconds
- C) 10 seconds
- D) 15 seconds

**Answer:** A ✓
**Trace:**
- alarm(10) set
- alarm(5) replaces it
- At 5s: SIGALRM → default = terminate
- Never reaches printf

---

### MCQ #6 - Multiple Signals
```c
void handler(int sig) {
    printf("%d ", sig);
}

int main() {
    signal(SIGINT, handler);
    signal(SIGALRM, handler);
    
    alarm(2);
    sleep(3);
    
    return 0;
}
```
**Q:** What is printed?
- A) Nothing
- B) 2
- C) 14
- D) 2 14

**Answer:** C ✓
**Trace:**
- At 2s: SIGALRM (14) → handler → prints "14"
- No SIGINT (2) sent
- Output: "14"

---

### MCQ #7-15: Quick Problems

**MCQ #7:** alarm(0) does what?
**Answer:** Cancels current alarm

**MCQ #8:** Can SIGKILL be caught?
**Answer:** NO - uncatchable

**MCQ #9:** Ctrl-Z sends which signal?
**Answer:** SIGTSTP (20), not SIGSTOP

**MCQ #10:** kill(getpid(), SIGSTOP) does what?
**Answer:** Stops own process

**MCQ #11:** Default SIGALRM action?
**Answer:** Terminate process

**MCQ #12:** Signal handler returns what type?
**Answer:** void

**MCQ #13:** How many alarms can be active?
**Answer:** 1 (only one at a time)

**MCQ #14:** setpgid(0,0) equivalent to?
**Answer:** setpgid(getpid(), getpid())

**MCQ #15:** SIGCHLD default action?
**Answer:** Ignore

---

# 📚 TOPIC 4: PIPES

## ⏰ Thursday 5:00-6:00 PM (1 HOUR)

---

## 🎓 CONCEPT: What are Pipes?

**Simple Analogy:** A pipe is like a tube connecting two processes.

```
Process A ─[writes]→ PIPE ─[reads]→ Process B
```

**Key Points:**
- One-way communication (unidirectional)
- FIFO (First In, First Out)
- Data flows: write end → read end

---

## 🔑 CRITICAL CONCEPT #1: pipe() System Call

```c
int fd[2];
pipe(fd);

// fd[0] = READ end
// fd[1] = WRITE end

write(fd[1], "Hello", 5);  // Write to pipe
char buf[10];
read(fd[0], buf, 5);       // Read from pipe
```

---

## 🔑 CRITICAL CONCEPT #2: Pipe with fork()

**After fork(), both parent and child have access to SAME pipe!**

```c
int fd[2];
pipe(fd);

pid_t pid = fork();

if (pid == 0) {
    // CHILD: writes
    close(fd[0]);  // Close unused read end
    write(fd[1], "Message", 7);
    close(fd[1]);
    exit(0);
}
else {
    // PARENT: reads
    close(fd[1]);  // Close unused write end
    char buf[20];
    read(fd[0], buf, 20);
    printf("Received: %s\n", buf);
    close(fd[0]);
}
```

---

## 🔑 CRITICAL CONCEPT #3: Race Conditions (EXAM FAVORITE!)

**When multiple processes read from SAME pipe:**
```
Order is UNPREDICTABLE!
Cannot determine which process gets which bytes!
```

### T2 Question Pattern:
```c
int fd[2];
int main_pid = getpid();
pipe(fd);

fork(); fork();  // 4 processes

if (getpid() == main_pid) {
    // ONLY original parent writes
    write(fd[1], "msg", 20);  // 80 bytes total
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
    write(fd[1], "msg", 20);
}
else {
    // 3 children read
    char buf[20];
    read(fd[0], buf, 20);
}
```

**Trace:**
```
After 2 forks: 4 processes total

Identify roles:
- Original parent: writes 80 bytes
- Other 3 processes: each tries to read 20 bytes

RACE CONDITION!
- Order unpredictable
- Could be: 20+20+20=60, or 40+40+0=80, or 80+0+0=80
- Many possibilities!

Answer: "Cannot say" or "60-80 bytes" depending on options
```

**KEY RULE:** Multiple readers = RACE CONDITION = unpredictable

---

## 📝 TRUE/FALSE QUESTIONS - PIPES (15 Questions)

### T/F #1
**Q:** A pipe provides one-way communication.
**A:** TRUE ✓
**Why:** Unidirectional only.

### T/F #2
**Q:** fd[0] is the write end of the pipe.
**A:** FALSE ✗
**Why:** fd[0] = read, fd[1] = write

### T/F #3
**Q:** fd[1] is the write end of the pipe.
**A:** TRUE ✓
**Why:** fd[1] is write end.

### T/F #4
**Q:** After fork(), child inherits pipe file descriptors.
**A:** TRUE ✓
**Why:** All FDs inherited.

### T/F #5
**Q:** Multiple processes can read from same pipe.
**A:** TRUE ✓
**Why:** Possible but causes race condition.

### T/F #6
**Q:** Multiple readers from same pipe causes race condition.
**A:** TRUE ✓
**Why:** Order unpredictable!

### T/F #7
**Q:** Pipes are bidirectional.
**A:** FALSE ✗
**Why:** One-way only.

### T/F #8
**Q:** You need two pipes for bidirectional communication.
**A:** TRUE ✓
**Why:** One for each direction.

### T/F #9
**Q:** Pipes are FIFO (First In, First Out).
**A:** TRUE ✓
**Why:** Data order preserved.

### T/F #10
**Q:** read() from empty pipe blocks if write end open.
**A:** TRUE ✓
**Why:** Waits for data.

### T/F #11
**Q:** read() from empty pipe returns 0 if write end closed.
**A:** TRUE ✓
**Why:** EOF condition.

### T/F #12
**Q:** write() to pipe with no readers causes SIGPIPE.
**A:** TRUE ✓
**Why:** Broken pipe signal.

### T/F #13
**Q:** Pipe data is removed after being read.
**A:** TRUE ✓
**Why:** Consumed, not persistent.

### T/F #14
**Q:** You should close unused pipe ends.
**A:** TRUE ✓
**Why:** Best practice.

### T/F #15
**Q:** Named pipes (FIFOs) exist in the filesystem.
**A:** TRUE ✓
**Why:** Have pathname.

---

## 💻 CODE OUTPUT MCQs - PIPES (10 Questions)

### MCQ #1 - Basic Pipe
```c
int fd[2];
pipe(fd);

if (fork() == 0) {
    write(fd[1], "Hi", 2);
    exit(0);
}

char buf[10];
read(fd[0], buf, 10);
buf[2] = '\0';
printf("%s", buf);
```
**Q:** What is printed?
- A) Nothing
- B) Hi
- C) Undefined
- D) Error

**Answer:** B ✓
**Trace:**
- Child writes "Hi" (2 bytes)
- Parent reads up to 10 bytes
- Actually reads 2 bytes ("Hi")
- Null-terminates and prints "Hi"

---

### MCQ #2 - Race Condition
```c
int fd[2];
pipe(fd);

fork(); fork();  // 4 processes

if (fork() == 0) {
    char buf[5];
    read(fd[0], buf, 5);
} else {
    write(fd[1], "ABCDE", 5);
}
```
**Q:** How many processes read?
- A) 1
- B) 2
- C) 4
- D) 8

**Answer:** C ✓
**Trace:**
- After 2 forks: 4 processes
- All 4 do if(fork()): 8 total
- 4 parents write
- 4 children read
- Answer: 4 readers

---

### MCQ #3-10: Quick Problems

**MCQ #3:** What does close(fd[1]) in reader do?
**Answer:** Closes write end (best practice)

**MCQ #4:** If all write ends closed, read returns?
**Answer:** 0 (EOF)

**MCQ #5:** Pipe capacity typically?
**Answer:** 64KB (65536 bytes)

**MCQ #6:** Can lseek() on pipe?
**Answer:** NO - pipes don't support seeking

**MCQ #7:** SIGPIPE default action?
**Answer:** Terminate process

**MCQ #8:** Small writes < PIPE_BUF are?
**Answer:** Atomic

**MCQ #9:** Named pipe created with?
**Answer:** mkfifo()

**MCQ #10:** dup2 in pipeline?
**Answer:** Redirects stdin/stdout to pipe

---

# 📚 TOPIC 5: THREADS

## ⏰ Thursday 6:00-7:00 PM (1 HOUR)

---

## 🎓 CONCEPT: Threads vs Processes

**Threads = lightweight execution units WITHIN a process**

**Key Difference:**
- **Processes:** Separate memory, heavy, use fork()
- **Threads:** SHARED memory, lightweight, use pthread_create()

---

## 🔑 CRITICAL CONCEPT #1: Shared vs Private Memory

### SHARED (ALL threads see same)
```c
int counter = 0;  // ← GLOBAL = SHARED!

void* func(void* arg) {
    counter++;  // All threads modify SAME counter
    return NULL;
}
```

### PRIVATE (Each thread has own)
```c
void* func(void* arg) {
    int local = 0;  // ← LOCAL = PRIVATE!
    local++;  // Each thread has own copy
    return NULL;
}
```

---

## 🔑 CRITICAL CONCEPT #2: pthread Functions

```c
// Create thread
pthread_t thread;
pthread_create(&thread, NULL, function, arg);

// Wait for thread
pthread_join(thread, NULL);  // Like waitpid for threads

// Cancel thread
pthread_cancel(thread);

// Get own thread ID
pthread_self();
```

---

## 🔑 CRITICAL CONCEPT #3: THE GOLDEN RULE

```
🔥 When main() returns → ALL THREADS DIE! 🔥
Even if threads are in infinite loops!
```

### T2 Question Pattern:
```c
int counter = 0;  // SHARED!

void* func1(void* p) {
    while(1) {
        printf("thread one\n");
        sleep(1);
        counter++;
        if (counter == 3) {
            pthread_cancel(pthread_self());  // Exit at 3
        }
    }
}

void* func2(void* p) {
    while(1) {  // INFINITE LOOP!
        printf("thread two\n");
        sleep(1);
    }
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, func1, NULL);
    pthread_create(&t2, NULL, func2, NULL);
    
    pthread_join(t1, NULL);  // Wait for t1 only
    return 0;  // ← main() RETURNS HERE!
}
```

**Trace:**
```
Time 0-1s: both threads run
Time 1s: counter = 1
Time 2s: counter = 2
Time 3s: counter = 3
  → func1: pthread_cancel(self) → t1 EXITS
  → pthread_join returns
  → main() returns
  → t2 KILLED (even though infinite loop!)

Result: Program runs ~3 seconds, then ALL threads die
```

---

## 📝 TRUE/FALSE QUESTIONS - THREADS (15 Questions)

### T/F #1
**Q:** Threads share the same memory space.
**A:** TRUE ✓
**Why:** All threads in process share memory.

### T/F #2
**Q:** Global variables are shared by all threads.
**A:** TRUE ✓
**Why:** Global = shared.

### T/F #3
**Q:** Local variables are shared by all threads.
**A:** FALSE ✗
**Why:** Local = private (each thread has own).

### T/F #4
**Q:** When main() returns, all threads are terminated.
**A:** TRUE ✓
**Why:** 🔥 GOLDEN RULE!

### T/F #5
**Q:** Threads continue after main() exits.
**A:** FALSE ✗
**Why:** ALL killed when main() returns!

### T/F #6
**Q:** pthread_create() returns 0 on success.
**A:** TRUE ✓
**Why:** Standard return.

### T/F #7
**Q:** pthread_join() blocks until thread finishes.
**A:** TRUE ✓
**Why:** Like waitpid for threads.

### T/F #8
**Q:** Threads are lighter-weight than processes.
**A:** TRUE ✓
**Why:** Less overhead.

### T/F #9
**Q:** Each thread has its own stack.
**A:** TRUE ✓
**Why:** Private stack per thread.

### T/F #10
**Q:** All threads in a process share the heap.
**A:** TRUE ✓
**Why:** Heap is shared.

### T/F #11
**Q:** Threads share file descriptors.
**A:** TRUE ✓
**Why:** Part of process state.

### T/F #12
**Q:** pthread_cancel() immediately kills the thread.
**A:** FALSE ✗
**Why:** Requests cancellation at cancellation point.

### T/F #13
**Q:** Multiple threads can modify same variable without synchronization.
**A:** TRUE ✓ (possible but dangerous)
**Why:** Causes race conditions.

### T/F #14
**Q:** counter++ is an atomic operation.
**A:** FALSE ✗
**Why:** Requires synchronization!

### T/F #15
**Q:** pthread_exit() in main() keeps threads alive.
**A:** TRUE ✓
**Why:** Different from return!

---

## 💻 CODE OUTPUT MCQs - THREADS (10 Questions)

### MCQ #1 - Basic Thread
```c
void* func(void* arg) {
    printf("Thread\n");
    return NULL;
}

int main() {
    pthread_t t;
    pthread_create(&t, NULL, func, NULL);
    pthread_join(t, NULL);
    printf("Main\n");
    return 0;
}
```
**Q:** What is printed?
- A) Thread\nMain\n
- B) Main\nThread\n
- C) Only Thread
- D) Only Main

**Answer:** A ✓
**Trace:**
- Thread created and executes
- main() waits at pthread_join
- Thread prints "Thread"
- Thread finishes
- main() prints "Main"

---

### MCQ #2 - Shared Counter
```c
int count = 0;

void* func(void* arg) {
    count = 10;
    return NULL;
}

int main() {
    pthread_t t;
    pthread_create(&t, NULL, func, NULL);
    pthread_join(t, NULL);
    printf("%d", count);
    return 0;
}
```
**Q:** What is printed?
- A) 0
- B) 10
- C) Undefined
- D) 20

**Answer:** B ✓
**Trace:**
- count = 0 (global, shared)
- Thread sets count = 10
- main() waits, then prints 10

---

### MCQ #3 - The T2 Pattern
```c
int counter = 0;

void* func1(void* p) {
    while(1) {
        sleep(1);
        counter++;
        if (counter == 2) {
            pthread_cancel(pthread_self());
        }
    }
}

void* func2(void* p) {
    while(1) {
        sleep(1);
    }
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, func1, NULL);
    pthread_create(&t2, NULL, func2, NULL);
    pthread_join(t1, NULL);
    return 0;
}
```
**Q:** How long does program run?
- A) ~1 second
- B) ~2 seconds
- C) Forever
- D) Immediately exits

**Answer:** B ✓
**Trace:**
- At 1s: counter=1
- At 2s: counter=2 → t1 exits
- pthread_join returns
- main() returns → t2 KILLED
- Total: ~2 seconds

---

### MCQ #4-10: Quick Problems

**MCQ #4:** pthread_self() returns?
**Answer:** Calling thread's ID

**MCQ #5:** Heap memory shared?
**Answer:** YES (all threads)

**MCQ #6:** Stack memory shared?
**Answer:** NO (private per thread)

**MCQ #7:** After fork() in multithreaded program?
**Answer:** Only calling thread duplicated

**MCQ #8:** pthread functions return on success?
**Answer:** 0

**MCQ #9:** Can threads have same function?
**Answer:** YES

**MCQ #10:** Mutex provides?
**Answer:** Mutual exclusion

---

# 📚 TOPIC 6: SHELL SCRIPTING

## ⏰ Thursday 8:00-9:00 PM (1 HOUR)

---

## 🎓 CONCEPT: Shell Basics

**Variables:**
```bash
count=1          # Assign (NO SPACES!)
echo $count      # Use variable
count=$(($count+1))  # Arithmetic
```

**Conditions:**
```bash
[ $a -eq $b ]    # equal
[ $a -ne $b ]    # not equal
[ $a -lt $b ]    # less than
[ $a -gt $b ]    # greater than
```

**CRITICAL:** Spaces required inside [ ]!

---

## 🔑 CRITICAL CONCEPT: The T2 Question

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
        kill -19 $$      # Stop self
        fg               # Resume (user types)
        echo hello3
        break            # Exit loop
    fi
done
```

**Trace:**
```
Iter 1: count=1, prints hello1 hello2, count=2
Iter 2: count=2, prints hello1 hello2, count=3
...
Iter 9: count=9, prints hello1 hello2, count=10
Iter 10: count=10, count<10? NO
  → prints hello1
  → kill -19 $$ (SIGSTOP) → STOPS
  → User types "fg" → RESUMES
  → prints hello3
  → break (exits loop)

Last output: hello3
```

---

## 📝 TRUE/FALSE - SHELL (10 Questions)

### T/F #1
**Q:** count=1 assigns value without spaces.
**A:** TRUE ✓
**Why:** No spaces around =

### T/F #2
**Q:** count = 1 is valid syntax.
**A:** FALSE ✗
**Why:** Spaces cause error!

### T/F #3
**Q:** -eq tests equality.
**A:** TRUE ✓
**Why:** Numeric equal.

### T/F #4
**Q:** -lt tests less than.
**A:** TRUE ✓
**Why:** Less than operator.

### T/F #5
**Q:** $$ is current shell PID.
**A:** TRUE ✓
**Why:** Own process ID.

### T/F #6
**Q:** kill -19 stops process.
**A:** TRUE ✓
**Why:** SIGSTOP.

### T/F #7
**Q:** fg resumes stopped job.
**A:** TRUE ✓
**Why:** Foreground command.

### T/F #8
**Q:** break exits while loop.
**A:** TRUE ✓
**Why:** Loop control.

### T/F #9
**Q:** Spaces required inside [ ].
**A:** TRUE ✓
**Why:** Syntax requirement.

### T/F #10
**Q:** $(($count+1)) increments count.
**A:** TRUE ✓
**Why:** Arithmetic expansion.

---

# 📚 TOPIC 7: POINTERS (Brief)

## ⏰ Thursday 9:00-9:30 PM (30 MIN)

**Basics:**
```c
int x = 5;
int *ptr = &x;    // ptr = address of x
*ptr = 10;        // Value at address = 10
                  // x is now 10!
```

**Array/Pointer:**
```c
int arr[] = {10, 20, 30};
int *ptr = arr;
ptr++;            // Move to next element
*ptr;             // Now 20
```

---

## 📝 TRUE/FALSE - POINTERS (5 Questions)

### T/F #1
**Q:** &x returns address of x.
**A:** TRUE ✓

### T/F #2
**Q:** *ptr returns value at address ptr.
**A:** TRUE ✓

### T/F #3
**Q:** ptr++ moves to next element.
**A:** TRUE ✓

### T/F #4
**Q:** arr[i] equals *(arr+i).
**A:** TRUE ✓

### T/F #5
**Q:** Pointers and arrays are identical.
**A:** FALSE ✗
**Why:** Similar but not identical.

---

# 🎯 FRIDAY 3:00-5:00 PM - FINAL REVIEW

## Hour 1: T/F Mastery (60 min)

**Review all 120 T/F questions:**
- 20 Fork
- 20 File I/O
- 20 Signals
- 15 Pipes
- 15 Threads
- 10 Shell
- 5 Pointers

**Goal:** 95%+ correct (114/120)

---

## Hour 2: Code Output Drills (60 min)

**Practice tracing:**
- 10 Fork code outputs
- 10 File I/O traces
- 10 Signal traces
- 5 Pipe traces
- 5 Thread traces

**Goal:** 50%+ correct (20/40)

---

# 📋 EXAM DAY STRATEGY

## For True/False (10 questions):
1. Read carefully
2. Check your cheat sheets
3. Look for key words (catchable, shared, ONE-SHOT)
4. **Goal: 10/10**

## For Code Output MCQs (30 questions):
1. **Identify topic** (10 sec)
2. **Draw trace table** (2 min)
   - Fork: process tree
   - File I/O: position/size table
   - Signals: timeline
   - Pipes: count readers/writers
   - Threads: counter tracking
3. **Calculate answer** (30 sec)
4. **Goal: 15/30 (50%)**

## Time Management:
- T/F: 1-2 min each = 20 min
- Code MCQs: 3-4 min each = 100 min
- Buffer: 20 min
- Total: 140 min (2h 20min) for 40 questions

---

# ✅ FINAL CHECKLIST

**Thursday Night:**
- □ Understand all 5 core topics
- □ Can trace fork (2^n formula)
- □ Can trace file I/O (3 lseek formulas)
- □ Know alarm() is ONE-SHOT
- □ Know multiple readers = race
- □ Know main() returns → all threads die

**Friday:**
- □ 95%+ on all T/F questions
- □ Can trace any code output in 3 min
- □ Cheat sheets organized
- □ Confident and ready

**At Exam:**
- □ Use open book advantage
- □ Don't rush T/F (easy marks!)
- □ Draw tables for code tracing
- □ Trust your practice

---

# 💪 YOU'VE GOT THIS!

**Your Targets:**
- 10/10 T/F = 100%
- 15/30 Code = 50%
- **Total: 25/40 = 62.5% PASS!**

**Remember:**
- Open book = use notes
- T/F = easy marks (get them ALL!)
- Code tracing = practice pays off
- 14 hours is enough!

**NOW START WITH THURSDAY 1 AM!**

**YOU WILL PASS! 🎯🔥**
