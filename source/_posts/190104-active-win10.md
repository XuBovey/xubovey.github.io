---
title: 激活win10
toc: true
date: 2019-01-04 08:56:42
categories:
tags:
---


## 莫名的提升需要激活
自从自动升级到win10系统后，本来使用良好的，可近日开始自动弹出提示过期需激活之类的窗口。本来想不管继续使用的，可是每天提示不是办法。所以就尝试解决，经过一些处理，找到了很简单的方法，特记录如下。

## 相关密钥
### Win10家庭版密钥：
* 家庭版：TX9XD-98N7V-6WMQ6-BX7FG-H8Q99
* Core 家庭版：YTMG3-N6DKC-DKB77-7M9GH-8HVX7
* CoreSingleLanguage单语言win1064家庭版激活密钥：BT79Q-G7N6G-PGBYW-4YWX6-6F4BT
* CoreCountrySpecific激活64位win10特定国家家庭版：N2434-X9D7W-8PF6X-8DV9T-8TYMD
* Professional64位系统win10激活码(家庭版)：TX9XD-98N7V-6WMQ6-BX7FG-H8Q99
* Win10家庭版N：3KHY7-WNT83-DGQKR-F7HPR-844BM
* Win10单语音家庭版：7HNRX-D7KGG-3K4RQ-4WPJ4-YTDFH
* Win10特殊国家家庭版：PVMJN-6DFY6-9CCP6-7BKTT-D3WVR

* 专业版：W269N-WFGWX-YVC9B-4J6C9-T83GX
* 企业版：NPPR9-FWDCX-D2C8J-H872K-2YT43
* 教育版：NW6C2-QMPVW-D7KKK-3GKT6-VCFB2
* 专业版N：MH37W-N47XK-V7XM9-C7227-GCQG9
* 企业版N：DPH2V-TTNVB-4X9Q3-TJR4H-KHJW4
* 教育版N：2WH4N-8QGBV-H22JP-CT43Q-MDWWJ
* 企业版LSTB：WNMTR-4C88C-JK8YV-HQ7T2-76DF9
* 企业版LSTB N：2F77B-TNFGY-69QQF-B8YKP-D69TJ

### 密钥选择
当前电脑装的是win10单语言版，所以选择密钥为`BT79Q-G7N6G-PGBYW-4YWX6-6F4BT`

## 激活过程
1. 管理员权限打开命令窗口
2. 输入`slmgr.vbs /upk`卸载密钥，弹出窗口显示`已成功卸载了产品密钥`
3. 输入`slmgr /ipk BT79Q-G7N6G-PGBYW-4YWX6-6F4BT`安装密钥，提示`成功的安装了产品密钥`
4. 输入`slmgr /skms zh.us.to`弹出窗口`密钥管理服务计算机名成功的设置为zh.us.to`
5. 输入`slmgr /ato`，弹出窗口提示`成功的激活了产品`

