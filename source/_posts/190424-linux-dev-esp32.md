---
title: 190424-linux-dev-esp32
toc: true
date: 2019-04-24 16:34:26
categories:
tags:
---

# 开发环境
## 安装Linux下的软件需求
```
sudo apt-get install gcc git wget make libncurses-dev flex bison gperf python python-pip python-setuptools python-serial python-cryptography python-future python-pyparsing python-pyelftools
```
## linux环境设置
[下载地址：](https://dl.espressif.com/dl/xtensa-esp32-elf-linux64-1.22.0-61-gab8375a-5.2.0.tar.gz)
```
mkdir -p /home/username/esp
cd /home/username/esp
tar -xzf /home/username/Downloads/xtensa-esp32-elf-linux64-1.22.0-61-gab8375a-5.2.0.tar.gz
```
2.设置环境变量PATH，以便能搜索到我们的编译工具链
在/home/username/.profile中添加
```
export PATH=$PATH:/home/username/esp/xtensa-esp32-elf/bin
```
具体操作：
/usrname路径下```vi .profile```, 然后在文件尾部输入`o`，写入上面内容，输入`wq`退出。
```
source .profile
```
执行`echo $PATH`查看环境变量结果
## git clone SDK ESP-IDF
在esp目录下
```
git clone --recuresive http://github.com/espressif/esp-idf.git
```
...等一会

## 环境变量
```
echo "export IDF_PATH=~/esp/esp-idf" >> ~/.bashrc
source ~/.bashrc
```

## 编译helloworld
```
cd examples/get-started/hello_world/
make
```
## 烧写固件
识别到ttyUSB0设备后执行命令
```
sudo chmod 777 /dev/ttyUSB0
make flash monitor
```
提示`recipe for target 'flash' failed`，输入`sudo chmod 777 /dev/ttyUSB0`即可

# ESP-TOUCH
https://github.com/EspressifApp/ESP-TOUCHForAndroid




