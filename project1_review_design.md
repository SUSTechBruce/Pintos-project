# **Design review**

# Task1: Efficient Alarm Clock

# Data structures and functions

## timer.c
` enum intr_level{INTR_OFF,INTR_ON};`
- Data structures enum for interrupts disabled and interrupt enabled.
 
` enum intr_level Intr_enable(void);`
- Enable interrupts and returns the previous interrupt status.
 
` enum intr_level intr_disable(void);`
- disable interrupts and returns the previous interrupt status.

` static int64_t record_blocked;`
- Record how much time the thread has been blocked.

`.timer_interrupt(struct intr_fame *args UNUSED)`
- For timer interrupt handler.

## Thread.h
` int64_t record_blocked;`
 - Record how much time the thread has been blocked.
 
`Struct thread{tid_t tid; enum thread_status status; char name[16]; unin8_t *stack; int priority; struct list_elem allelem; struct    list_elem elem; uint32_t *pagedir; unsigned magic; int64_t record_blocked;}`
  - Structure of thread for implementation of algorithm;
  
 `tid_t thread_create(const char *name, int priority, thread_func *, void *)`
  - Create a new kernel thread named NAME.
  
  ## Thread.c
  `void thread_block(void);`
  - Put the current thread to sleep until the calling of thread_unblock;
  
  `void thread_unblock(struct thread *);`
  - Put the thread into ready queue and keep on using it.
  
 `check_thread_status(struct thread *thre, void *aux UNUSED);`
  - Check the blocked thread status, if the blocked time is deducted to 0, then call the function `thread_unblock()`;
  
  
# Algorithm

   - When a thread is sleeps, if we want it doesn’t occupy the resource of cpu, we need to use a counter called `record_blocked` to record how much time the thread has slept, so I add the function `check_thread_status` to detect each thread’s status(blocked or not) and record_blocked’s time, if the conditions do not satisfy sleep time , it will call function `thread_unblock` to arouse the thread and put it into ready queue to keep on executing.
  
# Synchronization 

   - First of all, in order to ensure the synchronization of the thread, I will use the variable `int64_t tricks_blocked` that record thread blocked time and add it in the `struct thread` data structure to ensure that all threads have this property. Secondly, I add the new function check_thread_status to the `thread_foreach` function to ensure that the judgment of the thread state and sleep time is performed for each thread. Third, if this function does not match the condition of the blocked time, it will be woken up and put back into the ready queue, which is also a guarantee of synchronization.
   
# Rationale
  
   - When timer_sleep is used, the thread is directly blocked, and then a member `int64_t record_blocked` is added to the thread structure to record how much time the thread has been sleeping. Then, the operating system's own clock interrupt (each tick is executed once) is added to the thread state. Detect, subtract ticks_blocked by 1 for each test, and wake up this thread if it is reduced to zero.
   
   
 
# Task2: Priority Scheduler

# Data structures and functions

## thread.h

`Struct thread{tid_t tid; enum thread_status status; char name[16]; unin8_t *stack; int priority; struct list_elem allelem; struct list_elem elem; uint32_t *pagedir; unsigned magic; int64_t record_blocked;}`
- Using the struct thread including int priority in the algorithm.

`struct list_elem elem;`
- List element for priority;

`int max_priority;`
- Max priority among the threads acquiring the locks

`int rudimentary_priority;`
- Rudimentary for priority;

`struct list locks;`
- Locks that the thread is holding;

`struct lock *lock_waiting;`
- The lock that the thread is waiting for.

## list.c

`struct list_elem{struct list_elem *prev; struct list_elem *next};`
- Structure for list element;

`struct list {struct list_elem head; struct list_elem tail};`
- Structure for list;

`list_insert_ordered (&all_list, &t->allelem, (list_less_func *) &thread_cmp_priority, NULL);`
- Inserts ELEM in the proper position in LIST, which must be sorted according to LESS given auxiliary data AUX

## thread.c

`void thread_set_priority(int new_priority);`
- Set the current thread’s priority to new_priority.

`void thread_yield(void);`
- Yields the CPU.

## Synch.c
`void lock_acquire(struct lock *lock);`
- Releases lock, which must be owned by current thread;

# Algorithm

