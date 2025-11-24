# 🏆 COMP8567 ULTIMATE EXAM GUIDE - PART 5/9
## THREADS & CONCURRENCY
## Master Threads and the "Main Returns → All Die" Rule!

---

# 📚 PART 5: THREADS & CONCURRENCY

## 🎯 WHAT YOU'LL MASTER:
- ✅ Threads vs Processes
- ✅ pthread_create, pthread_join, pthread_cancel
- ✅ Shared vs Private memory
- ✅ THE CRITICAL RULE: main() returns → ALL threads die!
- ✅ T2 thread question pattern
- ✅ 65+ True/False questions!

---

## 📖 SECTION 1: THREADS VS PROCESSES

### What is a Thread?
**Thread** = lightweight execution unit within a process

### Threads vs Processes
```
PROCESS:
  ✓ Separate memory space
  ✓ Heavy-weight creation
  ✓ IPC needed for communication
  ✓ Isolated from each other

THREAD:
  ✓ SHARED memory space
  ✓ Light-weight creation
  ✓ Direct communication (shared variables)
  ✓ Part of same process
```

---

## 📖 SECTION 2: PTHREAD BASICS

### pthread_create - Create Thread
```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                  void *(*start_routine)(void*), void *arg);

// Returns: 0 on success, error number on failure
```

**Example:**
```c
void* my_function(void* arg) {
    printf("Thread running!\n");
    return NULL;
}

pthread_t thread;
pthread_create(&thread, NULL, my_function, NULL);
```

### pthread_join - Wait for Thread
```c
int pthread_join(pthread_t thread, void **retval);

// Returns: 0 on success, error number on failure
// Blocks until thread terminates (like waitpid for threads)
```

### pthread_cancel - Cancel Thread
```c
int pthread_cancel(pthread_t thread);

// Requests thread termination
```

### pthread_self - Get Own Thread ID
```c
pthread_t pthread_self(void);

// Returns calling thread's ID
```

---

## 📖 SECTION 3: SHARED VS PRIVATE MEMORY

### Global Variables = SHARED
```c
int counter = 0;  // ← ALL threads see SAME counter!

void* thread_func(void* arg) {
    counter++;  // All threads modify SAME counter
    return NULL;
}
```

### Local Variables = PRIVATE
```c
void* thread_func(void* arg) {
    int local = 0;  // ← Each thread has OWN copy!
    local++;
    return NULL;
}
```

---

## 📖 SECTION 4: THE T2 THREAD QUESTION - CRITICAL!

### The Question
```c
int counter = 0;  // SHARED!

void* func1(void* p) {
    while(1) {
        printf("thread one\n");
        sleep(1);
        counter++;
        if(counter == 3) {
            pthread_cancel(pthread_self());
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
    
    pthread_join(thread_one, NULL);
    
    return 0;  // ← CRITICAL LINE!
}
```

### The Complete Trace

```
═══════════════════════════════════════════════════════════

TIME: 0 seconds
  - Main creates 3 threads
  - All 3 start IMMEDIATELY
  - Main blocks at pthread_join(thread_one)

───────────────────────────────────────────────────────────

TIME: 0-1 second
  - thread_one: prints, sleeps
  - thread_two: prints, sleeps
  - thread_three: prints, sleeps

───────────────────────────────────────────────────────────

TIME: 1 second
  - thread_one wakes up
  - counter++ → counter = 1
  - counter == 3? NO
  - All threads print again

───────────────────────────────────────────────────────────

TIME: 2 seconds
  - thread_one wakes up
  - counter++ → counter = 2
  - counter == 3? NO
  - All threads print again

───────────────────────────────────────────────────────────

TIME: 3 seconds
  - thread_one wakes up
  - counter++ → counter = 3
  - counter == 3? YES!
  - pthread_cancel(pthread_self())
  - thread_one EXITS
  
  ⚡ pthread_join RETURNS
  ⚡ main() RETURNS
  
  🚨 CRITICAL: When main() returns, ALL THREADS DIE!
  
  - thread_two KILLED (even though infinite loop!)
  - thread_three KILLED (even though infinite loop!)

═══════════════════════════════════════════════════════════

RESULT: Program runs ~3 seconds, then terminates
        All threads die when main() returns!
```

### 🔑 THE GOLDEN RULE - MEMORIZE THIS!

**WHEN main() RETURNS → ALL THREADS ARE KILLED!**
**Even if threads are in infinite loops!**

This is THE most common exam trap!

---

## 📖 SECTION 5: THREAD SYNCHRONIZATION

