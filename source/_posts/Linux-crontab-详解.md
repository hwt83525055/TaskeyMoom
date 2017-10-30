---
title: Linux crontab 详解
date: 2017-4-26 16:02:27
tags: Linux
---
#### 欢迎大家加我的微信
因在应用中要挂项目的脚本所以需要使用到Crontab，所以查了查资料整理了一下crontab的基础使用方法

```bash
$ cron status
```
查询当前crontab的运行状态
```bash
$ crontab -e
$ crontab -l
$ crontab -r
```
这个则是对crontab 脚本的 edit， list, remove  ，crontab功能的主题部分
![Aaron Swartz](https://raw.githubusercontent.com/hwt83525055/PhotoSource/master/Crontab_1.jpg)
可以看到crontab-edit中的每一行记录由以下几个部分组成
#### 分钟 小时 日期 周 Act

 这个应该很好理解， 即在某日期周几某小时某分钟 执行某操作,值得注意的是一般在command中我们需要使用的是绝对地址
 | 字段名称        | 说明         | 取值范围  |
|:---|:---:|:---:|
| 分钟      | 每小时的第几分钟执行 | 0-59 |
| 小时    | 每日的第几个小时执行 |0-23|
| 日期 | 每月的第几天执行 | 1-31 |
|月历 |每年的第几月执行|1-12|
|星期|每周的第几天执行|0-6|

### 特殊的字符含义
在crontab中常用的几个特殊字符
| 字段名称        | 说明    |     
|:---:|:---:|
|s| 表示任何时刻|
|-|表示一个段|
|,|表示分割|
|/n|表示每个n的单位执行一次，可以写成1-23/1.|
### 服务的启动
想控制crontab的运行状态依靠以下几个命令
```bash
service cron start
service cron stop
service cron restart
```
### 日志
有关crontab的log一般都存储在/var/log/cron.log 这个文件里，自行查看即可
