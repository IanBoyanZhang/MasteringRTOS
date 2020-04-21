# MasteringRTOS
Running FreeRTOS on Arduino, STM32F4x and Cortex M based MCUs

# Notes:

Concepts of task and task control block(TCB)

[FreeRTOS Documentation](https://www.freertos.org/Documentation/RTOS_book.html)

[FreeRTOS introduction](https://www.aosabook.org/en/freertos.html)

vTaskStartScheduler() has to be called to make tasks run on the cpu

#### Let’s say there are two Tasks in the task list of different priority , you know that higher priority task always runs on the CPU in  a priority based preemptive scheduling, So, whats the way to give the CPU to lower priority task from higher priority task ?

Blocking, suspending and task yielding of higher priority task can make lower priority run on the CPU

#### You can create a task by static allocation method in FreeRTOS

#### When you create a task statically, the stack memory of the task will be allocated in global space of the RAM

Suppose there is a non zero initialized static variable is declared in the task function, where exactly memory for that static variable is allocated. ?

```
void task_function(void *p)

{

/*this is the task function */

static int i=10;

}
```
In the global area of the RAM, also called as .DATA section

Suppose there is a non static local variable declared in the task function, where exactly the memory for the non static variable is allocated during the execution of task function  ?

```
void task_function(void *p)

{

 /*this is task function */

int i ; /* non static variable */



}
```

In the task's stack space


STM32 

RCC (Reset and Control Clock), clock tree

Key files:

`/config/FreeRTOSConfig.h`

[Customization: https://www.freertos.org/a00110.html](https://www.freertos.org/a00110.html)


Main Stack Pointer (MSP) and Process Stack Pointer (PSP)

[Main Stack Pointers MSP vs Process Stack Pointers PSP](https://electronics.stackexchange.com/questions/403967/main-stack-pointermsp-vs-process-stack-pointerpsp)

How it is related to Context Switching and hardware resources 

[Context switching on the Cortex M3](https://blog.stratifylabs.co/device/2013-10-09-Context-Switching-on-the-Cortex-M3/)

[System Timer (SysTick)](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dai0179b/ar01s02s08.html)

[Significance of ARM SysTick timer vs other Timers](https://electronics.stackexchange.com/questions/350577/significance-of-arm-systick-timer-vs-other-timers)

[ARM developer community, using the static keyword in c](https://community.arm.com/developer/ip-products/system/b/embedded-blog/posts/using-the-static-keyword-in-c)

[Where are static variables stored in C and C++?](https://stackoverflow.com/questions/93039/where-are-static-variables-stored-in-c-and-c)

[.bss](https://en.wikipedia.org/wiki/.bss)

![Program Memory Layout](https://upload.wikimedia.org/wikipedia/commons/thumb/5/50/Program_memory_layout.pdf/page1-299px-Program_memory_layout.pdf.jpg)


#### Debugging

[Segger SemiHosting](https://wiki.segger.com/Semihosting)

[How to use semihosting with STM32](https://shawnhymel.com/1840/how-to-use-semihosting-with-stm32/)


#### Kernel

##### Multi-task handler Techniques 

[Cooperative Multitasking](https://en.wikipedia.org/wiki/Cooperative_multitasking)

[What is the difference between cooperative multitasking and preemptive multitasking](https://stackoverflow.com/questions/55703365/what-is-the-difference-between-cooperative-multitasking-and-preemptive-multitask)

[taskYield, FreeRTOS Kernel Control](https://www.freertos.org/a00020.html)

[RTOS idle task](https://www.freertos.org/RTOS-idle-task.html)

The Idle task is created automatically when the RTOS scheduler is started to ensure there is always at least one task that is able to run.

It is created at the lowest priority to ensure it does not use any CPU time if there are higher priority application tasks in the ready state

[Tasks Idletask](https://www.hackster.io/Niket/tasks-idletasks-freertos-tutorial-7-42126c)

Some facts about idle task

- It is a lowest priority task which is automatically created when the scheduler is started

- The idle task is responsible for freeing memory allocated by the RTOS to tasks that have been deleted

- When there are no task running, idle task will always run on the CPU

- You can give an application hook function in the idle task to send the CPU to lower mode when there are no useful tasks are executing

##### Idle Task hook function

- Idle task hook function implements a callback from idle task to your application

- You have to enable the idle task hook function feature by setting this config item `configUSE_TICK_HOOK` to 1 within FreeRTOSConfig.h

- Then implement the below function in your application `void vApplicationIdleHook(void)`

- Whenever idle task is allowed to run, your hook function will get called, where you can do some useful stuffs like sending the MCU to lower mode to save power


##### Timer Services Task (Timer_svc)

- This is also called as timer daemon task

- The timer daemon task deals with "Software timers"

- This task is created automatically when the scheduler is started and if `configUSE_TIMER = 1` in FreeRTOSConfig.h

- The RTOS uses this daemon to manage FreeRTOS software timers and nothing else

- If you don't use software timers in your FreeRTOS application then you need to use this Timer daemon task. For that just make `configUSE_TIMERS = 0` in FreeRTOSConfig.h

- All software timer callback functions execute in the context of the time daemon task 


##### Scheduler

- Implements task switching in and task switching out according to the scheduling policy selected

- Scheduler is the reason why multiple tasks run on your system efficiently

- The basic job of the scheduler is to determine which is the next potential task to run on the CPU

- Scheduler has the ability to preempt a running task if you configure so

##### Scheduling Policies (Scheduler Types)

- Simple Pre-emptive scheduling (Round robin)

- Priority based pre-emptive scheduling

- Co-operative scheduling

The scheduling policy is the algorithm used by the scheduler to decide which task to execute at any point in time

FreeRTOS or most of the real time OS most likely woud be using `Priority Based` `Pre-emptive Scheduling` by default


##### FreeRTOS scheduler implementation

In FreeRTOS the scheduler code is actually combination of FreeRTOS Generic code + Architecture specfic codes

FreeRTOS Generic Code in `tasks.c`

Arch. Specific codes for `port.c`

All architecture specific codes and configurations are implemented in `port.c` and `portmacro.h`

If you are using ARM Cortex Mx processor then you should be able to locate the below interrupt handlers in `port.c` which are part of the scheduler implementation of FreeRTOS

`vPortSVHandler()` Used to launch the very first task. Triggered by SVC instruction

`xPortPendSVHandler()` Used to achieve the context switchin between tasks. Triggered by pending the PendSV system exception of ARM

`xPortSysTickHandler()` This implements the RTOS Tick management. Triggered periodically by Systick timer of ARM cortex Mx processor

`vTaskStartScheduler()` implemented in `tasks.c` of FreeRTOS kernel and used to start the RTOS scheduler

Remember that after calling this function only the scheduler code is initialized and all the Arch. Specific interrupts will activated

- This function also creates the idle and Timer daemon task

- This function calls `xPortStartScheduler()` to do the Arch. specific initializations

##### FreeRTOS Kernel interrupts

When FreeRTOS runs on ARM cortex Mx processor based MCU, below interrupts are used to implement the Scheduling of Tasks

- 1. SVCInterrupt (SVC handler will be used to launch the very first Task)

- 2. PendSVInterrupt (PendSV handler is used to carry out context switching between tasks)

- 3. SysTick interrupt (SysTick handler implements the RTOS Tick Management)

If SysTick interrupt is used for some other purpose in your application, then you may use any other available timer peripheral

All interrupts are configured at the lowest interrupt priority possible


#### RTOS Tick (The heart beat)

##### Why is it needed?

- To keep track of time elapsed

- There is a global variable called `xTickCount` and it is incremented by one whenever tick interrupt occurs

- RTOS ticking is implemented using SysTick Timer of the ARM Cortex Mx

- Tick interrupt happens at the rate of `configTICK_RATE_HZ` configured in the FreeRTOSConfig.h

Used for ContextSwitching to the next potential task

- The tick ISR runs

- All the ready state tasks are scanned

- Determined which is the next potential task to run

- If found, triggers the context switching by pending the PendSV interrupt

- The PendSV handler takes care of switching out of old task and switching in of a new task


Pending an interrupt also means "Activating" that interrupt

PendSV is a System exception which can be triggered by enabling its pending bit in the ARM cortex Mx processor