-  **Step 1** : In the arousing execution of the thread, we need to call the `thread_unblock` function, but the disadvantage of this function is that when the thread is placed into the ready list, there is no guarantee that the list will be sorted according to the priority order. When I look at the source code of `list.c`, I found that the `list_insert_ordered` function can be sorted in order of priority, so I need to change the original insert queue method to `list_insert_ordered`.-`

- **Step 2** : When we want to reset the thread priority, we need to rethink the execution order of all threads, so we need to change the priority, we using function `thread_set_priority`，and then throw the thread into the ready queue to continue execution, so we need to call the `thread_yield` function, this is A good way to ensure that queues are executed in order of priority.

- **Step 3** :When a thread acquires a lock, if the thread with the lock has a lower priority than its own, it raises the priority of owning the lock. If the lock is locked by another lock, the priority is recursively given. After the thread releases the lock, the priority under the undone logic is restored, that is, the original priority of the thread.So I have to implement the `thread_hold_the_lock` function and the `thread_donate_priority` function, and encapsulate them in the lock_acqcuire function.When a thread is donated by multiple threads, change the current priority to the highest priority among the donation threads to ensure the rationality of this thread.

- **Step 4** :When a thread is prioritized, if the thread is in a donated state, the original priority of the thread is set, and then if the set priority is greater than the current priority, the current priority is changed, otherwise When the donation status is cancelled, the original priority is restored.So we need to implement a `thread_update _priority` to implement this change process.

- **Step 5** :The remaining donated priority and current priority should be considered when releasing the lock to a lock priority.

- **Step 6** :The wait queue for the semaphore is implemented as a priority queue. Implement the condition's waiters queue as a priority queue. And if the priority is changed when the lock is released, preemption can occur. In the `cond_signal` function, use the `list_sort` to sort the condition queue, Then implement the wait queue of the `semaphore` as a priority queue.

# Synchronization 

  - If we want to consider synchronization issues in the Priority Scheduler task, we must consider how to solve a  potential race in thread_set_priority. In the algorithm, I implement a tatic method to turn off interrupts in `thread_set_priority`, the strategy can be done because we have to read/write to the current thread's priority that is updated every four tricks in the interrupt handler. Because of that reason, we can not use locks to solve it since the interrupt.

# Rationale

- This algorithm is rational and rational because it follows a certain amount of algorithmic logic, namely two aspects, acquiring locks and giving priority to donations. First, when the thread that owns the lock has a higher priority than its own priority, it raises its own priority, and when the thread releases the lock, it restores the initial priority. Second, when a thread is donated by multiple thread threads, the highest priority is selected in the thread that is donated. In addition to this, you need to implement a semaphore priority queue and a waiters of condition priority queue.


# Task3: Priority Scheduler

# Data structures and functions

## thread.h

`int nice;`
- nice priority of threads.

`fixed_t recent_cpu;`
- recent_cpu measures the amount of CPU time a thread has received "recently."

`Struct thread{tid_t tid; enum thread_status status; char name[16]; unin8_t *stack; int priority; struct list_elem allelem; struct list_elem elem; uint32_t *pagedir; unsigned magic; int64_t record_blocked;}`
- Using the struct thread including int nice, fixed_t recent_cpu in the algorithm.

## thread.c

`fixed_t load_avg;`
- estimates the average number of threads ready to run over the past minute.

# Algorithm 

The algorithm is still clear and easy to understand in the way of implementation. 

- **step 1** : Implement floating point arithmetic logic in `fixed_point.h`.

- **step 2** : Calculate the priority of the update thread in the `timer_interrupt` for a period of time. Here, the system   `load_avg` and the `recent_cpu` of all threads are updated every `TIMER_FREQ` time. 

- **step 3** : Every 4 `timer_ticks` is updated once. Thread priority.

- **step 4** : Add one of the `recent_cpu` of each timer_tick running thread.

- **step 5** : Although it is said that 64 priority queue scheduling is maintained, the essence is priority scheduling. We reserve the priority scheduling code written before, and remove the priority donation.

# Synchronization  

- In order to ensure the synchronization of the algorithm, one is turn off `interrupts`, the other is to store `int nice` and `fixed_t recent_cpu` as variables in the `struct thread`, to ensure that all threads share these two properties to ensure thread synchronization.

# Rationale

* Achieve multi-level feedback scheduling, its role is to reduce the average response time of the system, therefore, it is more reasonable to first solve the operating system floating-point arithmetic problem, and then, according to the above mentioned algorithm, update in a fixed period of time The priority of the thread completes the multi-level feedback scheduling.


# Design Document Additional Questions

- (This question uses the MLFQS scheduler.) Suppose threads A, B, and C have nice
values 0, 1, and 2. Each has a recent_cpu value of 0. Fill in the table below showing
the scheduling decision and the recent_cpu and priority values for each thread after
each given number of timer ticks. We can use R(A) and P(A) to denote the recent_cpu
and priority values of thread A, for brevity.

```
timer ticks       R(A)    R(B)    R(C)    P(A)    P(B)    P(C)     thread to run 
    0              0       0       0       63      61      59            A
    4              4       0       0       62      61      59            A
    8              8       0       0       61      61      59            B
    12             8       4       0       61      60      59            A
    16            12       4       0       60      60      59            B
    20            12       8       0       60      59      59            A
    24            16       8       0       59      59      59            C
    28            16       8       4       59      59      58            B
    32            16      12       4       59      58      58            A
    36            20      12       4       58      58      58            C
```

* Did any ambiguities in the scheduler specification make values in the table (in the
previous question) uncertain? If so, what rule did you use to resolve them?

```
Yes, because if the threads have the same priority, it's unconspicuous to choose which threads to execute.
So, I choose two strategies to solve them:

1.  If one of the threads in the ready list have the highest priority, it will execute first obviously;
2.  If the threads in ready list have the same priority, then choose the thread has the most least run-time recently.
```
