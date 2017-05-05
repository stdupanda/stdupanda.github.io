---
layout: post
title: Ubuntu 下 Python + psutil 获取系统信息
categories: Python
description: Ubuntu 下 Python + psutil 获取系统信息
keywords: Python, Ubuntu, psutil
---

`psutil`是一个跨平台 `Python` 库，能够轻松实现获取系统运行的进程和系统利用率（包括CPU、内存、磁盘、网络等）信息。

它主要应用于系统监控，分析和限制系统资源及进程的管理。它实现了同等命令行工具提供的功能，如ps、top、lsof、netstat、ifconfig、who、df、kill、free、nice、ionice、iostat、iotop、uptime、pidof、tty、taskset、pmap等。目前支持32位和64位的Linux、Windows、OS X、FreeBSD和Sun Solaris等操作系统，

# 配置 Python
```bash
sudo apt-get install python
sudo apt-get install python-dev
# 同时需要保证 gcc 已安装
wget https://pypi.python.org/packages/source/p/psutil/psutil-3.2.1.tar.gz --no-check-certificate
tar zxvf psutil-3.2.1.tar.gz
cd psutil-3.2.1
sudo python setup.py install
# 即可完成安装
```
# 创建 py 脚本

创建 py 脚本文件：
```bash
vim get_sys_info.py
```
内容如下：
```python
#!/usr/bin/env python
#coding:utf-8
import psutil
import time
import sys
from optparse import OptionParser
parser = OptionParser()
parser.add_option("-t", "--time", dest="time",
         help="此参数可查看当前下载占的带宽,-t是测试时间", metavar="10")
parser.add_option("-d", "--deamon",action="store_false", dest="deamon", default=True,
         help="后台运行此脚本")

def Sysinfo():
  Boot_Start = time.strftime("%Y-%m-%d %H:%M:%S",time.localtime(psutil.boot_time()))
  time.sleep(0.5)
  Cpu_usage = psutil.cpu_percent()
  RAM = int(psutil.virtual_memory().total/(1027*1024))
  RAM_percent = psutil.virtual_memory().percent
  Swap = int(psutil.swap_memory().total/(1027*1024))
  Swap_percent = psutil.swap_memory().percent
  Net_sent = psutil.net_io_counters().bytes_sent
  Net_recv = psutil.net_io_counters().bytes_recv
  Net_spkg = psutil.net_io_counters().packets_sent
  Net_rpkg = psutil.net_io_counters().packets_recv
  BFH = r'%'
  print " \033[1;32m开机时间：%s\033[1;m" % Boot_Start
  print " \033[1;32m当前CPU使用率：%s%s\033[1;m" % (Cpu_usage,BFH)
  print " \033[1;32m物理内存：%dM\t使用率：%s%s\033[1;m" % (RAM,RAM_percent,BFH)
  print "\033[1;32mSwap内存：%dM\t使用率：%s%s\033[1;m" % (Swap,Swap_percent,BFH)
  print " \033[1;32m发送：%d Byte\t发送包数：%d个\033[1;m" % (Net_sent,Net_spkg)
  print " \033[1;32m接收：%d Byte\t接收包数：%d个\033[1;m" % (Net_recv,Net_rpkg)

  for i in psutil.disk_partitions():
    print " \033[1;32m盘符: %s 挂载点: %s 使用率: %s%s\033[1;m" % (i[0],i[1],psutil.disk_usage(i[1])[3],BFH)

def Net_io(s):
  x = 0
  sum = 0
  while True:
    if x >= s:
      break
    r1 = psutil.net_io_counters().bytes_recv
    time.sleep(1)
    r2 = psutil.net_io_counters().bytes_recv
    y = r2 - r1
    print "%.2f Kb/s" % (y / 1024.0)
    sum += y
    x += 1
  result = sum / x
  print "\033[1;32m%s秒内平均速度：%.2f Kb/s \033[1;m" % (x,result / 1024.0)
if __name__ == "__main__":
  (options, args) = parser.parse_args()
  if options.time:
    Net_io(int(options.time))
  else:
    Sysinfo()
```
# 脚本运行方式

脚本运行方式有2种

- 不加参数，直接输出当前操作系统信息
```bash
python get_sys_info.py
```

- 加入“-t 一个数字”，查看给定秒内的平均网络速度
```bash
python get_sys_info.py -t 10
```
