---
title: Electron serialport 应用
toc: true
date: 2016-11-05 19:50:52
categories:
tags:
---
# 参考1
[nodejs/node-gyp](https://github.com/nodejs/node-gyp)

* npm install -g node-gyp
* npm install --global --production windows-build-tools

如果有多个Python版本，需要给`node-gyp`指定版本：
```
node-gyp --python /path/to/python2.7
npm config set python /path/to/executable/python2.7
```
# 参考2
[paulolc/electron-quick-start-serialport](https://github.com/paulolc/electron-quick-start-serialport)
```
git clone https://github.com/paulolc/electron-quick-start-serialport
cd electron-quick-start-serialport
npm install && npm start
```
