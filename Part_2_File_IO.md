# 🏆 COMP8567 ULTIMATE EXAM GUIDE - PART 2/9
## UNIX FILE INPUT/OUTPUT
## Master File I/O and Crush the lseek() Questions!

---

# 📚 PART 2: UNIX FILE INPUT/OUTPUT

## 🎯 WHAT YOU'LL MASTER IN THIS SECTION:
- ✅ File descriptors and how they work
- ✅ open(), read(), write(), close() system calls
- ✅ lseek() and the THREE CRITICAL FORMULAS
- ✅ File position tracking (EXAM KILLER TOPIC!)
- ✅ File size calculations
- ✅ dup() and dup2() for file descriptor manipulation
- ✅ 70+ True/False questions for guaranteed marks!

---

## 📖 SECTION 1: FILE DESCRIPTORS FUNDAMENTALS

### What is a File Descriptor?
A **file descriptor** is a small NON-NEGATIVE integer that represents an open file.

**Key Points:**
- File descriptor = index into process's file descriptor table
- Each process has its own file descriptor table
- 0, 1, 2 are always pre-opened:
  - **0 = stdin** (standard input)
  - **1 = stdout** (standard output)
  - **2 = stderr** (standard error)

### File Descriptor Table Structure
```
Process File Descriptor Table:
┌────┬──────────────────┐
│ 0  │ → stdin         │
│ 1  │ → stdout        │
│ 2  │ → stderr        │
│ 3  │ → (available)   │
│ 4  │ → (available)   │
│... │                 │
└────┴──────────────────┘
```

### Open File Table (System-wide)
```
File Descriptor → Open File Table Entry → Inode
                  (offset, flags, etc.)

Multiple FDs can point to SAME open file table entry!
```

---

## 📖 SECTION 2: OPEN() - OPENING FILES

### Syntax
```c
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);

// Returns: file descriptor on success, -1 on error
```

### Common Flags
```c
O_RDONLY    // Read only
O_WRONLY    // Write only
O_RDWR      // Read and write

O_CREAT     // Create if doesn't exist
O_TRUNC     // Truncate to 0 length
O_APPEND    // Append mode (writes at end)
O_EXCL      // Fail if file exists (with O_CREAT)
```

### Examples
```c
// Open for reading
int fd = open("file.txt", O_RDONLY);

// Open for writing, create if doesn't exist
int fd = open("file.txt", O_WRONLY | O_CREAT, 0644);

// Open for read/write, truncate
int fd = open("file.txt", O_RDWR | O_TRUNC);

// Open for append
int fd = open("file.txt", O_WRONLY | O_APPEND);
```

### Always Check for Errors!
```c
int fd = open("file.txt", O_RDONLY);
if (fd == -1) {
    perror("open failed");
    exit(1);
}
```

---

## 📖 SECTION 3: READ() AND WRITE()

### read() - Reading from File
```c
ssize_t read(int fd, void *buf, size_t count);

// Parameters:
//   fd: file descriptor
//   buf: buffer to store read data
//   count: maximum number of bytes to read

// Returns: 
//   number of bytes actually read
//   0 at EOF (end of file)
//   -1 on error
```

**CRITICAL:** read() may read FEWER bytes than requested!

### write() - Writing to File
```c
ssize_t write(int fd, const void *buf, size_t count);

// Parameters:
//   fd: file descriptor
//   buf: data to write
//   count: number of bytes to write

// Returns:
//   number of bytes actually written
//   -1 on error
```

### Examples
```c
// Read example
char buffer[100];
ssize_t n = read(fd, buffer, 100);
if (n == -1) {
    perror("read failed");
}
else if (n == 0) {
    printf("EOF reached\n");
}
else {
    printf("Read %ld bytes\n", n);
}

// Write example
const char *msg = "Hello";
ssize_t n = write(fd, msg, 5);
if (n == -1) {
    perror("write failed");
}
```

### How read() and write() Affect File Position
```
Initial position: 0

After read(fd, buf, 10):
  - Reads 10 bytes
  - Position moves forward by 10
  - New position: 10

After write(fd, buf, 5):
  - Writes 5 bytes at current position
  - Position moves forward by 5
  - New position: 15
```

---

## 📖 SECTION 4: LSEEK() - THE EXAM KILLER! 🎯

### What is lseek()?
**lseek()** changes the file position (file offset).

