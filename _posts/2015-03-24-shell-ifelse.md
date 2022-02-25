---
title: shell之if else判断总结
excerpt: "总结shell的逻辑运算，if else判断语句，以及示例。"
date: 2015-03-24
tags: [shell, linux]
categories: [linux]
render_with_liquid: false
---

## 条件语句
条件语句和是编程语言中的必要要素，shell中使用和使用常见编程语言中的if/else关键字提供双路决策，if/elif/../else提供多路决策。

语法：

```bash
if expression
then
	statements
elif expression
then
	statements
...
else
	statements
fi
#或者expression和then写在同一行的方式
if expression;then
	statements
elif expression;then
	statements
...
else
	statements
fi
```

## 逻辑表达式
shell中常用于判断的表达式有三种：

* test命令
* [ ]命令
* [[ ]]关键字

此外还有let,(())

eg.  
```bash
[root@localhost ~]# test 1 = 1 && echo OK
OK
[root@localhost ~]# [ 1 = 1 ]&& echo OK
OK
[root@localhost ~]# [[ 1 = 1 ]]&& echo OK
OK

#或者写在if语句里
[root@localhost ~]# \
> if test 1 = 1;then
>   echo OK
> fi
OK
```

### 三者区别
1. []和test一样是shell命令，[]里面的条件表达式相当于test的**参数**。
2. [[]]是bash的关键字，老的shell解释器可能不支持。
3. [[]]比[]支持更丰富的表达式。

对于这三者的区别笔者也没有找到更详细准确的文档，总之在不确定结果前先测试一下。

## 逻辑运算符
* = 等于
* == 等于
* ！= 不等于
* \> 大于，[]和test中使用需转义，如[ 1 \> 0 ]
* \>= 大于等于，[]和test中使用需转义
* < 小于，[]和test中使用需转义
* <= 小于等于，[]和test中使用需转义
* -eq 等于，数字比较
* -ne 不等于，数字比较
* -lt 小于，数字比较
* -gt 大于，数字比较
* -le 小于等于，数字比较
* -ge 大于等于，数字比较
* -a 与，[]和test中用
* -o 或，[]和test中用
* && 与，[[]]中用
* \|\| 或，[[]]中用
* ！ 非
* -z 空字符串
* -n 非空字符串

## 文件测试操作

	-b FILE
		FILE exists and is block special
	-c FILE
		FILE exists and is character special
	-d FILE
		FILE exists and is a directory
	-e FILE
		FILE exists
	-f FILE
		FILE exists and is a regular file
	-g FILE
		FILE exists and is set-group-ID
	-G FILE
		FILE exists and is owned by the effective group ID
	-h FILE
		FILE exists and is a symbolic link (same as -L)
	-k FILE
		FILE exists and has its sticky bit set
	-L FILE
		FILE exists and is a symbolic link (same as -h)
	-O FILE
		FILE exists and is owned by the effective user ID
	-p FILE
		FILE exists and is a named pipe
	-r FILE
		FILE exists and read permission is granted
	-s FILE
		FILE exists and has a size greater than zero
	-S FILE
		FILE exists and is a socket
	-t FD  file descriptor FD is opened on a terminal
	-u FILE
		FILE exists and its set-user-ID bit is set
	-w FILE
		FILE exists and write permission is granted
	-x FILE
		FILE exists and execute (or search) permission is granted

更多关于文件判断的信息点击[这里][1]查看。


[1]:http://bash.cyberciti.biz/guide/File_attributes_comparisons