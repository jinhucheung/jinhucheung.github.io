---
title: 一次 Windows 批处理脚本编写经历
date: 2020-03-21 18:02:39
tags: [Windows, Batch, Script]
categories: [运维]
---

前段时间遇到一个需求，需要在 Windows 平台上做文件的 MD5 校验。由于之前已有 Linux 的 MD5 校验脚本，故需在 Windows 上也实现同样的功能，然后就有了这次 Windows 批处理脚本编写的经历。

<!--more-->

## 介绍

Windows 批处理 (Batch) 通常被认为是一种脚本语言，它应用于 DOS 和 Windows 系统中，由系统内置的解析器 (COMMAND.COM 或 CMD.EXE) 解析运行。它类似 Unix 中的 Shell 脚本，具有 .bat 或 .cmd 的扩展名。

## 描述需求

上面提到需要在 Windows 平台实现文件 MD5 校验工具。考虑需要运行在不同的 Windows 环境上，所以决定用 Windows 内置解析的批处理脚本实现。而文件 MD5 校验基本流程如下：

1. 获取用户输入的文件
2. 检查是否输入文件的 MD5 值文件
3. 获取输入文件的 MD5 值
4. 从 MD5 值文件获取相应文件的值
5. 对比上两步的 MD5 值

## 实现需求

### 了解批处理脚本

在编写批处理脚本前，需要先了解批处理的基本命令、语法及一些内置变量等。


下面是批处理的基本命令：

| 命令  | 描述                                       | 示例                                 |
| ----- | ------------------------------------------ | ------------------------------------ |
| echo  | 打印                                       | echo Press Enter                     |
| rem   | 注释                                       | REM 获取文件值                       |
| pause | 暂停                                       | pause > null                         |
| set   | 设置变量                                   | set "sha="                           |
| call  | 调用另一个批处理程序，并不终止父批处理程序 | call callchild.bat                   |
| start | 调用外部批处理程序，非同一直进程           | start startchild.bat                 |
| for   | 循环命令                                   | FOR %%parameter IN (set) DO command  |
| if    | 判断命令                                   | IF condtion (command) ELSE (command) |

而批处理变量使用 `%$name%` 或者 `!$name!` 引用变量，如 `%file%` 引用定义好的 file 变量。其中 `%$name%` 与 `!$name!` 的区别在于，`!$name!` 会延迟处理这个变量，而 `%$name%` 则在命令解释时将用具体值替换该变量。

下面是一些预设的变量：

| 变量     | 描述                            |
| -------- | ------------------------------- |
| %CD%     | 当前目录字符串                  |
| %DATE%   | 当前日期                        |
| %TIME%   | 当前时间                        |
| %RANDOM% | 0 和 32767 之间的任意十进制数字 |
| %0       | 批处理脚本所在的文件路径        |
| %1       | 批处理脚本所传入的参数1..n      |
| %path%   | 当前环境变量                    |

下面是一些扩展变量，假设当前批处理脚本为 `C:\Documents\script\check.bat`

| 变量  | 描述                       | 取值                          |
| ----- | -------------------------- | ----------------------------- |
| %0    | 批处理脚本所在的文件路径   | C:\Documents\script\check.bat |
| %~dp0 | 批处理脚本所在的目录       | C:\Documents\script\          |
| %~nx0 | 批处理脚本文件名(含扩展)   | check.bat                     |
| %~n0  | 批处理脚本文件名(不含扩展) | check                         |
| %~x0  | 批处理脚本文件扩展名       | .bat                          |

### 编写批处理脚本

在了解完批处理的基本命令后，可以编写批处理脚本实现上面的需求了。

文中实现的代码可在 [jinhucheung/shell-script-sample](https://github.com/jinhucheung/shell-script-sample/blob/master/md5_check_bat/check_change.bat) 中获取。

首先需要一个工具编写脚本文本，Linux 上需要注意换行符、编码等与 Windows 平台不同。而 Windows 上可以用文本工具另存为 bat 即可。

然后先关闭命令回显，并设置变量延迟处理：

```bat
@echo off
setlocal enableDelayedExpansion
```

之后进入批处理脚本目录，等待用户输入校验的文件名：

```bat
cd %~dp0
set /p file="Please enter checking file name, e.g. attachment.zip: "
```

需要注意文件名可能存在空格，而批处理接收到这类文件名是带引号的，需要将引号过滤：

```bat
set file=%file:"=%
```

获取输入文件对应的 MD5 值文件：

```bat
set "md5_file_name=md5_!file!.txt"
for /r "%~dp0" %%a in (*) do if "%%~nxa"=="!md5_file_name!" set md5_file_path="%%~dpnxa"
if defined md5_file_path (
  echo !md5_file_path! is found.
) else (
  echo It can not check because !md5_file_name! is not found.
  echo Press Enter for exiting
  pause > nul
  exit
)
```

使用 certutil 工具获取输入文件的 MD5 值：

```bat
set "check_file_sha="
for /f "skip=1 tokens=* delims=" %%# in ('certutil -hashfile "!file!" MD5') do (
	if not defined check_file_sha (
		for %%Z in (%%#) do set "check_file_sha=!check_file_sha!%%Z"
	)
)
```

获取 MD5 值文件中的 MD5 值：

```bat
set "valid_file_sha="
for /f "tokens=1 delims= " %%i in ('findstr "!file!" !md5_file_path!') do set "valid_file_sha=!valid_file_sha!%%i"
```

校验输入文件与 MD5 值文件的 MD5 值是否一致：

```bat
if "!valid_file_sha!" == "" (
  echo It's Invalid. the !md5_file_name! is not found or sha of !file! is no existed!!
) else (
  if "!valid_file_sha!" == "!check_file_sha!" (
    echo It's Ok.
  ) else (
    echo It's Invalid. current file sha is not matched for valid file sha!!
  )
)
```

最后暂停脚本，已查看结果：

```bat
pause > nul
```

## 参考

1. [Windows CMD commands](https://ss64.com/nt)
2. [Windows批处理命令](https://www.jianshu.com/p/97cfbb5f2baf)
3. [批处理中setlocal enabledelayedexpansion的作用详细整理](https://www.jb51.net/article/29323.htm)