### Syntax
```c
off_t lseek(int fd, off_t offset, int whence);

// Returns: new file position, or -1 on error
```

### THE THREE CRITICAL FORMULAS - MEMORIZE THESE!

**Formula 1: SEEK_SET (whence = 0)**
```
new_position = offset
```
Sets position to ABSOLUTE offset from beginning.

**Formula 2: SEEK_CUR (whence = 1)**
```
new_position = current_position + offset
```
Sets position RELATIVE to current position.

**Formula 3: SEEK_END (whence = 2)**
```
new_position = file_size + offset
```
Sets position RELATIVE to end of file.

### Critical Rules for lseek()

**RULE 1:** lseek() does NOT change file size!
**RULE 2:** If calculated position < 0, lseek() FAILS (returns -1)
**RULE 3:** Failed lseek() does NOT change current position
**RULE 4:** Can seek beyond end of file (creates "hole")

---

## 📖 SECTION 5: THE T1 FILE I/O QUESTION - STEP BY STEP

### The Question
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

### The Complete Trace

```
═══════════════════════════════════════════════════════════

OPERATION 1: int fd = open("check.txt", O_RDWR);
  
  File size: 40 bytes
  Position: 0
  Status: ✓ File opened

───────────────────────────────────────────────────────────

OPERATION 2: lseek(fd, 100, SEEK_END);

  Formula: SEEK_END → new_pos = file_size + offset
  Calculation: new_pos = 40 + 100 = 140
  
  File size: 40 bytes (lseek doesn't change size!)
  Position: 140
  Status: ✓ Position moved beyond EOF

───────────────────────────────────────────────────────────

OPERATION 3: write(fd, buf, 9);

  Writes 9 bytes at position 140
  Position: 140 + 9 = 149
  File size: 149 (file grew! wrote beyond previous end)
  
  File size: 149 bytes
  Position: 149
  Status: ✓ File extended with hole from 40-140

───────────────────────────────────────────────────────────

OPERATION 4: write(fd, buf, 9);

  Writes 9 bytes at position 149
  Position: 149 + 9 = 158
  File size: 158
  
  File size: 158 bytes
  Position: 158
  Status: ✓ File extended

───────────────────────────────────────────────────────────

OPERATION 5: lseek(fd, -159, SEEK_CUR);

  Formula: SEEK_CUR → new_pos = current_pos + offset
  Calculation: new_pos = 158 + (-159) = -1
  
  ⚠️ NEGATIVE POSITION!
  lseek() FAILS and returns -1
  Position does NOT change!
  
  File size: 158 bytes (no change)
  Position: 158 (STAYS HERE!)
  Status: ✗ lseek failed

───────────────────────────────────────────────────────────

OPERATION 6: write(fd, buf, 9);

  Writes 9 bytes at position 158 (still there!)
  Position: 158 + 9 = 167
  File size: 167
  
  File size: 167 bytes
  Position: 167
  Status: ✓ Write at original position

───────────────────────────────────────────────────────────

OPERATION 7: write(fd, buf, 9);

  Writes 9 bytes at position 167
  Position: 167 + 9 = 176
  File size: 176
  
  File size: 176 bytes
  Position: 176
  Status: ✓ Write extended file

───────────────────────────────────────────────────────────

OPERATION 8: close(fd);

  Final file size: 176 bytes
  Status: ✓ File closed

═══════════════════════════════════════════════════════════

FINAL ANSWER: 176 bytes ✓
```

### Key Insights from This Question

**Insight 1:** lseek() NEVER changes file size
- It only moves the position pointer
- File size changes only with write()

**Insight 2:** Failed lseek() doesn't change position
- When new_pos < 0, lseek returns -1
- Position stays at the value before lseek

**Insight 3:** File size = furthest byte written
- File grows only when you write beyond current end
- File size = highest position reached + bytes written

**Insight 4:** Holes in files are allowed
- Seeking beyond EOF and writing creates a "hole"
- Hole contains zero bytes but doesn't occupy disk space

---

## 📖 SECTION 6: FILE SIZE VS POSITION

### Critical Distinction

**File Position (Offset):**
- Where next read/write will occur
- Changed by: read(), write(), lseek()
- Can be anywhere (even beyond file size)

**File Size:**
- Actual number of bytes in file
- Changed ONLY by: write() (and truncate)
- NOT changed by: lseek() or read()

