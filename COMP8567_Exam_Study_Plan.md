# COMP8567 EXAM STUDY PLAN
## Target: 70%+ on MCQ/T-F Exam (Nov 28, 4 PM)
## Strategy: CODE TRACING MASTERY

---

## 📅 OVERVIEW: 4.5 Days to Exam

**Today:** Sunday, November 24, 2025  
**Exam:** Thursday, November 28, 2025 at 4:00 PM  
**Total Study Time Available:** 33 hours
- Sunday (today): 4 hours
- Monday: 4 hours  
- Tuesday: 5 hours
- Wednesday: 10 hours
- Thursday: 10 hours (exam day - study morning till 4 PM)

#### **Morning Session: 9:00 AM - 12:00 PM (3 hours)**
**Topic: POINTERS + Review Uploaded Materials**

**9:00 - 10:00 AM: Pointer Tracing Basics**
- Read pointer sections from uploaded PPTs
- Create POINTER TRACING TEMPLATE:
  ```
  Address | Variable | Value | Notes
  --------|----------|-------|-------
  1000    | x        | 5     | int
  1004    | ptr      | 1000  | points to x
  ```
- Practice 5 simple pointer trace problems

**10:00 - 11:00 AM: Advanced Pointer Concepts**
- Arrays as pointers
- Pointer arithmetic
- Double pointers
- Practice 5 advanced problems

**11:00 - 12:00 PM: Create Pointer Reference Sheet**
- Syntax rules
- Common patterns
- Tracing checklist
- **PRINT THIS SHEET**

---

#### **Afternoon Session: 1:00 PM - 6:00 PM (5 hours)**
**Topic: FORK - Process Tree Mastery**

**1:00 - 2:30 PM: Fork Tracing Template Development**
- Review Process_Control_F25.pptx thoroughly
- Study the T1 fork question (pid3 example)
- Create FORK TRACING TEMPLATE:
  ```
  Step 1: Draw initial state [Parent PID=1000]
  Step 2: After each fork(), split the tree
  Step 3: Label return values (Parent: child PID, Child: 0)
  Step 4: Track which processes execute which code
  Step 5: Count processes OR track variable values
  ```
- **KEY INSIGHT:** Last assignment wins (variable overwriting)

**2:30 - 4:00 PM: Fork Practice Problems**
Create FORK CODE TRACING CHECKLIST:
- □ Draw process tree
- □ Label each fork's return value for each process
- □ Track which processes execute which code blocks
- □ Count processes OR track variable values
- □ Final answer: _______

**Practice 10 problems:**
1. Count processes in T2 sample fork question
2. Sequential forks: `fork(); fork(); fork();` → ? processes
3. Loop forks: `for(i=0; i<3; i++) fork();` → ? processes
4. Conditional forks: `if(fork()) fork();` → ? processes
5. Track variable values across forks
6. waitpid() return value tracing
7. Multiple waitpid() calls (like T1 question)
8. Fork with exit() calls
9. Fork with exec() calls
10. Complex nested forks

**4:00 - 5:00 PM: Master the T1 Fork Question**
Trace step-by-step multiple times until you can do it in under 2 minutes:
```c
int pid1, pid2, pid3;
pid1 = fork();
pid2 = fork();
pid3 = fork();

if(pid1>0 && pid2>0 && pid3>0){
    int child_pid;
    child_pid = waitpid(pid1, &status, 0);  // child_pid = pid1
    child_pid = waitpid(pid2, &status, 0);  // child_pid = pid2
    child_pid = waitpid(pid3, &status, 0);  // child_pid = pid3
    printf("%d", child_pid);  // Prints pid3!
}
```

**5:00 - 6:00 PM: Create Fork Reference Sheet**
```
FORK REFERENCE SHEET
====================
fork() return values:
  Parent: child's PID (positive)
  Child: 0
  Error: -1

Common patterns:
  for(i=0;i<n;i++) fork() → 2^n processes
  n sequential forks → 2^n processes
  if(fork()) fork() → 6 processes (special case)

waitpid(pid, &status, 0):
  Returns: PID of child that terminated
  Blocks until that specific child exits

Tracing strategy:
  1. Draw tree after each fork
  2. Label return values for ALL processes
  3. Track what each process executes
  4. Count or calculate final values

CRITICAL: Variables get OVERWRITTEN
  x = 1;
  x = 2;  // x is now 2
  x = 3;  // x is now 3 (last one wins!)
```
- **PRINT THIS BEFORE DINNER**

---

#### **Evening Session: 7:00 PM - 11:00 PM (4 hours)**
**Topic: FILE I/O - Position Tracking Mastery**

