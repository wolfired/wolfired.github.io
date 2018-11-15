---
layout: post
---

<!-- TOC -->

- [Download Items and Attributes](#download-items-and-attributes)
    - [d.* 命令](#d-命令)
    - [f.* 命令](#f-命令)
    - [p.* 命令](#p-命令)
    - [t.* 命令](#t-命令)
    - [load.* 命令](#load-命令)
    - [session.* 命令](#session-命令)
- [Scripting](#scripting)
    - [method.* 命令](#method-命令)
    - [event.* 命令](#event-命令)
    - [Scheduling 命令](#scheduling-命令)
    - [Importing Script Files](#importing-script-files)
    - [Conditional Operators](#conditional-operators)
    - [String Functions](#string-functions)
    - [Array Functions](#array-functions)
    - [Math Functions](#math-functions)
    - [Value Conversion & Formatting](#value-conversion--formatting)

<!-- /TOC -->

# Download Items and Attributes

## d.* 命令

1. 显示全部任务名称

`d.multicall2=,print=(d.name)`

2. 启动全部任务

`d.multicall2=,d.start=`

3. 停止全部任务

`d.multicall2=,d.stop=`

4. 打开全部任务

`d.multicall2=,d.open=`

5. 关闭全部任务

`d.multicall2=,d.close=`

6. 暂停全部任务

`d.multicall2=,d.pause=`

7. 继续全部任务

`d.multicall2=,d.resume=`

8. 显示全部任务在rtorrent的本次运行中由哪个种子文件加载, 首次加载的任务显示为源种子文件, 后续重启rtorrent后显示为session文件夹里的种子文件

`d.multicall2=,print=(d.loaded_file)`

9. 显示全部任务的初始种子文件

`d.multicall2=,print=(d.tied_to_file)`

10. 显示全部任务的完成状态, 0=未完成, 1=已完成

`d.multicall2=,print=(d.complete)`

11. 显示全部任务的完成状态, 1=未完成, 0=已完成

`d.multicall2=,print=(d.incomplete)`

## f.* 命令

## p.* 命令

## t.* 命令

## load.* 命令

## session.* 命令

1. 显示会话名

`print=(session.name)`

2. 命名会话

`session.name.set="custom session name"`

3. 显示会话状态, 0 or 1

`print=(session.on_completion)`

4. 修改会话状态

`session.on_completion.set=0`

5. 显示会话保存路径

`print=(session.path)`

6. 修改会话保存路径, 禁止在rtorrent运行中修改, 也就是说只能通过.rtorrent.rc设置

`session.path.set="/path/to/new/session"`

7. 立即保存会话

`session.save=`

8. 显示会话用户锁, 0=未锁, 1=已锁

`print=(session.use_lock)`

9. 修改会话用户锁

`session.use_lock.set=0`

# Scripting

## method.* 命令

1. 定义新命令的通用方式

`method.insert=命令的名称,类型[|子类型...],命令的定义`

2. 定义一个新的`simple`命令

`method.insert.c_simple=命令的名称,命令的定义`

3. 定义一个新的`static const simple`命令

`method.insert.s_c_simple =命令的名称,命令的定义`

## event.* 命令

## Scheduling 命令

## Importing Script Files

## Conditional Operators

## String Functions

1. 连接并输出字符串

`print=(cat, hello\ world!\ , {"My", " name", \ , "is", " LinkWu."})`

## Array Functions

## Math Functions

## Value Conversion & Formatting