### Example: Position vs Size
```c
// File initially has 50 bytes
int fd = open("file.txt", O_RDWR);

// Position: 0, Size: 50

lseek(fd, 100, SEEK_SET);
// Position: 100, Size: 50 (size unchanged!)

write(fd, "X", 1);
// Position: 101, Size: 101 (size grew!)

lseek(fd, 20, SEEK_SET);
// Position: 20, Size: 101 (size unchanged!)

write(fd, "Y", 1);
// Position: 21, Size: 101 (size unchanged - wrote within file)
```

---

## 📖 SECTION 7: DUP() AND DUP2()

### dup() - Duplicate File Descriptor
```c
int dup(int oldfd);

// Returns: new file descriptor (lowest available)
// New FD points to SAME open file table entry
```

**Example:**
```c
int fd1 = open("file.txt", O_RDONLY);  // fd1 = 3
int fd2 = dup(fd1);                     // fd2 = 4

// Both fd1 and fd2 point to SAME file
// Closing one doesn't affect the other
// They SHARE file position!
```

### dup2() - Duplicate to Specific FD
```c
int dup2(int oldfd, int newfd);

// Makes newfd a copy of oldfd
// If newfd is open, closes it first
// Returns: newfd on success, -1 on error
```

**Example:**
```c
int fd = open("output.txt", O_WRONLY | O_CREAT, 0644);
dup2(fd, 1);  // stdout (1) now points to output.txt
close(fd);

printf("This goes to output.txt!\n");
```

### Common Use: Redirecting stdout
```c
// Redirect stdout to file
int fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
dup2(fd, STDOUT_FILENO);
close(fd);

// Now all printf() goes to output.txt
printf("Redirected!\n");
```

### Counting dup2() in Pipelines (EXAM QUESTION!)

**Example:** `ls -l | grep txt | wc -l`

**Analysis:**
- 2 pipes needed (| is the pipe operator)
- Each pipe needs: 
  - Left command: redirect stdout to pipe write end (1 dup2)
  - Right command: redirect stdin from pipe read end (1 dup2)

**Breakdown:**
```
Pipe 1: ls to grep
  - ls: dup2(pipe1[1], STDOUT_FILENO)   // 1 dup2
  - grep: dup2(pipe1[0], STDIN_FILENO)  // 1 dup2

Pipe 2: grep to wc
  - grep: dup2(pipe2[1], STDOUT_FILENO) // 1 dup2
  - wc: dup2(pipe2[0], STDIN_FILENO)    // 1 dup2

Total: 4 dup2() calls
```

**BUT WAIT!** grep both reads AND writes, so:
```
Actually:
  - ls: 1 dup2
  - grep: 2 dup2 (stdin and stdout)
  - wc: 1 dup2
  
Total: 4 dup2() calls (but could be 6 depending on implementation!)
```

**EXAM TIP:** Count carefully! Usually 2 dup2 per pipe, but may vary!

---

## 📖 SECTION 8: CLOSE() AND FILE CLEANUP

### close() - Close File Descriptor
```c
int close(int fd);

// Returns: 0 on success, -1 on error
```

### When to close():
```c
int fd = open("file.txt", O_RDONLY);

// ... do work ...

close(fd);  // Always close when done!
```

### What happens when you don't close()?
- File descriptors are limited (typically 1024 per process)
- Running out = "Too many open files" error
- Process termination closes all FDs automatically

---

## 📖 SECTION 9: SPECIAL CONCEPTS

### Sparse Files (Holes)
```c
int fd = open("sparse.txt", O_WRONLY | O_CREAT, 0644);
write(fd, "A", 1);         // Position 0
lseek(fd, 1000000, SEEK_SET);
write(fd, "B", 1);         // Position 1000000

// File size: 1000001 bytes
// Actual disk usage: ~8KB (not 1MB!)
// Positions 1-999999 are a "hole" (all zeros)
```

### O_APPEND Mode
```c
int fd = open("file.txt", O_WRONLY | O_APPEND);

// Every write() automatically seeks to end first
// Even if you lseek() elsewhere, write goes to end!
lseek(fd, 0, SEEK_SET);    // Try to go to beginning
write(fd, "X", 1);         // Still writes at END!
```

### Atomic Operations
- **open() with O_CREAT | O_EXCL**: atomic create-or-fail
- **write() to O_APPEND**: atomic seek-and-write
- Prevents race conditions in concurrent access

---

