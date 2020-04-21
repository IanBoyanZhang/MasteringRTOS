# MasteringRTOS
Running FreeRTOS on Arduino, STM32F4x and Cortex M based MCUs


# Notes:

Concepts of task and task control block(TCB)

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



