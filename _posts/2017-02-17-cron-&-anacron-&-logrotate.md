---
layout: post
title:  "cron & anacron & logrotate"
date:   2017-02-17 23:53:58 +0800
categories: my2829
---


#### Note: 以下所有文件、目录全是在CentOS 7中的默认位置，不同OS版本文件位置可能不一样

### cron

#### cron daemon
The cron service (daemon) runs in the background and constantly checks the `/etc/crontab` file, and `/etc/cron.*/` directories. It also checks the `/var/spool/cron/` directory.

- **`/etc/crontab`**文件 system-wide cron jobs，可以指定用户
- **`/etc/cron.*/`**文件夹
    - **`/etc/cron.d/`** 里面的文件格式同`/etc/crontab`
    - **`/etc/cron.hourly/`** 里面的文件是每小时执行的程序，`anacron`即是在这里配置了一个脚本，每小时检查是否有需要执行的任务
    - **`/etc/cron.daily/`**，**`/etc/cron.weekly/`**，**`/etc/cron.monthly/`** 里面的文件是每天/周/月执行的程序，由 anacron 调度执行，配置在 `/etc/anacrontab`文件
- **`/var/spool/cron/`** 文件夹下保存的是每个用户使用`crontab -e`编辑的任务

#### anacron

- 本身没有daemon，是通过在 /etc/cron.hourly 里配置一个任务来调度的
- 任务时间粒度是天，只能配置每多少天执行一次
- 并且，只比较day，不比较hour， e.g. 如果昨天晚上22点执行了一次，今天0点会再执行一次，不会判断中间是否间隔了24小时，只要日期中的day不一样就执行
- 不受电脑关机影响，电脑关机重启后原本计划在关机期间执行的任务会被执行
- 调度脚本 `/etc/cron.hourly/0anacron`

#### run-parts

- anacron 通过 run-parts 执行一个目录里的可执行文件
- run-parts 对文件名有要求，只能包含：大小写字母、数字、下划线、减号
- run-parts 按照文件名的“字典排序”顺序执行文件，文件名以0开头可以让文件先执行；通过--reverse选项可以翻转执行顺序


### logrotate

- 自动轮换、压缩、删除日志文件
- 可以按天、周、月，或者按日志文件的大小执行
- 默认按天检查，通过 anacron 配置定时任务 `/etc/cron.daily/logrotate`
- 如果需要按日志大小判断，需要手动设置定时任务，e.g. 每小时或每5小时执行logrotate
- 配置文件 `/etc/logrotate.conf` 、`/etc/logrotate.d/*`


### Reference

[https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/](https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/)


