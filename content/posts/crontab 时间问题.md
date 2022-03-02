---
title: "crontab 时间问题"
date: 2022-01-17T21:50:00+08:00
tags: ['linux', 'crontab']
---
在使用 crontab 时我发现从我 crontab 中运行的任务 log 的时间与我 crontab 设置的时间不一样

```bash
$ crontab -l
0 5-22/2 * * * /usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1
```

输出的 log 文件

```bash
2021-06-05T23:00:24.873Z        info    app/main.go:105 Logger init finish
Creating dev_update-free-class_run ...
Creating dev_update-free-class_run ... done
2021-06-06T01:01:50.501Z        info    app/main.go:105 Logger init finish
Creating dev_update-free-class_run ...
Creating dev_update-free-class_run ... done
2021-06-06T03:03:12.640Z        info    app/main.go:105 Logger init finish
Creating dev_update-free-class_run ...
Creating dev_update-free-class_run ... done
2021-06-06T05:00:24.019Z        info    app/main.go:105 Logger init finish
Creating dev_update-free-class_run ...
Creating dev_update-free-class_run ... done
2021-06-06T07:01:14.994Z        info    app/main.go:105 Logger init finish
Creating dev_update-free-class_run ...
Creating dev_update-free-class_run ... done
2021-06-06T09:01:04.559Z        info    app/main.go:105 Logger init finish
Creating dev_update-free-class_run ...
Creating dev_update-free-class_run ... done
2021-06-06T11:03:42.683Z        info    app/main.go:105 Logger init finish
Creating dev_update-free-class_run ...
Creating dev_update-free-class_run ... done
2021-06-06T13:00:14.605Z        info    app/main.go:105 Logger init finish
```

但从 crontab 的运行日志来看，运行时间是没有问题的

```bash
➜  ~ grep CRON /var/log/syslog |grep docker
Jun  6 07:00:02 iZuf6af0ku1fuddplsmti5Z CRON[15266]: (root) CMD (/usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1)
Jun  6 09:00:01 iZuf6af0ku1fuddplsmti5Z CRON[20702]: (root) CMD (/usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1)
Jun  6 11:00:01 iZuf6af0ku1fuddplsmti5Z CRON[26089]: (root) CMD (/usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1)
Jun  6 13:00:01 iZuf6af0ku1fuddplsmti5Z CRON[31397]: (root) CMD (/usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1)
Jun  6 15:00:01 iZuf6af0ku1fuddplsmti5Z CRON[4488]: (root) CMD (/usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1)
Jun  6 17:00:01 iZuf6af0ku1fuddplsmti5Z CRON[10056]: (root) CMD (/usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1)
Jun  6 19:00:01 iZuf6af0ku1fuddplsmti5Z CRON[15631]: (root) CMD (/usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1)
Jun  6 21:00:01 iZuf6af0ku1fuddplsmti5Z CRON[23226]: (root) CMD (/usr/local/bin/docker-compose -f /root/dev/docker-compose.yml run --rm update-free-class >> ~/cron.log 2>&1)
```

## 结论

问题出在我的 docker image 中没有设置时区，导致时间错误

## 解决

首先镜像中需要安装 `tzdata`

在 Dockerfile 中加入

```bash
ENV TIME_ZONE=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone
```

就可以设置时区

---

但为了减小 image 体积，一般要将 tzdata 删除，所以不能使用链接 `ln` -snf ，而要直接复制 `cp`

```bash
ENV TIME_ZONE=Asia/Shanghai
RUN apk update && \
    apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone && \
    apk del tzdata
```

## 扩展

<https://askubuntu.com/questions/54364/how-do-you-set-the-timezone-for-crontab>

可以通过在 crontab 运行的命令中加入 `TZ='Asia/Shanghai 来改变时区`

使用命令 `tzselect 可以交互式的获取 TZ 的值`
