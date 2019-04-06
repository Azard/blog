---
title: 修改Linux环境变量
date: 2014-09-08 16:01:22
categories: 技术
tags: [Linux]
---
由于前段时间在开发下学期数据结构课用的作业成绩查询系统，需要在`Linux`下配置`Golang`的一些环境变量，因此记录下探索的Linux环境变量配置的几种方法。
<!-- more -->


* 修改`/etc/environment`
例如
``` bash
PATH="/usr/local/bin":"/usr/local/sbin"
GOPATH="/home/azard/go_workspace"
```
注意，同一个环境变量名的多个值用`:`分割  
不直接使用source命令运行一遍的话都要**重启后**生效


* 修改`/etc/profile`
增加
``` bash
export GOPATH=/home/azard/go_workspace
export PATH=$GOPATH:$PATH
```
可以使用`source /etc/profile`立即生效


* 修改`$Home/.profile`
增加
``` bash
export GOPATH=/home/azard/go_workspace
export PATH=$GOPATH:$PATH
```


* 小结
以上3种方法最终效果一样，但还是有细微的区别。  
系统开机时首先读取 `/etc/environment` 的值，然后再执行 `/etc/profile`，最后执行 `$HOME/.profile`。一个环境变量最终的值是这些执行合并后的最终结果。  
可以使用`echo $GOPATH`查看相应环境变量的值
