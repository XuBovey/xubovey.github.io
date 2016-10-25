---
title: Contiki etimer_process进程分析
toc: true
date: 2016-10-24 14:00:43
categories:
- Contiki
tags:
- Contiki
- YIELD
---

contiki代码分析过程中，发现对进程的多次调用有必要进行一定的分析。  
在分析etimer_process进程的时候当时有点想不明白，后来想通了就来做个记录。

<!--more-->
# etmer_process函数体
为了方便分析while(1)中代码略去，只保留了PROCESS_*部分。
``` c
PROCESS_THREAD(etimer_process, ev, data)
{
  struct etimer *t, *u;
	
  PROCESS_BEGIN();

  timerlist = NULL;
  
  while(1) {
    PROCESS_YIELD();
    ...   
  }
  
  PROCESS_END();
}
```
对代码进行宏展开，代码中注释了重要的分析：  
第一次执行是在`process_start`函数中的`process_post_synch(p, PROCESS_EVENT_INIT, data);`语句完成的。
``` c
static char process_thread_etimer_process(struct pt *etimer_process, process_event_t ev, process_data_t data)
{
    struct etimer *t, *u;

    //PROCESS_BEGIN start
    char PT_YIELD_FLAG = 1;

    if (PT_YIELD_FLAG) {;}

    //第一次执行进程，lc=0，执行case 0
    //第二次执行进程，lc=__LINE__， 直接执行case __LINE__:
    switch((process_pt) -> lc) {
        case 0: 
    //PROCESS_BEGIN end

    timerlist = NULL; //第一次执行进程，etimer链表初始化。
    while(1) {
        //PROCESS_YIELD start
        do{
            //第一次执行进程到此时,执行清零操作；
            //第二次执行进程中，第二次执行while(1)循环时，执行清零操作。
            PT_YIELD_FLAG = 0; 
            (process_pt) -> lc = __LINE__;
            
            case __LINE__: 
            //第一次执行进程到此时，进入if语句，直接退出；
            //第二次执行进程，第二个循环执行到此，进入if语句，直接退出；
            if (PT_YIELD_FLAG == 0) 
            { return ;}            
        }while(0)
        //PROCESS_YIELD end
        
        //每次进入进程只执行1次while(1)循环。
        函数体代码部分   
    }

    //PROCESS_END start       
    };

    PT_YIELD_FLAG = 0;
    process_pt->lc = 0;

    return PT_ENDED;
    //PROCESS_END  end
}
```
来看看和常见的程序有什么不一样，我们常见的switch是这样的：
``` c
switch(c)
{
    case 0;
    case 1;
    case 2;
}
```
而这里的是这样的：
``` c
switch(c)
{
    case 0:
    /*do something*/
    while(1)
    {
        c = __LINE__;
        case __LINE__:
        /*do something*/
    }
}
```
即便时while(1)和case混合使用了，可是在语法上也是没有错的。__LINE__是内置宏，表示当前代码所在的行号。这里的case用0以外的数字都行，但是是用宏的方式给出的。为了通用性，避免出错，用行号再好不过了。  
上面etimer_process的代码，采用了case语句成功的解决了1个问题：可以退出while(1)循环，保证进程能按时结束。因此只有main函数中的while(1)能一直循环下去。
