---
title: 环境变量
date: 2016-10-10 12:06:27
tags: 
---
### 设置环境变量
``` shell
export name=value
```

### 查看进程环境变量
``` shell
# 查看某个进程的环境变量
cat /proc/$PID/environ
```
每一个环境变量以name=value形式来描述，彼此间有null字符(\0)分割。

将\0替换成\n输出
``` shell
# 查看某个进程的环境变量
cat /proc/$PID/environ | tr '\0' '\n'
```