## 🎯 TRUE/FALSE QUESTIONS - FILE I/O
### 70 Questions to Master File I/O!

---

### CATEGORY 1: File Descriptors Basics (Q1-15)

**Q1. T/F:** A file descriptor is a non-negative integer.
**ANSWER: TRUE** ✓
**Explanation:** FDs are 0, 1, 2, 3, ...

**Q2. T/F:** File descriptor 0 represents standard input (stdin).
**ANSWER: TRUE** ✓

**Q3. T/F:** File descriptor 1 represents standard error (stderr).
**ANSWER: FALSE** ✗
**Explanation:** 1 = stdout, 2 = stderr

**Q4. T/F:** File descriptor 2 represents standard error (stderr).
**ANSWER: TRUE** ✓

**Q5. T/F:** Each process has its own file descriptor table.
**ANSWER: TRUE** ✓

**Q6. T/F:** open() always returns file descriptor 3 for the first file opened.
**ANSWER: FALSE** ✗
**Explanation:** Returns lowest available FD (usually 3 if 0-2 are in use)

**Q7. T/F:** Multiple file descriptors can point to the same open file.
**ANSWER: TRUE** ✓
**Explanation:** After dup() or fork()

**Q8. T/F:** Closing one file descriptor closes all FDs pointing to that file.
**ANSWER: FALSE** ✗
**Explanation:** Each FD is independent

**Q9. T/F:** File descriptors are inherited by child processes after fork().
**ANSWER: TRUE** ✓

**Q10. T/F:** After fork(), parent and child share the same file position.
**ANSWER: TRUE** ✓
**Explanation:** They point to same open file table entry

**Q11. T/F:** A process can run out of file descriptors.
**ANSWER: TRUE** ✓
**Explanation:** Limited number (typically 1024)

**Q12. T/F:** stdin, stdout, stderr are always open when a process starts.
**ANSWER: TRUE** ✓

**Q13. T/F:** File descriptor -1 is a valid file descriptor.
**ANSWER: FALSE** ✗
**Explanation:** -1 indicates error

**Q14. T/F:** dup() creates a new file, duplicating the contents.
**ANSWER: FALSE** ✗
**Explanation:** Duplicates FD, not file contents

**Q15. T/F:** All processes share a single file descriptor table.
**ANSWER: FALSE** ✗
**Explanation:** Each process has its own table

---

### CATEGORY 2: open() System Call (Q16-25)

**Q16. T/F:** open() returns -1 on error.
**ANSWER: TRUE** ✓

**Q17. T/F:** O_RDONLY means read and write mode.
**ANSWER: FALSE** ✗
**Explanation:** Read only

**Q18. T/F:** O_RDWR allows both reading and writing.
**ANSWER: TRUE** ✓

**Q19. T/F:** O_CREAT creates the file if it doesn't exist.
**ANSWER: TRUE** ✓

**Q20. T/F:** O_TRUNC truncates the file to zero length.
**ANSWER: TRUE** ✓

**Q21. T/F:** O_APPEND makes all writes go to the end of file.
**ANSWER: TRUE** ✓

**Q22. T/F:** O_EXCL with O_CREAT fails if file already exists.
**ANSWER: TRUE** ✓

**Q23. T/F:** open() with O_CREAT requires a mode argument.
**ANSWER: TRUE** ✓
**Explanation:** Specifies file permissions

**Q24. T/F:** Opening a file moves the position to the end.
**ANSWER: FALSE** ✗
**Explanation:** Position starts at 0 (unless O_APPEND)

**Q25. T/F:** You can open the same file multiple times with different FDs.
**ANSWER: TRUE** ✓

---

### CATEGORY 3: read() and write() (Q26-40)

**Q26. T/F:** read() returns the number of bytes actually read.
**ANSWER: TRUE** ✓

**Q27. T/F:** read() always reads the exact number of bytes requested.
**ANSWER: FALSE** ✗
**Explanation:** May read fewer (e.g., at EOF, or interrupted)

**Q28. T/F:** read() returns 0 when EOF is reached.
**ANSWER: TRUE** ✓

**Q29. T/F:** read() returns -1 on error.
**ANSWER: TRUE** ✓

**Q30. T/F:** write() returns the number of bytes written.
**ANSWER: TRUE** ✓

**Q31. T/F:** write() can write fewer bytes than requested.
**ANSWER: TRUE** ✓
**Explanation:** Possible but rare for regular files

