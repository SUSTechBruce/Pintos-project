# Final report for project1

## 1. Design Changes

## Task 1 Efficient Alarm Clock
1. Change the variable name:  In Task1, I renamed the variable name in the previous design document to make the variable life more in line with its own meaning. For example, I add a member to the thread structure in the code to add a member `int64_t ticks_blocked` to the thread structure to record how long the thread has been sleeped, and then use the operating system's own clock interrupt (each tick will be executed once) to join the pair. 
2. Thread state detection, each test ticks_blocked minus 1, if reduced to 0, wake up this thread to record how long this thread has been sleep, and then use the operating system's own clock interrupt (each tick will be executed once) join the thread State detection, ticks_blocked is decremented by 1 for each test, and if it is reduced to 0, the thread is woken up.
