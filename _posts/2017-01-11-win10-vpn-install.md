---
layout: post
title:  "Win10下无法安装思科VPN的解决方法"
date:   2017-01-11
categories: VPN
---

最近由于工作需要，需要远程接入客户的局域网进行工作，所以用到了思科VPN5.0。由于我的开发环境是Win10 64系统，在安装的时候出现了一些问题，特此记录。

### 错误一：
始终安装不成功，提示错误：Error 27848，经过搜索后得知，需要先安装64位补丁后才能完成安装。补丁下载地址：[http://download.csdn.net/detail/lc_1994/9431154
](http://download.csdn.net/detail/lc_1994/9431154)，这是一位好心人留下的。补丁安装完成后，在安装VPN5.0，就可以安装成功。

### 错误二：
安装成功后，却链接不了，提示错误：
![]({{ site.url }}/image/20170111/vpnerror.png)。<br />
这时候，需要修改注册表：
1. 找到 HKEY_LOCAL_MACHINE→SYSTEM→CurrentControlSet→Services→CVirtA，将DisplayName的值修改为：Cisco Systems VPN Adapter for 64-bit Windows；
2. 打开服务，找到Cisco Systems, Inc. VPN Service，重新启动服务，在链接VPN 就好了；
