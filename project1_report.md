# Final report for project1

## 1. Design Changes

## Task 1 Efficient Alarm Clock
- 1. **Change the variable name**:  In Task1, I renamed the variable name in the previous design document to make the variable life more in line with its own meaning. For example, I add a member to the thread structure in the code to add a member `int64_t waiting_ticks` to the thread structure to record how long the thread has been sleeped, and then use the operating system's own clock interrupt (each tick will be executed once) to join the pair. 
- 2. **Thread state detection** ： each test ticks_blocked minus 1, if reduced to 0, wake up this thread to record how long this thread has been sleep, and then use the operating system's own clock interrupt (each tick will be executed once) join the thread State detection, `waiting_ticks` is decremented by 1 for each test, and if it is reduced to 0, the thread is woken up.
   Specific Changing implementation：
    ```c
    /*Timer interrupt handler, check block status every interrupt*/
     static void timer_interrupt (struct intr_frame *args UNUSED){
        ticks++;
        thread_tick();
        thread_foreach(check_block_status,NULL);
        advance_scheduler();
     }
    ```
    ```c
    void check_block_status(void *temp UNUSED, struct thread *ele){
      if (ele->waiting_ticks > 0){
         if (ele->status == THREAD_BLOCKED){
           ele->waiting_ticks --;
           if (ele->waiting_ticks == 0){  // awake the thread .
              enum intr_level old_level;
              ASSERT (is_thread(ele));
              old_level = intr_disable();
              ASSERT (ele->status == THREAD_BLOCKED);
              list_push_back(&ready_list, &ele->elem);
              ele->status = THREAD_READY;
              intr_set_level(old_level);
           }
         }
      } 
    } 
    ```
## Task2: Priority Scheduler
- 1. **donating the priority**: When implementing the thread priority donation, I no longer implement the thread_donate_priority in the original design review separately, but write the method of comparing the current thread with the newly added thread and the method of thread donation. This better represents the reason for the thread donation.
    ```c
    /*The current thread is compared with the waiting thread to achieve priority donation*/
    if(default_n!=0){
           do
            {   
                int lock_priority;
                int initial_status = 1;
                init_lock_status();
                lock_obj->max_priority = curr_t->priority;
                enum intr_level old_level = intr_disable ();
                int max_priority = lock_obj->holder->base_priority;
                   if (!check_empty(lock_obj))
                      {
                      sort_lock_cmp_priority(lock_obj);
                      lock_priority = list_entry (list_front (&lock_obj->holder->locks),
                      struct lock, elem)->max_priority;
                      if (lock_priority > max_priority && default_n == 1 ){ 
                      // default_n helps to debug.
                          max_priority = lock_priority;
                          }
                            else{
                          whether_change = false;
                         }
                           }
                       lock_obj->holder->priority = max_priority;
                       intr_set_level (old_level); 
                if (set_value != 1 && lock_obj->holder->status == THREAD_READY && default_n == 1)
                    {
                   remove_lock_t_ele(lock_obj);
                   thread_insert_order(lock_obj->holder);
                   }
                   intr_set_level (old_level);
                   lock_obj = lock_obj->holder->lock_waiting;
                    initial_status = 0;
         }while (lock_obj && curr_t->priority > lock_obj->max_priority && set_value == 0);
    ```
## Task3: Priority Scheduler
- 1. **Use floating point number calculation update**: When implementing code, use floating point number calculation to realize multi-level feedback scheduling, update priority, recent_cpu, load_avg, the specific implementation are as follows：
  ```c
  /*update recent_cpu*/
  void update_cpu_by_FP(struct thread *curr_t){
  curr_t->recent_cpu = FP_ADD_MIX(curr_t->recent_cpu,1);
  }
  ```
  ```c
  /*update load_avg*/
  void update_load_avg(){
  load_avg = FP_ADD (FP_DIV_MIX (FP_MULT_MIX (load_avg, 59), 60),
  FP_DIV_MIX (FP_CONST (ready_t), 60));
  }
  ```
  
  ```c
  /*update priority*/  
  void update_priority(){
     t->priority = FP_INT_PART (FP_SUB_MIX (FP_SUB (FP_CONST (PRI_MAX),
     FP_DIV_MIX (t->recent_cpu, 4)), 2 *   t->nice));
  if (t->priority < PRI_MIN){
      t->priority = PRI_MIN;
  }else{
      t->priority = t->priority;
  }
 
   if (t->priority > PRI_MAX){
      t->priority = PRI_MAX;
    }else{
      t->priority = t->priority;
   } 
  }
  ```
  ## Task4 Using GDB to debug
   - In this part, here are my execution to find the bug using GDB
   - First, I make check the pintos-GDB's src, and the result shows that `mlfqs-load-60` fail with bug that some load average values were missing or differed from those expected by more than 3.5.
   - Second, I realize that there may have some fatal floating calculating problem in function `update_recent_cpu_all()` or `update_load_average()`.
   - Third, I set two breakpoints in these two functions, and in the functions , I print values of `recent_cpu` and `load_average` respectively, and comparing these results with correct answser in test files.
   - Fourth, fortunately, some results in recent_cpu does not correct, that is definitely the problem of `FP_ADD_MIX` function, mutiply first and divide second will cause the bug.
   - Fifth, I modify this bug by replacing the function as I implement in my code, that 
    ```c
    t->recent_cpu = FP_ADD_MIX (FP_MULT (FP_DIV (FP_MULT_MIX (load_avg, 2), 
    FP_ADD_MIX (FP_MULT_MIX  (load_avg, 2), 1)), t->recent_cpu), t->nice);
               }
    ```
    
  ## 2. Reflection
- Yes, I am. In implementing the pintos project1 task, I realized the importance of carefully reading the documentation and materials and designing a design review before actually working on the code. The purpose of this is to let us clarify the logic of the task and the way it is implemented, to help the implementation and reconstruction of the code.

- Secondly, this pintos project1 encountered a lot of obstacles. The biggest obstacle is the various data structures and deep calls that are not familiar with the source code. So in the next project, the improvement is to familiarize yourself with the materials and documentation in advance, and work harder on the use of source code functions.
  
