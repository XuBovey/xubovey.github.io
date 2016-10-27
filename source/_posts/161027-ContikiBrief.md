---
title: 转载-Contiki简介
toc: true
date: 2016-10-27 15:57:33
categories:
- Contiki
- 转载
tags:
- Contiki
- 简介
---

[转载](https://segmentfault.com/a/1190000003850974)
注：文章转自他处，未找到原创作者，有不妥之处请原创作者联系笔者。

>本文介绍了 Contiki 是什么、contiki 的特点、Contiki 事件驱动(Event-driven) 编程模型、contiki 包含的无线网络协议栈 6Lowpan/RPL/Coap、仿真工具 Cooja/MSPsim、文件系统 Coffee File system(CFS)、shell 命令工具等,指出了 Contiki 的应用领域。最后给出了 Contiki 相关网站、教程和代码等。

<!--more-->
# Contiki操作系统介绍
Contiki 是一个开源的、高度可移植的多任务操作系统,适用于联网嵌入式 系统和无线传感器网络,由瑞典计算机科学学院(Swedish Institute of Computer Science)的 Adam Dunkels 和他的团队开发。Contiki 完全采用 C 语言开发,可移 植性非常好,对硬件的要求极低,能够运行在各种类型的微处理器及电脑上,目 前已经移植到 8051 单片机、MSP430、AVR、ARM、PC 机等硬件平台上。Contiki 适用于存储器资源十分受限的嵌入式单片机系统,典型的配置下 Contiki 只占用 约 2Kbytes 的 RAM 以及 40Kbytes 的 Flash 存储器。Contiki 是开源的操作系统, 适用于 BSD 协议,即可以任意修改和发布,无需任何版权费用,因此已经应用 在许多项目中。

Contiki 操作系统是基于事件驱动(Event-driven)内核的操作系统,在此内 核上,应用程序可以在运行时动态加载,非常灵活。在事件驱动内核基础上, Contiki 实现了一种轻量级的名为 protothread 的线程模型,来实现线性的、类似 于线程的编程风格。该模型类似于 Linux 和 windows 中线程的概念,多个线程共 享同一个任务栈,从而减少 RAM 占用。Contiki 还提供一种可选的任务抢占机制、 基于事件和消息传递的进程间通信机制。Contiki 中还包括一个可选的 GUI 子系统,可以提供对本地串口终端、基于 VNC 的网络化虚拟显示或者 Telnet 的图形 化支持。

Contiki 系统内部集成了两种类型的无线传感器网络协议栈:uIP 和 Rime。 uIP 是一个小型的符合 RFC 规范的 TCP/IP 协议栈,使得 contiki 可以直接和 Internet 通信。uIP 包含了 IPv4 和 IPv6 两种协议栈版本,支持 TCP、UDP、ICMP 等协议,但是编译时只能二选一,不可以同时使用。Rime 是一个轻量级为低功 耗无线传感器网络设计的协议栈,该协议栈提供了大量的通信原语,能够实现从 简单的一跳广播通信,到复杂的可靠多跳数据传输等通信功能。

# Contiki操作系统特点
* 事件驱动(Event-driven)的多任务内核  
    Contiki 基于事件驱动模型,即多个任务共享同一个栈(stack),而不是每个 任务分别占用独立的栈(如 uCOS、FreeRTOS、Linux 等)。Contiki 每个任务只 占用几个字节的 RAM,可以大大节省 RAM 空间,更适合节点资源十分受限的 无线传感器网络应用。

* 低功耗无线传感器网络协议栈  
    Contiki 提供完整的 IP 网络和低功耗无线网络协议栈。对于 IP 协议栈,支 持 IPv4 和 IPv6 两个版本,IPv6 还包括 6Lowpan 帧头压缩适配器,ROLL RPL 无线网络组网路由协议、CoRE/CoAP 应用层协议,还包括一些简化的 Web 工具, 包括 Telnet、http 和 web 服务等。Contiki 还实现了无线传感器网络领域知名的 MAC 和路由层协议,其中 MAC 层包括 X-MAC、CX-MAC、ContikiMAC、 CSMA-CA、LPP 等,路由层包括 AODV、RPL 等。

* 集成无线传感器网络仿真工具  
    Contiki 提供了 Cooja 无线传感器网络仿真工具,能够多对协议在电脑上进行仿真,仿真通过后才下载到节点上进行实际测试,有利于发现问题,减少调试工作量。除此之外,Contiki 还提供 MSPsim 仿真工具,能够对 MSP430 微处理 器进行指令级模拟和仿真。仿真工具对于科研、算法和协议验证、工程实施规划、 网络优化等很有帮助。

* 集成 Shell 命令行调试工具  
    无线传感器网络中节点数量多,节点的运行维护是一个难题,contiki 可以通 过多种交互方式,如 Web 浏览器,基于文本的命令行接口,或者存储和显示传 感器数据的专用程序等。基于文本的命令行接口是类似于 Unix 命令行的 Shell 工具,用户通过串口输入命令可以查看和配置传感器节点的信息、控制其运行状 态,是部署、维护中实用而有效的工具。

* 基于Flash的小型文件系统:CoffeeFileSystem  
    Contiki 实现了一个简单、小巧、易于使用的文件系统,称为 Coffee File System (CFS),它是基于 Flash 的文件系统,用于在资源受限的的节点上存储数据和程 序。CFS 是充分传感器网络数据采集、数据传输需求以及硬件资源受限的特点而 设计的,因此在耗损平衡、坏块管理、掉电保护方面、垃圾回收、映射机制方等 方面进行优化,具有使用的存储空间少、支持大规模存储的特点。CFS 的编程方 法与常用的 C 语言编程类似,提供 open、read、write、close 等函数,易于使用。

* 集成功耗分析工具  
    为了延长传感器网络的生命周期,控制和减少传感器节点的功耗至关总重 要,无线传感器网络领域提出的许多网络协议都围绕降低功耗而展开。为了评估 网络协议以及算法能耗性能,需要测量出每个节点的能量消耗,由于节点数量多, 使用仪器测试几乎不可行。Contiki 提供了一种基于软件的能量分析工具,自动 记录每个传感器节点的工作状态、时间,并计算出能量消耗,在不需要额外的硬 件或仪器的情况下就能完成网络级别的能量分析。Contiki 的能量分析机制既可 用于评价传感器网络协议,也可用于估算传感器网络的生命周期。

* 开源免费  
    Contiki 采用 BSD 授权协议,用户可以下载代码,用户科研和商业,且可以任意修改代码,无需任何专利以及版权费用,是彻底的开源软件。尽管是开源软 件,但是 contiki 开发十分活跃,在持续不断更新和改进之中。Contiki 的作者 Adam 是一个编程的天才,它发明了 LwIP、uIP、Protothred、contiki 等软件,都在工 业界得到广泛应用,大家熟知的 LwIP 就是一个例子。Adam 还是 IPSO 组织的发 起人之一,未来将会不断推进 6Lowpan 的标准化及应用。

# 总结
Contiki 完全 C 语言开发、易于移植、支持大量的硬件平台和开发工具、事 件驱动机制占用内存小、集成了多种无线传感器网络协议、无专利和版权费、集 成仿真工具等特点和优势,已经成为无线传感器网络学术研究和产品开发的理想 平台,在欧洲已经得到广泛应用,并逐渐得到其它地区开发人员的支持。随着物 联网、无线传感器网络的发展,IP 地址将耗尽,骨干网络必将升级到 IPv6,因 此 6Lowpan 标准被越来越多的标准化组织所采纳,研发 6lowpan 的人员将越来 越多,这将使得 contiki 很可能成为嵌入系统中的 Linux,在物联网领域得到广泛 应用,发挥重要作用。

# 参考资料

[Contiki 官方网站]
[Contiki Wiki]
[Instant Contiki 开发环境]
[物联网开发论坛]
[Contiki 源代码文档]
[Contiki 代码下载]
或者使用Git 工具下载 contiki 代码:
`git clone git://contiki.git.sourceforge.net/gitroot/contiki/contiki`