**Q32. T/F:** After read(), the file position moves forward.
**ANSWER: TRUE** ✓
**Explanation:** By the number of bytes read

**Q33. T/F:** After write(), the file position moves forward.
**ANSWER: TRUE** ✓
**Explanation:** By the number of bytes written

**Q34. T/F:** read() changes the file size.
**ANSWER: FALSE** ✗
**Explanation:** Only reading, doesn't modify size

**Q35. T/F:** write() can change the file size.
**ANSWER: TRUE** ✓
**Explanation:** If writing beyond current end

**Q36. T/F:** Writing to a file with O_APPEND ignores lseek().
**ANSWER: TRUE** ✓
**Explanation:** Always writes at end regardless of position

**Q37. T/F:** read() from stdin uses file descriptor 0.
**ANSWER: TRUE** ✓

**Q38. T/F:** write() to stdout uses file descriptor 1.
**ANSWER: TRUE** ✓

**Q39. T/F:** write() to stderr uses file descriptor 2.
**ANSWER: TRUE** ✓

**Q40. T/F:** Multiple processes can read from the same file simultaneously without issues.
**ANSWER: TRUE** ✓
**Explanation:** Reading is safe; concurrent writes may cause issues

---

### CATEGORY 4: lseek() - THE CRITICAL TOPIC! (Q41-60)

**Q41. T/F:** lseek() changes the file position.
**ANSWER: TRUE** ✓

**Q42. T/F:** lseek() changes the file size.
**ANSWER: FALSE** ✗
**Explanation:** NEVER changes size, only position!

**Q43. T/F:** lseek() returns the new file position on success.
**ANSWER: TRUE** ✓

**Q44. T/F:** lseek() returns -1 on error.
**ANSWER: TRUE** ✓

**Q45. T/F:** SEEK_SET sets position relative to beginning of file.
**ANSWER: TRUE** ✓
**Explanation:** new_pos = offset

**Q46. T/F:** SEEK_CUR sets position relative to current position.
**ANSWER: TRUE** ✓
**Explanation:** new_pos = current + offset

**Q47. T/F:** SEEK_END sets position relative to end of file.
**ANSWER: TRUE** ✓
**Explanation:** new_pos = file_size + offset

**Q48. T/F:** lseek(fd, 0, SEEK_SET) moves to beginning of file.
**ANSWER: TRUE** ✓

**Q49. T/F:** lseek(fd, 0, SEEK_END) moves to end of file.
**ANSWER: TRUE** ✓

**Q50. T/F:** lseek(fd, -10, SEEK_CUR) moves backward 10 bytes.
**ANSWER: TRUE** ✓

**Q51. T/F:** lseek() can fail if the calculated position is negative.
**ANSWER: TRUE** ✓
**Explanation:** Position < 0 is invalid

**Q52. T/F:** If lseek() fails, the file position is set to 0.
**ANSWER: FALSE** ✗
**Explanation:** Position doesn't change on failure

**Q53. T/F:** If lseek() fails, the file position remains unchanged.
**ANSWER: TRUE** ✓
**Explanation:** CRITICAL! Failed lseek doesn't move position

**Q54. T/F:** You can seek beyond the end of a file.
**ANSWER: TRUE** ✓
**Explanation:** Creates a "hole"

**Q55. T/F:** Seeking beyond EOF and writing creates a sparse file.
**ANSWER: TRUE** ✓

**Q56. T/F:** In the T1 question, lseek(fd, -159, SEEK_CUR) succeeds.
**ANSWER: FALSE** ✗
**Explanation:** Results in negative position, fails!

**Q57. T/F:** After a failed lseek(), subsequent write() uses the original position.
**ANSWER: TRUE** ✓
**Explanation:** Position didn't change

**Q58. T/F:** lseek(fd, 10, SEEK_SET) sets position to 10.
**ANSWER: TRUE** ✓
**Explanation:** SEEK_SET: new_pos = offset = 10

**Q59. T/F:** If file size is 50, lseek(fd, 10, SEEK_END) sets position to 60.
**ANSWER: TRUE** ✓
**Explanation:** SEEK_END: new_pos = 50 + 10 = 60

**Q60. T/F:** If position is 100, lseek(fd, -50, SEEK_CUR) sets position to 50.
**ANSWER: TRUE** ✓
**Explanation:** SEEK_CUR: new_pos = 100 + (-50) = 50

