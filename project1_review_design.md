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

   When a thread is sleeps, if we want it doesn’t occupy the resource of cpu, we need to use a counter called `record_blocked` to record how much time the thread has slept, so I add the function `check_thread_status` to detect each thread’s status(blocked or not) and record_blocked’s time, if the conditions do not satisfy sleep time , it will call function `thread_unblock` to arouse the thread and put it into ready queue to keep on executing.
  
# Synchronization 

   First of all, in order to ensure the synchronization of the thread, I will use the variable `int64_t tricks_blocked` that record thread blocked time and add it in the `struct thread` data structure to ensure that all threads have this property. Secondly, I add the new function check_thread_status to the `thread_foreach` function to ensure that the judgment of the thread state and sleep time is performed for each thread. Third, if this function does not match the condition of the blocked time, it will be woken up and put back into the ready queue, which is also a guarantee of synchronization.
   
# Rationale
  
   When timer_sleep is used, the thread is directly blocked, and then a member `int64_t record_blocked` is added to the thread structure to record how much time the thread has been sleeping. Then, the operating system's own clock interrupt (each tick is executed once) is added to the thread state. Detect, subtract ticks_blocked by 1 for each test, and wake up this thread if it is reduced to zero.
   
   
 
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

# list.c
`struct list_elem{struct list_elem *prev; struct list_elem *next};`
- Structure for list element;

`struct list {struct list_elem head; struct list_elem tail};`
- Structure for list;

`list_insert_ordered (&all_list, &t->allelem, (list_less_func *) &thread_cmp_priority, NULL);`
- Inserts ELEM in the proper position in LIST, which must be sorted according to LESS given auxiliary data AUX













  
