---
layout: post
---

[TOC]

## 一些可能有用的命令
1. `xrandr`: 图形模式下查看与修改分辨率

## 查看当前系统支持分辨率
* grub console下使用`vbeinfo`
* linux console下使用`hwinfo --framebuffer`, 如果没有请安装`pacman -S hwinfo`

## 查看当前系统分辨率
* linux console下使用`fbset -i`, 如果没有请安装`pacman -S fbset`

## 设置虚拟机
* 显存`VBoxManage.exe modifyvm 虚拟机名 --vram 256`
* 分辨率`VBoxManage.exe setextradata 虚拟机名 CustomVideoMode1 1600x900x32`

## 设置grub console分辨率
* 编辑`vi /etc/default/grub` 
```
GRUB_GFXMODE="1600x900"
````

## 设置linux console分辨率
* 编辑`vi /etc/default/grub` 
```
GRUB_CMDLINE_LINUX_DEFAULT="quit video=1600x900"
```