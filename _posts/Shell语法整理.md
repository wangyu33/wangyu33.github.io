---
layout: post
title: Shell语法整理
categories: Blog
description: 我的 2018 年全年盘点
keywords: 2018, 计划, 总结
---



Shell语法整理

1. ls

| ls     | 作用                                                        |
| ------ | ----------------------------------------------------------- |
| ls     | 显示不隐藏的文件与文件夹                                    |
| ls -a  | 显示当前目录下的所有文件及文件夹包括隐藏的.和..等           |
| ls -l  | 显示不隐藏的文件与文件夹的详细信息                          |
| ls -al | 显示当前目录下的所有文件及文件夹包括隐藏的.和..等的详细信息 |

-a  a表示all

-l l表示limits，权限

ls -al示意图

![shell](C:\Users\wangyu\Desktop\博客\shell.png)

第一列代表权限，第二列代表连接，第三列代表所有者，第四列代表用户组，第五列代表大小，第六列代表日期，第七列是文件名。



第一列字符串的含义：

-rwxrwxrwx 第1个字符表示类型，2-4表示拥有者权限，5-7表示同用户组权限，8-10表示其他用户组权限。r-read，w-write，x-execute



2. 改变文件属性与权限

```shell
#chgrp:改变文件所属用户组

chgrp [-R] dirname/filrname  #-R表示递归
chgrp mozi 1.txt

#chown 改变文件所有者
```

```shell
#chmod
```

