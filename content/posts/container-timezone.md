---
title: "如何在 Linux 容器中修改时区"
date: 2021-02-06T22:01:00+08:00
tags: ['docker']
---

在使用 Docker 打包运行软件时，经常遇到需要修改时区的情况。那么如何修改容器中的时区呢？下面收集了自己常用的几种方法。

- `/etc/localtime` 符号链接
- `TZ` 环境变量

<!--more-->

在了解如何在容器中修改时区前，先了解一下如何在非容器环境中修改时区。

## Linux 修改时区

 ### 检查当前时区

`timedatectl  ` 可以查看和更改系统的时间和日期。在所有基于systemd的现代Linux系统上都可以用。

```shell
➜  ~ timedatectl
```

输出：

```shell
               Local time: Fri 2021-02-12 18:18:05 CST
           Universal time: Fri 2021-02-12 10:18:05 UTC
                 RTC time: n/a
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

上面的输出显示系统的时区设置为 `Asia/Shanghai`

### 修改时区

在更改时区之前，需要找出要使用的时区的长名称。时区命名约定通常使用“地区/城市”格式。

此命令可以列出所有可用时区的长名称，或者通过 `ls` 查看 `/usr/share/zoneinfo`目录来获取时区长名称

```shell
timedatectl list-timezones
```

修改时区的思路基本有两个，设置 `localtime` 符号链接，或设置 `TZ` 环境变量。

- 通过将 `/etc/localtime` 文件符号链接到 `/usr/share/zoneinfo`目录中的二进制时区标识符来配置系统时区

  这里介绍两种操作方法，一种呢，是使用前面提到的 `timedatectl` 来设置；另一种呢，是直接创建符号链接。

  - `timedatectl`

    命令基本格式：

    ```shell
    timedatectl set-timezone <your_time_zone>
    ```

    例如，设置时区为 `Asia/Shanghai`

    ```shell
    timedatectl set-timezone Asia/Shanghai
    ```

  - 直接创建符号链接

    例如，设置时区为 `Asia/Shanghai`

    ```shell
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    ```

    

- 设置`TZ`环境变量

  当需要临时改变时区或只修改当前用户时区时，可以通过设置 `TZ` 环境变量来实现

  例如，设置时区为 `Asia/Shanghai`

  ```shell
  export TZ=Asia/Shanghai
  ```



## Linux 容器中修改时区

Linux 容器中设置时区，所使用的方法基本和非容器 Linux 的方法一致。

- `/etc/localtime` 符号链接

- `TZ` 环境变量

**不过，容器中必须已经安装了 `tzdata` 包才行，有些容器镜像中可能没有安装。**



### 修改时区

在构建容器镜像和创建容器这两个时间段设置时区，就可以达到很好的效果。

#### 镜像构建阶段

- `/etc/localtime` 符号链接

  ```dockerfile
  FROM nginx:1.16.1-alpine
  RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
  ```

- `TZ` 环境变量

  ```dockerfile
  FROM nginx:1.16.1-alpine
  ENV TZ=Asia/Shanghai
  ```

#### 容器创建阶段

- `TZ` 环境变量

  ```shell
  docker run -e TZ=Asia/Shanghai nginx:1.16.1-alpine
  ```



#### 参考资料

- https://linuxize.com/post/how-to-set-or-change-timezone-in-linux/#changing-the-time-zone-in-linux

