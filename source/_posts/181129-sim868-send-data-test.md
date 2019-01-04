---
title: SIM868通信测试
toc: true
date: 2018-11-29 11:28:40
categories:
tags:
---

## 移动状态
### 测试条件
* 兰州-郑州高铁列车
* SIM868模块
* 移动卡
* mqtt

### 测试结果
* 建立网络连接后可在较大范围内（超过单基站覆盖范围）不掉线保持数据通信
* 数据延迟情况

|发送数据长度|发送延迟|
|--|--|
|字节|秒
|500|1-1.5|
|1000|3-4|  


* 移动状态联网比较困难，但是可在站台联网成功，之后保持较长距离和时间可持续通信

## 程序改进
### 使用AT+IPQSEND
通信情况改善很多，测试中最高达到2kB/s，测试为静态市内测试，直接给服务器发数据，没经过mqtt.

## bug
```
mqtt init do it.
[12:04:09.044] task_mqtt.c [user_msg_proc] rxdata from TASK 1,len =13,msg_p=0x0
[12:04:09.044] cmdType = 0
[12:04:09.044] mqtt_connect.
[12:04:09.044] 
[12:04:09.044] ---> try to connect mqtt server:gxd729
[12:04:09.044] 
[12:04:09.044] mqtt->keepalive = 0x3c
[12:04:09.044] mqtt->clientid = gxd729
[12:04:09.044] INFO: the at cmd func[0]'s range is 46-47
[12:04:09.049] INFO: funFirst:46  funLast:47
[12:04:09.049] simcom data send result = 1
[12:04:09.049] app_send_data_to_server return ok, send....
[12:04:09.049] mqttc is connecting to 114.67.229.68:3881...
[12:04:09.049] 
[12:04:09.049] 1. WrToModem:AT+CIPSEND=0,124
[12:04:09.143] 
[12:04:09.143] len=4,ReFrModem:
[12:04:09.147] > 
[12:04:09.147] at+cipsend? return >
[12:04:09.147] at+cipsend? return 1
[12:04:09.147] INFO: callback return 46,1
[12:04:09.147] 1. WrToModem:z
[12:04:09.255] 
[12:04:10.693] sec:340
[12:04:10.693] tcp=1,mqtt=0,tcp_tx=0
[12:04:10.693] 
[12:04:15.680] sec:345
[12:04:15.680] tcp=1,mqtt=0,tcp_tx=0
[12:04:15.680] len=14,ReFrModem:
[12:04:15.799] 0, SEND OK
[12:04:15.799] 
[12:04:15.799] send data done,rx SEND OK
[12:04:15.799] send data, return 4
[12:04:15.799] INFO: callback return 47,4
[12:04:15.799] 
[12:04:20.669] sec:350
[12:04:20.669] tcp=1,mqtt=0,tcp_tx=1
[12:04:20.669] 
[12:04:22.401] len=13,ReFrModem:
[12:04:22.401] 0, CLOSED
[12:04:22.401] 
[12:04:22.401] mqtt_disconnect :gxd729
[12:04:22.401] 
[12:04:22.401] send data err tcp disconnected
[12:04:22.401] app_send_data_to_server return err
[12:04:22.401] mqttc is disconnected.
[12:04:22.401] 
[12:04:22.401] connect 0 lose.
[12:04:22.401] INFO: the at cmd func[0]'s range is 48-7
[12:04:22.401] INFO: funFirst:48  funLast:7
[12:04:22.401] simcom_gsm_connect_server start.
[12:04:22.401] 1. WrToModem:AT+CIPSHUT
```

