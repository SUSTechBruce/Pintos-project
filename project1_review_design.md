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







  