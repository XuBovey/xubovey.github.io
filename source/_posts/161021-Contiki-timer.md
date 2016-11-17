---
title: Contiki定时器
toc: true
date: 2016-10-21 10:25:59
categories:
- Contiki
- timer
tags:
- Contiki
---

了解contiki进程调度之前有必要先了解下它的定时器使用。因为作为轮询方式执行的函数，除了外部中断，其他事件中断可触发一些任务外，只有了解了才能明白系统是如何进行任务调度的。

Contiki有5种定时器：  
* clock : 用于处理系统时间，是下面集中timer的基础；而这个是需要platform提供的。
* timer : 以clock的tick为单位进行计时，精度最高；
* stimer: second timers，秒计时器，比timer计时时间长；timer和stimer库函数可以在中断中调用。
* etimer: event timers，事件定时器，定期进行进程的事件调度；被用来上电时等待系统稳定，或进入低功耗模式；
* ctimer: callback timers，回调定时器，定期执行进程回调函数，和etimer一样可被用于等待系统稳定和进入低功耗模式；但是更实用的是用在协议栈之类的非进程函数中。
* rtimer: real-time timers，实时时钟定时器，用于实时任务的调度，rtimer优先级高于其它任何进程,以保证实时任务被及时调用。

本文参考Contiki的[Timers](https://github.com/contiki-os/contiki/wiki/Timers)

<!--more-->
# 1. clock  
clock是需要不同平台(platform)中提供的，Clock Module提供如下API:
``` c
clock_time_t clock_time(); // Get the system time. 
unsigned long clock_seconds(); // Get the system time in seconds. 
void clock_delay(unsigned int delay); // Delay the CPU. 
void clock_wait(int delay); // Delay the CPU for a number of clock ticks. 
void clock_init(void); // Initialize the clock module. 
CLOCK_SECOND; // The number of ticks per second.
```
clock_time返回ticks计数值，而每秒钟有多少个ticks，则由宏定义CLOCK_SECOND决定，clock_time_t多是unsigned，具体因platform而异。  
clock_seconds是秒计数，返回值为unsigned long型数据（MSP430可计时136年），不同处理器字节数可能不同。  
delay用于us延时，和wait用于ticks延时（利用clock_time延时）。这两个延时均独占CPU。

# 2. timer
timer调用clock的clock_time函数完成各种功能，每一个需要用到timer的地方需要定义一个struct timer数据结构，所有的timer函数均是对这个数据结构的操作，API函数如下：
``` c
void timer_set(struct timer *t, clock_time_t interval); // Start the timer.
void timer_reset(struct timer *t); // Restart the timer from the previous expiration time.
void timer_restart(struct timer *t); // Restart the timer from current time. 
int timer_expired(struct timer *t); // Check if the timer has expired. 
clock_time_t timer_remaining(struct timer *t); // Get the time until the timer expires.
```
例如串口接收中断函数，判断等待超时：
``` c
 #include "sys/timer.h"
 
 static struct timer rxtimer;
 
 void init(void) {
   timer_set(&rxtimer, CLOCK_SECOND / 2);
 }
 
 interrupt(UART1RX_VECTOR)
 uart1_rx_interrupt(void)
 {
   if(timer_expired(&rxtimer)) {
     /* Timeout */
     /* ... */
   }
   timer_restart(&rxtimer);
   /* ... */
 }
```
使用前先调用init函数，当前时间（clock_time函数获取）保存在timer->start中，将设定的时间间隔保存在timer->interval中，之后每次执行timer_expired函数时用当前时间减去start，然后和interval比较，返回ture/false。  
timer数据结构
``` c
struct timer {
  clock_time_t start; //= unsigned short start;
  clock_time_t interval; //= unsigned short interval; 
};
```
# 3. stimer
用法和timer一样，只是时间精度和最大时间长度不一样，API函数如下：
``` c
void stimer_set(struct stimer *t, unsigned long interval); // Start the timer.
void stimer_reset(struct stimer *t); // Restart the stimer from the previous expiration time.
void stimer_restart(struct stimer *t); // Restart the stimer from current time. 
int stimer_expired(struct stimer *t); // Check if the stimer has expired. 
unsigned long stimer_remaining(struct stimer *t); // Get the time until the timer expires.
```

# 4. 进程事件标识符
``` c
#define PROCESS_EVENT_NONE 128
#define PROCESS_EVENT_INIT 129
#define PROCESS_EVENT_POLL 130
#define PROCESS_EVENT_EXIT 131
#define PROCESS_EVENT_CONTINUE 133
#define PROCESS_EVENT_MSG 134
#define PROCESS_EVENT_EXITED 135
#define PROCESS_EVENT_TIMER 136
```

# 5. etimer
参考[Contiki 调度内核不完全介绍](https://my.oschina.net/lgl88911/blog/139463)
 在进程事件标识符中的PROCESS_EVENT_TIMER就是由etimer所触发的。etimer同样是通过clock_time函数进行时间的设定，比较的。不同的是etiemr必须绑定到一个process上，etimer到期后就触发PROCESS_EVENT_TIMER到对应的process上。etimer基于timer和etimer_process，而etiemr_process只响应两种事件：PROCESS_EVENT_EXITED和PROCESS_EVENT_POLL。  
 PROCESS_EVENT_EXITED告诉etimer_process有进程执行了process_exit，需要从etimer链表中删除相应process的节点；PROCESS_EVENT_POLL事件由etimer_request_poll()驱动，告诉etimer_process，让其检查etimer链表上是否有到期事件发生。  
 > 当etimer检查到timer到期，会发生PROCESS_EVENT_TIMER到绑定的process，如果发送失败，etimer_process会再自己call etimer_request_poll()来触发一次检查。如果发生成功了etimer就会从timer list中删除，因此一个etimer启动后到期一次就失效了。

 其API函数与timer类似：
## 5.1 etimer API函数
 ``` c
void etimer_set(struct etimer *t, clock_time_t interval); // Start the timer.
void etimer_reset(struct etimer *t); // Restart the timer from the previous expiration time.
void etimer_reset_with_new_interval(struct etimer *et, clock_time_t interval);// Reset an event timer with a new interval.
void etimer_restart(struct etimer *t); // Restart the timer from current time. 
void etimer_adjust(struct etimer *et, int td); // Adjust the expiration time for an event timer.
clock_time_t etimer_start_time(struct etimer *et);//Get the start time for the event timer.
void etimer_stop(struct etimer *t); // Stop the timer. 
int etimer_expired(struct etimer *t); // Check if the timer has expired. 
int etimer_pending(); // Check if there are any non-expired event timers.
clock_time_t etimer_next_expiration_time(); // Get the next event timer expiration time.
void etimer_request_poll(); // Inform the etimer library that the system clock has changed.
PROCESS_NAME(etimer_process);
 ```  
etimer的数据结构：
``` c
struct etimer {
  struct timer timer;
  struct etimer *next;
  struct process *p;
};
```
可以看出etimer与process数据结构类似都是链表形式。并且包含有timer结构类型变量.API函数etimer的也要丰富的多。并且有了一个etimer_process进程。还是举个实际的例子看看：
``` c
 #include "sys/etimer.h"
 
 PROCESS_THREAD(example_process, ev, data)
 {
   static struct etimer et;
   PROCESS_BEGIN();
 
   /* Delay 1 second */
   etimer_set(&et, CLOCK_SECOND);
 
   while(1) {
     PROCESS_WAIT_EVENT_UNTIL(etimer_expired(&et));
     /* Reset the etimer to trig again in 1 second */
     etimer_reset(&et);
     /* ... */
   }
   PROCESS_END();
 }
```
可以看到timer是用在中断函数中，而etimer是用在进程中的。而进程相关的宏我们已经研究过了。这里的PROCESS_BEGIN和PROCESS_END展开为switch(c){ case 0:{...} };所以只需要看中间部分就好了。  
由于etimer_set函数是在进程被执行的时候才会被调用的，所以函数的入口参数不需要进程信息，在etimer_set函数中用PROCESS_CURRENT();(其实就是process_current变量，指向当前正在运行的进程)得到当前的进程，并将此变量赋值给et->p。etimer_set函数执行以下两条语句：
``` c
timer_set(&et->timer, interval);
add_timer(et);
```
第一条是timer数据结构的初始化，包含记录当前时间，并设置时间触发的距离。第二条语句是更新etimer的事件触发点，add_timer做四件事：
* 执行etimer_request_poll();// = process_poll(&etimer_process);
* 如果新的etimer数据结构不在链表中，则将其加入链表，并将链表指向新加入的数据结构；
* 将当前进程赋值给新的etimer数据结构的进程指针；
* 更新etimer事件触发的时间距离。

执行process_poll意味着etimer_process进程需要被执行。  
例如加入新的etimer之前需要在10s后执行某个etimer操作，加入后执行update_timer会更新这个时间为当前链表中的所有etimer的start+interval-now的最小值作为下次发生etimer事件的时间距离next_expiration。很简单，某事件需要在更短的时刻被执行，而事件的执行需要etimer条件，而etimer条件的出现依赖于next_expiration-now是否为0，now是clock_time函数得到的值。

## 5.2 etimer_process
etimer_process进程做了两件事：
* 删除已经停掉的或执行出错的进程的etimer在etimer_list上的节点；
* 如果有计时时间到的进程则，再次执行执行etimer_request_poll();// = process_poll(&etimer_process);
执行etimer_request_poll的作用，参考[Contiki进程分析-struct process](https://xubovey.github.io/2016/10/20/161020-Contiki-process-analysis/)。  
那么etimer_process进程什么时候被添加到进程链表，又什么时候开始执行了呢？  
答案是在main函数中有`process_start(&etimer_process, NULL);`语句完成了将etimer_process进程添加到process_list中。

# 6. ctimer
回调定时器，采用`clock_time()`获取当前时间，功能与etimer类似，不同的是`ctimer_set()`函数的入口参数和etimer不一样，多了一个函数指针和数据指针。  
还有ctimer不适合应用在中断函数中。
## 6.1 ctimer的API函数
``` c
void ctimer_set(struct ctimer *c, clock_time_t t, void(*f)(void *), void *ptr); // Start the timer.
void ctimer_reset(struct ctimer *t); // Restart the timer from the previous expiration time.
void ctimer_restart(struct ctimer *t); // Restart the timer from current time. 
void ctimer_stop(struct ctimer *t); // Stop the timer. 
int ctimer_expired(struct ctimer *t); // Check if the timer has expired. 
```
ctimer数据结构如下：
``` c
struct ctimer {
  struct ctimer *next;
  struct etimer etimer;
  struct process *p;
  void (*f)(void *);
  void *ptr;
};
```
在etimer基础上，多了进程指针`struct process *p;`，函数指针`void (*f)(void *);`和数据指针`void *ptr;`。  
## 6.2 举个例子
``` c
 #include "sys/ctimer.h"
 static struct ctimer timer;
 
 static void
 callback(void *ptr)
 {
   ctimer_reset(&timer);
   /* ... */
 }
 
 void
 init(void)
 {
   ctimer_set(&timer, CLOCK_SECOND, callback, NULL);
 }
```
函数ctimer_set内容如下：
``` c
void
ctimer_set(struct ctimer *c, clock_time_t t, void (*f)(void *), void *ptr)
{
  ctimer_set_with_process(c, t, f, ptr, PROCESS_CURRENT());
}
```
继续展开
``` c
void
ctimer_set_with_process(struct ctimer *c, clock_time_t t, void (*f)(void *), void *ptr, struct process *p)
{
  PRINTF("ctimer_set %p %u\n", c, (unsigned)t);
  c->p = p;  //将当前进添加到ctimer数据结构中.
  c->f = f;  //将回调函数添加到ctimer数据结构中.
  c->ptr = ptr; //将入口参数数据指针添加到ctimer数据结构中.
  if(initialized) { //初始化ctimer数据结构总的etimer数据结构.
  /*
    #define PROCESS_CONTEXT_BEGIN(p) {\
    struct process *tmp_current = PROCESS_CURRENT();\
    process_current = p
  */
    PROCESS_CONTEXT_BEGIN(&ctimer_process);

    etimer_set(&c->etimer, t);

  /*
    #define PROCESS_CONTEXT_END(p) process_current = tmp_current; }
  */
    PROCESS_CONTEXT_END(&ctimer_process);
  } else {
    c->etimer.timer.interval = t;
  }

  list_add(ctimer_list, c);
}
```
其中if语句展开如下：
``` c
if(initialized) { //1表示ctimer_process进程已经被执行.
  struct process *tmp_current = PROCESS_CURRENT();
  process_current = &ctimer_process;
  etimer_set(&c->etimer, t);
  process_current = tmp_current;
}
```
## 6.3 进程切换
为什么需要进程切换再执行`etimer_set`函数呢，因为`etimer_set`会在其链表中添加新节点，并将当前进程添加进来。此处是将`ctimer_process`进程添加到etimer的链表中，这样对应的etimer事件就不会直接触发最终进程，而是触发`ctimer_process`进程，再由`ctimer_process`进程找到对应的回调函数完成事件处理。
`ctimer_process`进程中核心代码如下：
``` c
while(1) {
    PROCESS_YIELD_UNTIL(ev == PROCESS_EVENT_TIMER);
    for(c = list_head(ctimer_list); c != NULL; c = c->next) {
      if(&c->etimer == data) {
	      list_remove(ctimer_list, c);
	      PROCESS_CONTEXT_BEGIN(c->p);
	      if(c->f != NULL) {
	        c->f(c->ptr);
	      }
	      PROCESS_CONTEXT_END(c->p);
	      break;
      }
    }
  }
```
`ctimer_process`进程是在执行过`ctimer_init()`函数中启动的。

# 7. rtimer
实时时钟定时器，用于处理实时的、紧急的事件，被处理的事件或被调用的处理函数的运行事件必须是可评估的，防止事件处理函数执行事件过长，影响实时性能。和其它几种定时器依赖与tiemr或stimer，rtimer有独立的定时器和调度方式。  
函数`RTIMER_NOW()`可获得当前时间，`TIMER_SECOND`指定1s钟需要多少个tiks。另外rtimer最大的特点是可以抢占其它正在运行的进程。同样也是这个特点，只有中断中可安全使用的process_poll()之类的函数可以被执行，而会引起与其它进程冲突的函数则必须要异步方式执行。  
rtimer的API函数：
``` c
RTIMER_CLOCK_LT(a, b); // This should give TRUE if 'a' is less than 'b', otherwise false.
RTIMER_ARCH_SECOND; // The number of ticks per second. 
void rtimer_arch_init(void); // Initialize the rtimer architecture code. 
rtimer_clock_t rtimer_arch_now(); // Get the current time. 
int rtimer_arch_schedule(rtimer_clock_t wakeup_time); //  Schedule a call to rtimer_run_next().
```


