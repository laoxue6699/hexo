---
title: git 帮助文档如何以HTML格式浏览
date: 2019-09-30 17:39:33
tags:
---

我们在命令行使用 git 命令非常方便。但查询 git 帮助文档时，由于命令行窗口的限制，帮助内容比较简略。本文介绍如何以 HTML 格式浏览 git 的帮助文档。

<!-- more -->

如果我们需要查看 git 中关于日志的帮助文档，通常的做法是在命令行窗口输入命令：

```
git help log
```

在命令行窗口会显示出帮助的结果。

如果我们需要将帮助的详细内容通过浏览器显示出来，需要进行配置。

```
# create directory to keep Git documentation html-files
$ sudo mkdir -p /usr/local/git/share/doc # or whatever directory you choose

# change to that directory
$ cd /usr/local/git/share/doc

# clone repo with documentation
$ sudo git clone git://git.kernel.org/pub/scm/git/git-htmldocs.git git-doc

# point your Git explicitly to a new documentation directory
$ git config --global help.htmlpath /usr/local/git/share/doc/git-doc

# tell Git to use html-formatted help by default
$ git config --global help.format html
```

配置完成后，会在 git 配置文件`~/.gitconfig`中创建关于帮助文档的配置。

```
[help]
    format = html
    htmlpath = /usr/local/git/share/doc/git-doc
```

这时候我们再输入：

```

git help log --web

```

或者：

```

git help log

```

在浏览器页面就会看到相关帮助的详细内容了。
![](https://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-30%E4%B8%8B%E5%8D%885.54.09.png)

[参考文档：]()