---

### CATEGORY 5: Advanced Concepts (Q61-70)

**Q61. T/F:** close() releases the file descriptor.
**ANSWER: TRUE** ✓

**Q62. T/F:** Forgetting to close() a file causes a memory leak.
**ANSWER: FALSE** ✗
**Explanation:** FD leak, not memory leak (though related)

**Q63. T/F:** All file descriptors are closed when process terminates.
**ANSWER: TRUE** ✓

**Q64. T/F:** dup2(oldfd, newfd) makes newfd a copy of oldfd.
**ANSWER: TRUE** ✓

**Q65. T/F:** After dup2(), both FDs share the same file position.
**ANSWER: TRUE** ✓

**Q66. T/F:** dup2() closes newfd if it's already open.
**ANSWER: TRUE** ✓

**Q67. T/F:** In a pipeline with 3 commands, there are 4 dup2() calls.
**ANSWER: TRUE** ✓ (Usually)
**Explanation:** 2 pipes × 2 dup2 each = 4

**Q68. T/F:** O_APPEND ensures atomic append operations.
**ANSWER: TRUE** ✓

**Q69. T/F:** A sparse file with a 1GB hole uses 1GB of disk space.
**ANSWER: FALSE** ✗
**Explanation:** Holes don't consume disk space

**Q70. T/F:** lseek() can be used with pipes.
**ANSWER: FALSE** ✗
**Explanation:** Pipes don't support seeking

---

## 🔥 EXAM STRATEGY FOR FILE I/O QUESTIONS

### The Tracing Method for lseek() Questions

**Step 1: Create a tracking table**
```
| Operation | Formula | Calculation | New Pos | New Size |
|-----------|---------|-------------|---------|----------|
| Initial   | -       | -           | 0       | ??       |
```

**Step 2: For each operation**
- Identify if it's lseek, read, or write
- Apply the correct formula
- Check if lseek fails (position < 0)
- Update position and size

**Step 3: Remember the rules**
- lseek NEVER changes size
- Failed lseek doesn't change position
- write changes both position AND size (if beyond end)
- read changes position but NOT size

**Step 4: Calculate final answer**
- Usually asking for final file size
- File size = furthest byte written

---

## 📝 QUICK REFERENCE - MEMORIZE THIS!

```
THE THREE LSEEK FORMULAS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SEEK_SET (0): new_position = offset
SEEK_CUR (1): new_position = current_position + offset
SEEK_END (2): new_position = file_size + offset

CRITICAL RULES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. lseek() NEVER changes file size
2. Failed lseek() (pos < 0) → position unchanged
3. File size changes ONLY with write()
4. File size = furthest byte written

POSITION VS SIZE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Position: Where next read/write happens
Size: Actual bytes in file

lseek()  → Changes: Position  | Size: NO
read()   → Changes: Position  | Size: NO
write()  → Changes: Position  | Size: YES (if beyond end)

FILE DESCRIPTORS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
0 = stdin
1 = stdout
2 = stderr
```

---

## ✅ CHECKLIST - DID YOU MASTER THIS?

- □ I know the three lseek formulas by heart
- □ I can trace file I/O operations step-by-step
- □ I understand position vs file size
- □ I know when lseek fails (negative position)
- □ I remember: failed lseek doesn't change position
- □ I can solve the T1 file I/O question in under 4 minutes
- □ I got 80%+ on the True/False questions

---

## 🎯 PRACTICE PROBLEM

**Try this yourself:**
```c
int fd = open("test.txt", O_RDWR);  // Size = 30
lseek(fd, 50, SEEK_SET);
write(fd, "X", 1);
lseek(fd, -60, SEEK_CUR);
write(fd, "Y", 1);
close(fd);

// What is the final size?
```

**SOLUTION:**
```
Initial: size=30, pos=0
lseek(50, SEEK_SET): pos=50, size=30
write 1: pos=51, size=51
lseek(-60, SEEK_CUR): 51-60=-9 → FAILS! pos=51, size=51
write 1: pos=52, size=52
Final: 52 bytes
```

---

## 🎯 NEXT STEP

**Move to PART 3: SIGNALS & SIGNAL HANDLING**

You've mastered File I/O! Time to conquer Signals!

---

**END OF PART 2/9**

**Progress: [████████░] 22% Complete**

**You're crushing it! Keep going! 💪🔥**
