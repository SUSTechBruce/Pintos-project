# Final report for project1

## 1. Design Changes

## Task 1 Efficient Alarm Clock
- 1. **Change the variable name**:  In Task1, I renamed the variable name in the previous design document to make the variable life more in line with its own meaning. For example, I add a member to the thread structure in the code to add a member `int64_t waiting_ticks` to the thread structure to record how long the thread has been sleeped, and then use the operating system's own clock interrupt (each tick will be executed once) to join the pair. 
- 2. **Thread state detection** ： each test ticks_blocked minus 1, if reduced to 0, wake up this thread to record how long this thread has been sleep, and then use the operating system's own clock interrupt (each tick will be executed once) join the thread State detection, ticks_blocked is decremented by 1 for each test, and if it is reduced to 0, the thread is woken up.
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
