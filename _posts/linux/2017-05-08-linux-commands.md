---
layout: post
title: linux 常用命令
categories: Linux
description: linux 常用命令
keywords: linux
---

整理下经常用到的命令备忘。

![image](/images/posts/linux_commands.jpg)

| command                         | desc |
|:--------------------------------|:------------|
| `scp -P <port_number> <local_file_paht> <username>@<ip>:<remote_path>`| 上传本地文件到远程主机 |
| `scp -P <port_number> <username>@<ip>:<remote_path> <local_file_paht>`| 下载远程主机文件到本地 |
| `w`                      | 查看系统当前登录用户、负载情况              |
| `pkill -kill -t pts/2`  | 踢出已登录用户 |
| `du -hs ./folder_name`  | 显示目录所占用磁盘空间的大小，而不显示其下子目录和文件占用磁盘空间的信息 |
| `du -ha ./folder_name`  | 显示目录占用的磁盘空间大小，还要显示其下目录和文件占用磁盘空间的大小 |
| `find ./ -type f|wc -l` | 显示目录以及子目录文件数量 |
| `find ./ -maxdepth 1 -type f|wc -l` | 只想查找当前目录的文件数量 |
| `du -h |sort -hr|head -20` | 按大小排序当前路径文件大小 |
| 网络相关 | |
| `ip a` | 查看所有网卡及状态 |
| 版本信息查看||
| `cat /proc/version` | 查看内核信息 |
| `uname -a` | 查看内核信息 |
| `cat /etc/issue` | 查看发行版 |
| `cat /etc/centos-release` | 查看发行版 |
| `lsb_release -a` | 查看发行版 |
| `cat error.log | grep -C 5 'nick'`| grep 查看日志|
| `nohup xxx > foo.log 2>&1 &` | 输出日志到指定文件 |
| `ps -ef |grep -v grep|grep tomcat |awk '{print $2}'|xargs kill -9` | 杀进程 |
| `mv /etc/localtime /etc/localtime.bak` | 备份时区配置 |
| `ln -s /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime` | 修改时区 |
| `ntpdate 1.cn.pool.ntp.org` | 同步时间 |
| `curl -H "Content-Type: application/json" -X POST --data '{"key":"val"}' http://ip:port/path` | 提交 `json` 请求 |
| `pv -L 2m file > newFile`| 复制文件并显示操作进度；限制速度为 2mb/s |
| `pv file | gzip > .file.gz`| 压缩文件并显示进度 |

CentOS 7 相关命令

|command | desc |
|:-------|:---|
| `systemctl status mysqld.service` | 查看 mysql 服务状态 |
| `firewall-cmd --zone=public --add-port=3306/tcp --permanent` | 防火墙添加端口|