### Mutex (Mutual Exclusion)
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&mutex);
// Critical section (only one thread at a time)
counter++;
pthread_mutex_unlock(&mutex);
```

### Why Synchronization?
```c
// WITHOUT MUTEX - RACE CONDITION!
int counter = 0;

void* increment(void* arg) {
    for(int i=0; i<1000000; i++)
        counter++;  // NOT ATOMIC!
    return NULL;
}

// counter may NOT be 2000000 due to race!
```

---

## 🎯 TRUE/FALSE QUESTIONS - THREADS (65 Questions)

### CATEGORY 1: Thread Basics (Q1-20)

**Q1. T/F:** Threads share the same memory space.
**ANSWER: TRUE** ✓

**Q2. T/F:** Threads are lighter-weight than processes.
**ANSWER: TRUE** ✓

**Q3. T/F:** pthread_create() creates a new thread.
**ANSWER: TRUE** ✓

**Q4. T/F:** pthread_create() returns 0 on success.
**ANSWER: TRUE** ✓

**Q5. T/F:** Thread starts executing immediately after pthread_create().
**ANSWER: TRUE** ✓

**Q6. T/F:** pthread_join() blocks until thread finishes.
**ANSWER: TRUE** ✓

**Q7. T/F:** pthread_join() is like waitpid() for threads.
**ANSWER: TRUE** ✓

**Q8. T/F:** pthread_cancel() immediately kills the thread.
**ANSWER: FALSE** ✗
**Explanation:** Requests cancellation, actual termination at cancellation point

**Q9. T/F:** pthread_self() returns own thread ID.
**ANSWER: TRUE** ✓

**Q10. T/F:** Each thread has its own stack.
**ANSWER: TRUE** ✓

**Q11. T/F:** All threads in a process share the heap.
**ANSWER: TRUE** ✓

**Q12. T/F:** Threads share file descriptors.
**ANSWER: TRUE** ✓

**Q13. T/F:** Threads have separate PIDs.
**ANSWER: FALSE** ✗
**Explanation:** Same PID, different TIDs

**Q14. T/F:** fork() from multi-threaded program duplicates all threads.
**ANSWER: FALSE** ✗
**Explanation:** Only calling thread is duplicated

**Q15. T/F:** exec() terminates all threads except caller.
**ANSWER: TRUE** ✓

**Q16. T/F:** A process can have multiple threads.
**ANSWER: TRUE** ✓

**Q17. T/F:** Thread creation is faster than process creation.
**ANSWER: TRUE** ✓

**Q18. T/F:** Threads eliminate need for IPC.
**ANSWER: TRUE** ✓
**Explanation:** Can use shared variables

**Q19. T/F:** pthread functions return 0 on success.
**ANSWER: TRUE** ✓

**Q20. T/F:** pthread functions return -1 on error.
**ANSWER: FALSE** ✗
**Explanation:** Return error number (not -1)

---

### CATEGORY 2: Shared vs Private (Q21-35)

**Q21. T/F:** Global variables are shared by all threads.
**ANSWER: TRUE** ✓

**Q22. T/F:** Local variables are shared by all threads.
**ANSWER: FALSE** ✗
**Explanation:** Each thread has own copy

**Q23. T/F:** Static variables are shared by all threads.
**ANSWER: TRUE** ✓

**Q24. T/F:** Heap memory is shared by all threads.
**ANSWER: TRUE** ✓

**Q25. T/F:** Stack memory is private to each thread.
**ANSWER: TRUE** ✓

**Q26. T/F:** In T2 question, counter is shared.
**ANSWER: TRUE** ✓

**Q27. T/F:** Multiple threads can modify same variable without synchronization.
**ANSWER: TRUE** ✓ (possible but dangerous!)
**Explanation:** Possible but causes race conditions

**Q28. T/F:** Race conditions occur when threads access shared data.
**ANSWER: TRUE** ✓

**Q29. T/F:** Function parameters are private to each thread.
**ANSWER: TRUE** ✓

**Q30. T/F:** malloc'd memory is shared between threads.
**ANSWER: TRUE** ✓
**Explanation:** Heap is shared

**Q31. T/F:** const global variables are still shared.
**ANSWER: TRUE** ✓

**Q32. T/F:** File offsets are shared between threads.
**ANSWER: TRUE** ✓

**Q33. T/F:** Thread-local storage (TLS) is private.
**ANSWER: TRUE** ✓

**Q34. T/F:** Signal handlers are per-thread.
**ANSWER: FALSE** ✗
**Explanation:** Per-process (shared)

**Q35. T/F:** errno is thread-safe in POSIX.
**ANSWER: TRUE** ✓
**Explanation:** Each thread has own errno

---

### CATEGORY 3: THE CRITICAL RULE! (Q36-50)

**Q36. T/F:** When main() returns, all threads are terminated.
**ANSWER: TRUE** ✓
**Explanation:** 🔑 GOLDEN RULE!

**Q37. T/F:** Threads continue after main() exits.
**ANSWER: FALSE** ✗
**Explanation:** ALL KILLED!

**Q38. T/F:** In T2 question, thread_two runs forever.
**ANSWER: FALSE** ✗
**Explanation:** Dies when main() returns!

**Q39. T/F:** In T2 question, thread_three survives.
**ANSWER: FALSE** ✗
**Explanation:** Dies when main() returns!

**Q40. T/F:** pthread_join() keeps threads alive after main().
**ANSWER: FALSE** ✗
**Explanation:** Only if main hasn't returned

**Q41. T/F:** In T2 question, program runs ~3 seconds.
**ANSWER: TRUE** ✓

**Q42. T/F:** In T2 question, counter reaches 3.
**ANSWER: TRUE** ✓

**Q43. T/F:** In T2 question, func1 exits first.
**ANSWER: TRUE** ✓

**Q44. T/F:** After thread_one exits, main() returns.
**ANSWER: TRUE** ✓
**Explanation:** pthread_join returns

**Q45. T/F:** Infinite loops in threads prevent program termination.
**ANSWER: FALSE** ✗
**Explanation:** main() return kills all!

**Q46. T/F:** pthread_exit() in main() keeps threads alive.
**ANSWER: TRUE** ✓
**Explanation:** Difference from return!

**Q47. T/F:** exit() kills all threads.
**ANSWER: TRUE** ✓

**Q48. T/F:** _exit() kills all threads.
**ANSWER: TRUE** ✓

**Q49. T/F:** Detached threads continue after main().
**ANSWER: FALSE** ✗
**Explanation:** Still die when process ends

**Q50. T/F:** Only threads that called pthread_join survive.
**ANSWER: FALSE** ✗
**Explanation:** ALL die when main() returns

---

### CATEGORY 4: Synchronization (Q51-65)

**Q51. T/F:** Mutex provides mutual exclusion.
**ANSWER: TRUE** ✓

**Q52. T/F:** pthread_mutex_lock() blocks if mutex already locked.
**ANSWER: TRUE** ✓

**Q53. T/F:** Only the thread that locked mutex can unlock it.
**ANSWER: TRUE** ✓

**Q54. T/F:** Forgetting to unlock mutex causes deadlock.
**ANSWER: TRUE** ✓

**Q55. T/F:** counter++ is an atomic operation.
**ANSWER: FALSE** ✗
**Explanation:** Requires synchronization!

**Q56. T/F:** Two threads incrementing same variable causes race.
**ANSWER: TRUE** ✓

**Q57. T/F:** Mutexes prevent race conditions.
**ANSWER: TRUE** ✓

**Q58. T/F:** Condition variables signal events.
**ANSWER: TRUE** ✓

**Q59. T/F:** Semaphores can be used for thread sync.
**ANSWER: TRUE** ✓

**Q60. T/F:** Read-write locks allow multiple readers.
**ANSWER: TRUE** ✓

**Q61. T/F:** Spinlocks are more efficient for short waits.
**ANSWER: TRUE** ✓

**Q62. T/F:** Deadlock occurs when threads wait for each other.
**ANSWER: TRUE** ✓

**Q63. T/F:** pthread_mutex_trylock() never blocks.
**ANSWER: TRUE** ✓

**Q64. T/F:** Barriers synchronize multiple threads.
**ANSWER: TRUE** ✓

**Q65. T/F:** Thread-safe functions can be called concurrently.
**ANSWER: TRUE** ✓

---

## 📝 QUICK REFERENCE

```
THREAD CREATION:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
pthread_create(&thread, NULL, func, arg);
pthread_join(thread, NULL);
pthread_cancel(thread);
pthread_self();

MEMORY MODEL:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SHARED:  Global vars, heap, static vars
PRIVATE: Local vars, stack, function params

🔑 GOLDEN RULE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
When main() returns → ALL threads die!
(Even in infinite loops!)

T2 PATTERN:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Thread checks counter
2. Counter reaches limit
3. Thread exits (pthread_cancel or return)
4. pthread_join returns
5. main() returns
6. ALL OTHER THREADS DIE
```

---

## ✅ CHECKLIST

- □ I know shared vs private memory
- □ I memorized: main() returns → all die
- □ I can trace T2 thread question
- □ I understand pthread_join blocks
- □ I got 85%+ on T/F questions

---

**END OF PART 5/9**
**Progress: [████████████████████░] 56% Complete**
