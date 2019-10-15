---
title: 三步搞定用Docker安装Drupal8运行环境
date: 2019-10-01 09:27:54
category:
  - 原创
tags:
  - Drupal 8
  - Docker
---

本文介绍用 Docker 安装 Drupal8 运行环境的实现方法

<iframe src="//player.bilibili.com/player.html?aid=69595627&cid=120616823&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

[配套视频](https://www.bilibili.com/video/av69595627)

## 一、关于 Docker

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux 或 Windows 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

<!-- more -->

[百度百科关于 Docker](https://baike.baidu.com/item/Docker/13344470?fr=aladdin)

## 二、关于 Drupal

Drupal 是使用 PHP 语言编写的开源内容管理框架（CMF），它由内容管理系统（CMS）和 PHP 开发框架（Framework）共同构成。连续多年荣获全球最佳 CMS 大奖，是基于 PHP 语言最著名的 WEB 应用程序。截止 2011 年底，共有 13,802 位 WEB 专家参加了 Drupal 的开发工作；228 个国家使用 181 种语言的 729,791 位网站设计工作者使用 Drupal。著名案例包括：联合国、美国白宫、美国商务部、纽约时报、华纳、迪斯尼、联邦快递、索尼、美国哈佛大学、Ubuntu 等。

[百度百科关于 Drupal](https://baike.baidu.com/item/Drupal)

## 三、关于 Drupal 运行环境

### Apache (Recommended)

你可以使用 apache 的 mod_rewrite 扩展 drupal 的 clean url，在 Drupal8 里，clean urls 默认是开启的，且不能关闭，所以为了 Drupal 能正常工作，mod_rewrite 需要安装并开启。

虚拟主机必须配置包含 AllowOverride All，允许 drupal 的.htaccess 文件起作用。

### Database

MySQL 5.5.3/MariaDB 5.5.20/Percona Server 5.5.8 or higher with PDO and an InnoDB-compatible primary storage engine,

PostgreSQL 9.1.2 or higher with PDO,

### SQLite 3.6.8 or higher

### PHP

Drupal 8: PHP 5.5.9 or higher

## 三步搞定

看到配置环境，很多初学者会望而却步。正是得益于 Docker 的镜像和容器技术，可以让我们省去配置环境的麻烦。而且，这个环境可以和上线环境一模一样，省去了我们部署和维护的烦恼。

用 Docker 安装 Drupal8 运行环境只要三步，需要安装两个容器，前提是我们已经安装了 Docker。

### 第一步 安装数据库

在命令行输入以下命令：

```
docker run \
-e MYSQL_ROOT_PASSWORD=admin \
-e MYSQL_DATABASE=drupal8 \
-e MYSQL_USER=drupal8 \
-e MYSQL_PASSWORD=drupal8 \
-v mariadb:/var/lib/mysql \
-d \
--name mariadb \
mariadb
```

第一行的意思是创建容器，
第二行到第六行是数据库配置参数，
-d 是在后台运行该容器，
--name 给该容器取个名字，
最后一行是镜像名称。

完整的理解就是从 mariadb 镜像创建一个名称为 mariadb 的在后台运行的容器，按照既定的数据库配置参数。如果本地没有 mariadb 镜像，则自动从 Docker hub 上拉取后再创建容器。

运行结果如下：

```
Unable to find image 'mariadb:latest' locally
latest: Pulling from library/mariadb
5667fdb72017: Pull complete
d83811f270d5: Pull complete
ee671aafb583: Pull complete
7fc152dfb3a6: Pull complete
9f669c535a8b: Pull complete
a6de1092ee4e: Pull complete
ee37a2c88dd9: Pull complete
d927a3dd356c: Pull complete
d83c9d39c64f: Pull complete
1b0644883413: Pull complete
09a38adc2558: Pull complete
3c853415b952: Pull complete
2690cf0bfab9: Pull complete
3c68d64f060f: Pull complete
Digest: sha256:a32daf0281803fd96e86daf6b0293b4d476cede1b5ce80b18452dfa1405360ff
Status: Downloaded newer image for mariadb:latest
8cae72ad7ff02870c575c09eb3ad6f053c395287a3cfe17d7888991acc6cc254
```

### 第二步 安装网站运行服务器和 Drupal 源码

在命令行输入以下命令：

```
docker run \
--name drupal8 \
--link mariadb:mysql \
-p 80:80 \
-d \
drupal
```

命令的意思是，drupal 镜像创建一个名称为 drupal8 的在后台运行的容器。如果本地没有 drupal 镜像，则自动从 Docker hub 上拉取后再创建容器。
--link 连接名为 mariadb 容器（第一步已经创建），取一个链接名为 mysql，这个名字在第三部安装 Drupal 的过程中需要用到。切记！
-p 将容器的 80 端口映射到宿主机的 80 端口，在宿主机的浏览器就可以直接访问 localhost 了。

运行结果如下：

```
Unable to find image 'drupal:latest' locally
latest: Pulling from library/drupal
8f91359f1fff: Pull complete
bf2faaedf741: Pull complete
24cd1299a53e: Pull complete
17091cc665e4: Pull complete
ac9365919f9b: Pull complete
4f1b34e209ee: Pull complete
832757fa04a4: Pull complete
640a8cc59ee4: Pull complete
375d45a647bd: Pull complete
c265603c2115: Pull complete
b0b436e89a13: Pull complete
752098124903: Pull complete
baaf39033af6: Pull complete
eac75ead14e0: Pull complete
8494df29c26d: Pull complete
b9dd3d0f6cb9: Pull complete
Digest: sha256:899473656db6b2fb7343d9cfd8ab6493199b60500eed9d7202c0d2552c8c5b1d
Status: Downloaded newer image for drupal:latest
84d185875cd1e27c301042c743aab01757d3e9814557698b8d40a6b7b977e4ca
```

由于我们在安装容器时没有制定镜像的版本号，所以自动拉取 latest 版本，意思是最新版本的镜像。

安装完成两个容器之后我们可以查看 Docker 中的容器和镜像。
查看容器：

```
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
84d185875cd1        drupal              "docker-php-entrypoi…"   About an hour ago   Up About an hour    0.0.0.0:80->80/tcp   drupal8
8cae72ad7ff0        mariadb             "docker-entrypoint.s…"   About an hour ago   Up About an hour    3306/tcp             mariadb
```

查看镜像：

```
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
drupal              latest              4ae9d94d03b6        3 days ago          453MB
mariadb             latest              92495405fc36        12 days ago         356MB
```

### 第三部 配置网站

在宿主机（就是我们当前使用的电脑）的浏览器中输入 localhost 开始进入 Drupal 安装页面。
需要注意的是数据库连接配置：

![](https://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-10-01%E4%B8%8A%E5%8D%8810.16.19.png)

这里的数据库配置参数中，数据库名称、数据库用户名、数据库密码是我们在创建第一个容器时定义的，分别都是`drupal8`，高级选项中的主机名是我们在创建第二个容器时设定的 link 链接名`mysql`。
