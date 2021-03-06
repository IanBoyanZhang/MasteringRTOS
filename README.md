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

`vTaskStartScheduler()/tasks.c` -> `xPortStartScheduler()/port.c` 


#### Context Switching

If the scheduler is priority based pre-emptive scheduler, the n for every RTOS tick interrupt, the scheduler will compare the priority of the running task with the priority of ready tasks list. If there is any ready task whose priority is higher than the running task then context switch will occur.

On FreeRTOS you can also trigger context switch manually using `taskYIELD()` macro

Context switch also happens immediately whenever new task unblocks and if its priority is higher than the currently running task. 

##### Task State

When a task executes on the processor it utilizes

- Processor core registers

- If a Task wants to do any push and pop operations (during function call) then it uses its own dedicated stack memory.

Contents of the processor core registers + Stack contents ==> state of the task

[ARM Cortex Core Registers](https://developer.arm.com/docs/dui0553/latest/the-cortex-m4-processor/programmers-model/core-registers)

##### Stacks

There are mainly 2 different Stack Memories during run time of FreeRTOS based application

Task's private stack (Process Stack) vs Kernel Stack (Main Stack)

SP (PSP) => PUSH and POP to Process Stack area is tracked by PSP register of ARM (When Task executes it does push/pop here)

SP (MSP) => PUSH and POP to this stack area is tracked by MSP Register of ARM (When ISR executes it does push/pop here)

##### Task Swiching out and in procedures

Before task is switched out, following things have be taken care of:

- Processor core registers R0, R1, R2, R3, R12, LR, PC, xPSR(stack frame) are saved on to the task's private stack automatically by the processor SysTick interrupt entry sequence.

- If context switch is required then SysTick timer will pend the PendSV Exception and PendSV handler runs

- Processor core registers (R4-R11, R14) have to be saved manually on the task's private stack memory (Saving the context)

- Save the new top of stack value (PSP) into first member of the TCB

- Select the next potential Task to execute on the CPU. Taken care by vTaskSwitchContext() implemented in tasks.c


## Source Code reading and Concepts

Common kernel code -> calling `port.c`? 


Some bullet points regarding preemptive vs non-preemptive scheduling

[CPU Scheduling](http://www.personal.kent.edu/~rmuhamma/OpSystems/Myos/cpuScheduling.htm)


## Concurrency

### How to achieve signaling

- Events (or Event Flags)

- Semaphores (Counting and binary)

- Queues and Message Queues 

- Pipes

- Mailboxes

- Signals (Unix like signals)

- Mutex

All these software subsystems support signaling hence can be used in synchronization purpose


## Interrupt Safe APIs and Task Yielding

FreeRTOS APIs which don't end with the word "FromISR" are called as interrupt unsafe APIs

**These APIs should not be called from an ISR **

e.g.

- xTaskCreate()
- xQueueSend()
- xQueueReceive()

etc

### Why separate interrupt Safe APIs?

handler mode vs thread mode

[Are exceptions stacked by the cortex m hardware in thread mode or handler mode](https://stackoverflow.com/questions/38269360/are-exceptions-stacked-by-the-cortex-m-hardware-in-thread-mode-or-handler-mode)

[Why do we need two priviledged modes can't one do the thing in cortex m3?](https://community.arm.com/developer/ip-products/processors/f/cortex-a-forum/7420/why-do-we-need-two-priviledged-modes-cant-one-do-the-thing-in-cortex-m3)

### Interrupt Safe APIs: Conclusion

Whenever you want to use FreeRTOS api from an ISR its ISR version must be used, which ends with the word "FromISR"

This is for the reason, being in the interrupt Context (i.e. being in the middle of servicing the ISR) you can not return to Task Context (i.e. making a task to run by pre-empting the ISR)

In ARM cortex M based processors usage exception will be raised by the processor if you return to the task context by preempting ISR


**The task yielding in FreeRTOS on ARM cortex M processor is nothing but just pending the PendSV exception **

**Triggering of task yielding is always architecture specific. In ARM cortex M processor architecture task yielding (i.e. context switch to potential task is triggered by pending the PendSV Exception**


**Is it allowed to use semaphore or mutex inside the ISR?**

Yes ! you can call the semaphore and mutex related APIs from the ISR. But the only condition is you cannot block on the semaphore or mutex. Thats the reason, in FreeRTOS you don't find blocking time parameter in the semaphore take and give apis. Please explore xSemaphoreTakeFromISR(), xSemaphoreGiveFromISR()

**Can I use PendSV in AVR and MSP430 architecture to implement task yielding ?**

No!

PendSV is an exception which is ARM Cortex M processor specific, so you can not trigger PendSV in MSP or AVR architecture.

## Task States

Running states vs Not-Running State

Not running state:

create task -> suspended -> Running
            -> ready
            -> blocked
            
What is blocking of a Task?

A task which is temporarily or permanently chosen not to run on CPU


Blocking is common technique for synchronization

**can we block the task indefinitely in FreeRTOS ?**

if you use vTaskDelay(portMAX_DELAY) the task will be blocked indefinitely . So, the parameter portMAX_DELAY causes the task to block indefinitely

**Can a suspended state task come out of suspended state by any external event or due to timeout ?**

No!

The only way for a task to come out of the suspended state is , some other task should call vTaskResume() with the handle of the suspended task

**If interrupt handler unblocks any higher priority task, then when ISR returns, will execution resume at the higher priority task which is unblocked or execution will resume at the task which was interrupted?**

Its a responsibility of the ISR to call for task yield if higher priority task unblocks


**FreeRTOS Hook Functions**

- Idle task hook function
- RTOS tick hook function
- Dynamic memory allocation failed hook function (Malloc Failed hook function)
- Stack overflow hook function



WFI Wait For Interrupt

It is a 16 bit thumb instruction

## Scheduling policies

1. Preemptive Scheduling

2. Priority based preemptive scheduling (mostly commonly used in RTOS)

3. Co-operative scheduling


Preemption is the act of temporarily interrupting an already executing task with the intention of removing it from the runing state without its co-operation


Tasks yielding, pendSV exception

Co-operative scheduling

Co-operative among different tasks

(Task decides when to leave the CPU?, taskYield(), equaly priority, not for RTOS)

## 

It is always a good strategy to check why certain interrupt is happening, triggered.

## Timer Queue

## Solving race condition between task and ISR

taskENTER_CRITICAL()

taskEXIT_CRITICAL()

Related ARM hardware, base register

In RTOS, we should avoid using dynamic allocation as much as possible


## Concurrency and Synchronization

If a task can acquire the key(semaphore), then only it can carry out the intended operation

Task-waiting list

If a task fails to get the key, then that task will be blocked and kernel will put that task into task waiting list.

That means tasks state will be changed from "Running" to "Blocked"!

A single semaphore can be acquired, finite number of times by the tasks depending upon, how you first initialize the semaphore.

If value=4 then 4 semaphore keys or tokens are currently available!

If a task tries to acquire the key for the 5th time, then it wil be blocked.

This value decrease when the key is acquired and it increases when the semaphore token or key is given back.

- Binary Semaphore

  1. Synchronization between tasks or synchronization between interrupts and tasks
  
  2. Mutual Exclusion Binary semaphore can also be used for Mutual exclusion to guard the critical section
  
  
- Counting Semaphore

  Resource management, count value indicates the number of resources available. To obtain control of a resource task must first obtain a semaphore by decrementing the semaphores count value.
  
  When the count value reaches zero there are no free resources. When a task finishes with the resource it 'gives' the semaphore back thus incrementing semaphores count value

## Events Latching

1. When the interrupts/events happen relatively slow, the binary semaphore can latch at most only one event

2. If multiple interrupts/events trigger back to back, then the binary semaphore will not able to latch all the events. So some events will be lost

3. How to solve the above issue? Welcome to the world of "counting semaphore"

### Counting semaphore

Counting semaphore can be used to latch multiple events

P/V acquire/release


```
#define UART_DR   *((unsigned long*)(0x40000000))

/* This is a common function which write to UART DR */
int UART_Write(uint32_t len, uint8_t *buffer)
{
  for (uint32_t i = 0; i < len; i++)
  {
    sema_take_key(bin_sema);
    /* if Data register is empty write it*/
    while(!is_DR_empty()) {}
    // Critical Section
    UART_DR = buffer[i];
  }
}

```
Actual example, use
```
xSemaphoreTake(xBinarySemaphore, portVMax_DELAY);
```
for spinlock, use

```
xSemaphoreTake(xBinarySemaphore, 0)
```
It will keep polling for the semaphore until it is available

**Do not use binary semaphore for mutual exclusion**

A higher priority task dependant on the lower priority task (Priority inversion)

## Advantage of Mutex over Binary Semaphore

- Priority Inheritance

mutexes and binary semaphores are very similar the only major difference is mutexes automatically provide a basic 'priority inheritance' mechanism

Priority inheritance is a technique by which mutex minimizes the negative effects of inversion. Mutex can not fix the priority inversion problem completely but it surely lessen its impact

Lower priority task acquires the priority that means inherits the priority of highest priority task executes and releases the key, then it comes back to medium priority task

Most of the RTOS including FreeRTOS mutex implementation implements priority inheritance feature.

Since mutex has all these features to avoid priority inversion, the memory consumed by mutex service may be higher than the binary semaphore

## Ways to protect the critical section

1. Binary semaphore

2. Mutex

3. Crude way (disabling interrupts of the system, either globally, or up to a specific interrupt priority level)

Disable/enable the interrupt
```
taskENTER_CRITICAL()
taskEXIT_CRITICAL()
```

Architecture dependent


## References

[FreeRTOS source code reading notes (CHN)](https://my.oschina.net/u/3699634/blog/1544909)

[Using the FreeRTOS Real Time Kernel](https://www.freertos.org/Documentation/Using-the-FreeRTOS-Real-Time-Kernel-A-Practical-Guide-LPC17xx-Edition-Document-Outline.pdf)

[Memory Management](https://www.freertos.org/a00111.html)


## General OS Tutorial 

This is really good course that covers necessary part of eduction!!

[CS194 Sp14 Berkeley CS194-24:	Advanced Operating Systems Structures and Implementation](https://people.eecs.berkeley.edu/~kubitron/courses/cs194-24-S14/index_lectures.html)

[Linux Kernel Understanding Queudet](https://people.eecs.berkeley.edu/~kubitron/courses/cs194-24-S14/hand-outs/linuxKernelUnderstandingQueudet.pdf)

What is 

[Berkely cs61c Spring 20 Great Ideas in Computer Architecture (Machine Structures)
](https://cs61c.org/su20/)

Paging and Virtual memory

Virtual memory, LRU cache, TLB (Translation Lookaside Table) and page table
[Virtual Memory Overview](https://inst.eecs.berkeley.edu/~cs61c/sp14/disc/12/dis12Sol.pdf)

[Page Replacement algorithm](https://en.wikipedia.org/wiki/Page_replacement_algorithm)

This is a good study guide
[Operating System study guide](http://faculty.salina.k-state.edu/tim/ossg/index.html)

Really good for learning Linux scheduling
[CSCI 560 Project Main](https://www.cs.montana.edu/~chandrima.sarkar/AdvancedOS/CSCI560_Proj_main/)

Maybe, this is an updated version of above material?
[AdvancedOS CSCI560](https://www.cs.montana.edu/~chandrima.sarkar/AdvancedOS/main/index.html)


**I think the question was about Prirority Scheduling**

[Wiki Scheduling](https://en.wikipedia.org/wiki/Scheduling_(computing))

## War stories

[How did NASA remotely fix the code on the M](https://space.stackexchange.com/questions/9178/how-did-nasa-remotely-fix-the-code-on-the-mars-pathfinder)

[Reports Mars Path Finder](http://www.cse.chalmers.se/~risat/Report_MarsPathFinder.pdf)

[Mike Jones, What really happend to Mars Path Finder](http://www.cs.cornell.edu/courses/cs614/1999sp/papers/pathfinder.html)