**7:00 - 8:00 PM: File I/O Tracing Template**
- Review Unix_File_Input_Output_F25.pptx
- Create FILE I/O TRACING TEMPLATE:
  ```
  Initial state:
    File size: _____
    Position: 0

  After each operation:
    Operation: _____________
    New Position: _____
    New File Size: _____

  Final answer: _____
  ```

**8:00 - 9:30 PM: Master the T1 File I/O Question**
Trace this repeatedly until perfect:
```c
int fd = open("check.txt", O_RDWR);  // Size = 40
lseek(fd, 100, SEEK_END);           // Pos: 140
write(fd, buf, 9);                   // Pos: 149, Size: 149
write(fd, buf, 9);                   // Pos: 158, Size: 158
lseek(fd, -159, SEEK_CUR);          // FAILS! Pos stays 158
write(fd, buf, 9);                   // Pos: 167, Size: 167
write(fd, buf, 9);                   // Pos: 176, Size: 176
close(fd);
// ANSWER: 176 bytes
```

**Practice tracking:**
- SEEK_SET: absolute positioning
- SEEK_CUR: relative to current
- SEEK_END: relative to file end
- Negative offsets (can fail!)
- File size = furthest byte written

**9:30 - 10:30 PM: File I/O Practice Problems**
Create 10 tracing problems:
1. Simple lseek with SEEK_SET
2. lseek with SEEK_CUR
3. lseek with SEEK_END
4. Multiple writes with positioning
5. Read operations (position changes)
6. Failed lseek (negative position)
7. Write beyond current file size
8. Overwriting existing data
9. Mixed read/write operations
10. Complex multi-operation sequences

**10:30 - 11:00 PM: Create File I/O Reference Sheet**
```
FILE I/O REFERENCE SHEET
========================
lseek(fd, offset, whence):
  SEEK_SET (0): position = offset
  SEEK_CUR (1): position = current + offset
  SEEK_END (2): position = filesize + offset
  Returns: new position, or -1 on error

read(fd, buf, n):
  Reads up to n bytes
  Position moves forward by bytes_read
  Returns: bytes read, or 0 at EOF, or -1 on error

write(fd, buf, n):
  Writes n bytes at current position
  Position moves forward by n
  File grows if position > current size
  Returns: bytes written, or -1 on error

TRACING STEPS:
1. Start: size = ___, pos = 0
2. After each operation:
   - Calculate new position using formula
   - Check if lseek failed (negative pos)
   - Update file size if write beyond end
3. Final size = furthest byte written

CRITICAL RULES:
- lseek can fail (returns -1) if negative position
- Failed lseek does NOT change position
- File size = max byte position written + 1
- Holes in file (sparse files) possible
```
- **PRINT THIS BEFORE BED**

---

### **MONDAY, NOV 25 - 8 hours**

#### **Evening Session: 6:00 PM - 8:00 PM (2 hours)**
**Topic: SIGNALS - Handler Execution Counting**

**6:00 - 7:00 PM: Signal Tracing Template**
- Review Signals_F25.pptx
- Create SIGNAL TRACING TEMPLATE:
  ```
  Step 1: Count total processes
  
  Step 2: Identify process groups
    - Default: all in parent's group
    - setpgid(0, getpid()): creates new group
  
  Step 3: When signal sent:
    - Which group receives signal?
    - Which processes have handlers?
    - What does each handler do?
  
  Step 4: Count executions or survivors
  ```

**7:00 - 8:00 PM: Signal Practice**
Key concepts:
- signal() registration
- alarm() - ONE-SHOT timer
- Process groups (setpgid)
- Ctrl-C kills foreground group only
- Common signals: SIGINT, SIGALRM, SIGTERM, SIGKILL

Practice the T1 alarm question:
```c
void handler1(int signo){
    signal(SIGALRM, handler1);  // Re-register
    printf("\nIn the handler\n");
}

int main(){
    signal(SIGALRM, handler1);
    alarm(8);  // Signal in 8 seconds
    
    for(int i=0; i<10; i++)
        sleep(1);  // Sleeps 10 seconds total
}
// Handler called: 1 time only (no new alarm set!)
```

---

#### **Night Session: 11:00 PM - 2:00 AM (3 hours)**
**Topic: Intensive Fork + File I/O Review**

**11:00 PM - 12:00 AM: Fork Speed Drills**
- 10 fork problems
- Time limit: 3 minutes each
- Use your template
- Check answers

**12:00 - 1:00 AM: File I/O Speed Drills**
- 10 file I/O problems
- Time limit: 3 minutes each
- Focus on lseek calculations
- Track position