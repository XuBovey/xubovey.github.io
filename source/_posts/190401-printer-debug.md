---
title: 190401-tr_printer_debug
toc: true
date: 2019-04-01 20:57:24
categories:
tags:
---

## 2A版本打印机调试记录

### 打印机接口调试
版本变更：更换电平转换芯片，更换数据输出接口使layout更容易。
修改代码后发现不能正常打印测试内容。经过各种验证得出如下结论：
* 可打印整行直线，即DAT线长高
* CLK,DAT,LATCH,DST信号波形良好无畸变

尝试上拉、下拉无果

修改GPIO输出速度为High 无果

示波器测试各信号线波形良好。各种测试无果。

最后在DAT,CLK,LATCH信号线上串联47ohm电阻，问题解决.
1. DAT+CLK+LATCH OK
2. ONLY DAT NOT OK
3. DAT+CLK OK
4. ONLY CLK OK
5. CLK串联电阻替换为对地并联电容，问题解决

因为所用示波器采样率低，测试为发现波形明显异常。
根据并联电容解决问题，分析CLK沿变太快导致接收异常。






