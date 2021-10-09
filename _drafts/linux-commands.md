---
layout: post
title: Linux常用指令
categories: Linux
description: Linux中一些常用指令的学习总结
keywords: Linux, Shell, Commands
---

## 查看并切换已安装shell

`cat /etc/shells` 查看已有shell，在笔者的电脑上一共有六种：

```shell
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh

```

`chsh -s /bin/zsh` 更改使用的 shell 种类，要输入开机密码以确认指令生效

## 配置文件

如果当前使用的 shell 是 bash，则写配置文件的命令为 `vi ~/.bashrc`

如果当前使用的 shell 是 zsh，则写配置文件的命令为 `vi ~/.zshrc`

写配置文件之后，若想让配置文件立即生效，需要用 `source` 命令。这里以 zsh 的配置文件命令为例，则生效指令为：`source ~/.zshrc`
