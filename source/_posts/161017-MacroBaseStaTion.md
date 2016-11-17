---
title: C语言宏定义的使用
toc: true
date: 2016-10-17 10:59:56
categories: 
- C语言
- 技术
tags: 
- C
---

最近研究Contiki，看别人教程上分析宏定义，一层又一层，所以给自己充个电。
本文参考了hbprotoss的博客的[C语言宏的特殊用法和几个坑](http://hbprotoss.github.io/posts/cyu-yan-hong-de-te-shu-yong-fa-he-ji-ge-keng.html)和Anker's的Blog的[C语言宏高级用法 [总结]](http://www.cnblogs.com/Anker/p/3418792.html)
<!--more-->

# 用的最多的
最基础的也是最简单的，我们把常数替换成一个更容易识别的大写字符组合：
```
$ #define BUFFER_SIZE 100
```
然后程序预处理的时候会把所有的BUFFER_SIZE替换成100。

## 小基础
宏定义中的换行需要用反斜杠`\`

# 稍高级点的
我们会把比较函数用宏定义，在预处理阶段进行替换，从而避免程序执行过程中有大量的函数调用而降低性能，专业点讲就是牺牲容量提高性能。例如：
```
$ #define min(X, Y)  ((X) < (Y) ? (X) : (Y))
```
这里的X，Y是宏的参数，称作宏参数。  
功能很简单，找到并返回较小的。这里没有说值或数，因为X，Y可以是任何东西：变量，数字，字符，函数。所以这里的括号用的很多，括号的增加会避免运算符优先级问题引起的错误，其它类似宏的定义中同样需要认真、小心对待。举个例子：a=b*min(c,d);在min中有括号和没括号的时候是不一样的。试试看，想想看。  
但是使用中应当注意，尽量不要用函数展开宏，例如，建设x为1个函数，那么这个宏展开后将被调用两次，有可能两次的结果是不一样的，这样就会带来逻辑错误，遇到这种情况时可以用变量对函数做缓存然后再对宏进行展开。

# 他人踩的坑
看人代码时发现有的代码行都没有分号结尾，为了代码美观给加上后有报错，很是不爽。例如下面这个
```
#define SKIP_SPACES(p, limit)  \
     { char *lim = (limit);         \
       while (p < lim) {            \
         if (*p++ != ' ') {         \
           p--; break; }}}
...省略很多...
if (*p != 0)
   SKIP_SPACES (p, limit);
else ...           
```
为了看清问题等效成这样的
```
if (*p != 0)
{
    SKIP_SPACES (p, limit)
};
else ...
```
问题出来了if和else之间多了个分号，导致else没有if。  
实际中还有这样的：
```
$ #define min(X, Y)  ((X) < (Y) ? (X) : (Y));
```
那么引用的时候就可能是：
```
c = min(a,b)
```
有没有很奇怪的感觉。  
所以,if语句需要有{}。每行代码要有分号结束。然而如果if后没有{},还想正确呢，怎么处理呢。请继续看  
C语言中常用的是while(){},很少用do{}while(),如果在宏中外面加上do{}while(0)会怎样呢？
```
#define SKIP_SPACES(p, limit)  \
     do
     { char *lim = (limit);         \
       while (p < lim) {            \
         if (*p++ != ' ') {         \
           p--; break; }}}while(0)
```
if语句展开后：
```
  if (*p != 0)
     do
     { char *lim = (limit);         
       while (p < lim) {            
         if (*p++ != ' ') {         
           p--; break; }}}while(0);
  else 
```
哈哈完美解决。最后的while(0)吃掉了分号，if和else重新组成了一个完整的语句。

# 宏的递归引用
不做详细介绍，只说明预处理中只展开一次。  

# 宏参数预处理
阅读一些C代码的时候会看到一些宏定义中有#或者##。那么怎么解读呢
其实就是特殊符合：# ##
## #
在一个宏中的参数前面使用一个#,预处理器会把这个参数转换为一个字符数组。  
简化理解：#是“字符串化”的意思，出现在宏定义中的#是把跟在后面的参数转换成一个字符串。  
```
#define ERROR_LOG(module)   fprintf(stderr,"error: "#module"\n")
...
ERROR_LOG("add"); //转换为 fprintf(stderr,"error: "add"\n");
ERROR_LOG(devied =0); //转换为 fprintf(stderr,"error: devied=0\n");
```

## ##
“##”是一种分隔连接方式，它的作用是先分隔，然后进行强制连接。  
在普通的宏定义中，预处理器一般把空格解释成分段标志，对于每一段和前面比较，相同的就被替换。但是这样做的结果是，被替换段之间存在一些空格。如果我们不希望出现这些空格，就可以通过添加一些##来替代空格。例如：
```
#define TYPE1(type,name)   type name_##type##_type
TYPE1(int, c); //转换为：int 　name_int_type ; 
#define TYPE2(type,name)   type name##_##type##_type
TYPE2(int, d); //转换为：int 　d_int_type ; 
```

参考：
[C语言宏高级用法 [总结]](http://www.cnblogs.com/Anker/p/3418792.html)  
[C语言宏的特殊用法和几个坑](http://hbprotoss.github.io/posts/cyu-yan-hong-de-te-shu-yong-fa-he-ji-ge-keng.html)

***
