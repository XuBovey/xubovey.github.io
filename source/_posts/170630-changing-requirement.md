---
title: 170630-changing requirement
toc: true
date: 2017-06-30 11:04:38
categories:
tags:
---

摘自[软件开发为什么很难](http://icodeit.org/2017/01/why-software-is-complex/)， 版权声明：自由转载-非商用-非衍生-保持署名| Creative Commons BY-NC-ND 3.0

## 一个有意思的举例
软件开发为什么很难中有一个需求不断变化的举例，很有意思，特意给记录下来  

需求的变化方向

作为程序员，有一天你被要求写一段代码，这段代码需要完成一件很简单的事：

打印”Hello, world”5次
很容易嘛，你想，然后顺手就写下了下面这几行代码：
```
print("Hello, world")
print("Hello, world")
print("Hello, world")
print("Hello, world")
print("Hello, world")
```
不过，拷贝粘贴看起来有点低端，你做了一个微小的改动：
```
for(var i = 0; i < 5; i++) {
  print("Hello, world")
}
```
看起来还不错，老板的需求又变成了打印”Goodbye, world”5次。既然是打印不同的消息，那何不把消息作为参数呢？
```
function printMessage(message) {
  for(i = 0; i < 5; i++) {
      print(message);
  }
}

printMessage("Hello, world")
printMessage("Goodbye, world")
```
有了这个函数，你可以打印任意消息5次了。老板又一次改变了需求：打印”Hello, world”13次（没人知道为什么是13）。既然次数也变化了，那么一个可能是将次数作为参数传入：
```
function printMessage(count, message) {
  for(i = 0; i < count; i++) {
      print(message);
  }
}

printMessage(13, "Hello, world");
printMessage(5, "Goodbye, world");
```
完美，这就是抽象的魅力。有了这个函数，你可以将任意消息打印任意次数。不过老板是永远无法满足的，就在这次需求变化之后的第二天，他的需求又变了：不但要将”Hello, world”打印到控制台，还要将其计入日志。

没办法，通过搜索JavaScript的文档，你发现了一个叫做高阶函数的东东：函数可以作为参数传入另一个参数！

```
function log(message) {
  system.log(message);
}

function doMessage(count, message, action) {
  for(i = 0; i < count; i++) {
      action(message);
  }
}

doMessage(5, "Hello, world", print);
doMessage(5, "Hello, world", log);
```
这下厉害了，我们可以对任意消息，做任意次的任意动作！再回过头来看看那个最开始的需求：

打印”Hello, world”5次
稍微分割一下这句话：打印，”Hello, world”，5次，可以看到，这三个元素最后都变成了可以变化的点，软件开发很多时候正是如此，需求可能在任意可能变化的方向上变化。这也是各种软件开发原则尝试解决的问题：如何写出更容易扩展，更容易响应变化的代码来。

小节

软件的复杂性来自于大量的不确定性，而这个不确定性事实上是无法避免的，而且每个软件都是独一无二的。另一方面，软件的需求会以各种方式来变化，而且往往会以开发者没有预料到的方向。比如上面这个小例子中看到的，最后的需求可能会变成将消息以短信的方式发送给手机号以185开头的用户手机上。
