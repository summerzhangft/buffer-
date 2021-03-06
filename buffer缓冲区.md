---
title: Python File Buffer
date: 2017-10-13 11:08:00
tags:
  - python
---

Python 内置的 `open` 函数有一个 `buffering` 参数，用来设置打开文件的缓冲策略(buffer policy)

<!--more-->

## 什么是 Buffer ?

可以将 buffer 理解为 内存数据写入硬盘时 的中间层，用于减少硬盘的 写入次数

例如:

​	有一个 500 byte 的文件，一 byte 一 byte 的写入硬盘，要写 500 次，如果 50  byte 写一次，只需要写十次，无论是对于硬盘(减少写入次数) 还是系统(减少写入时的系统调用) 来说都是极好的事情



## Python 写文件时候如何使用 buffer ?

Python 内置的 open 函数就带有 buffering 参数，用于设置打开文件的 buffer policy

open 函数对文件 buffer 的设置是对系统调用的封装，并不是 Python 本身实现的，而是系统 API 中早已提供的

对于不同参数(-1, 0, 1, > 1)，不同类型的文件 (Binary/Text) 还有不同的策略

| File Type | buffering=-1           | buffering=0 | buffering=1    | buffering>1         |
| --------- | ---------------------- | ----------- | -------------- | ------------------- |
| Binary    | io.DEFAULT_BUFFER_SIZE | unbuffered  | 1              | buffering           |
| Text      | io.DEFAULT_BUFFER_SIZE | 不允许         | line buffering | DEFAULT_BUFFER_SIZE |



| 名称                     | 含义              |
| ---------------------- | --------------- |
| io.DEFAULT_BUFFER_SIZE | 系统默认的 buffer 大小 |
| unbunffered            | 不使用 buffer      |
| line buffering         | 行缓冲             |
| buffering              | 指定缓冲区大小         |



## 实例

不理解 buffer 之前认为 open 函数打开的文件，写入了数据，如果不 close，或者不提前用 flush 是不会刷入硬盘的，实际上这和 buffer policy 有关，如果设置了 line buffering 的策略，则为每写一行数据就会刷入硬盘，如果指定 buffer size，写入大于 buffer size 数据才会刷入硬盘

**line buffering**

```python
In [1]: f = open('line_buffer.txt', 'w', buffering=1) # line buffering 策略写文件

In [2]: cat line_buffer.txt

In [3]: f.write('this is line\n') # 写入一行，没有 flush/close，实际上也刷入硬盘了
Out[3]: 13

In [4]: cat line_buffer.txt # 因为是 ipython，所以可以直接调用 bash 内建命令
this is line

In [5]: f.write('this not a line') # 写入数据，但是没有加换行符，没有刷入硬盘
Out[5]: 15

In [6]: cat line_buffer.txt
this is line

In [7]: f.write('\n') # 写入换行符，刷入了硬盘
Out[7]: 1

In [8]: cat line_buffer.txt
this is line
this not a line
```

**unbunffered**

只有二进制文件才能禁用 buffer

```python
In [10]: f = open('binary_unbuffered.txt', 'wb', buffering=0)

In [11]: cat binary_unbuffered.txt

In [13]: f.write(b'hello') # 写入任意大小的数据，都直接刷入硬盘
Out[13]: 5

In [14]: cat binary_unbuffered.txt
hello
In [15]: f.write(b' world')
Out[15]: 6

In [16]: cat binary_unbuffered.txt
hello world
```

**指定 buffer size**

也只有二进制文件才能指定 buffer size

```python
In [18]: f = open('binary_buffer_size.txt', 'wb', buffering=10) #设置 buffer size 为 10

In [19]: cat binary_buffer_size.txt

In [21]: f.write(b'hello') # 写入 5 byte 数据，由于 buffer 还没有满，所以不会刷入缓冲区
Out[21]: 5

In [22]: cat binary_buffer_size.txt

In [23]: f.write(b'helloworld')
Out[23]: 10

In [24]: cat binary_buffer_size.txt # buffer 满了，刷入缓冲区
hello
In [25]: f.write(b'helloworld111')
Out[25]: 13

In [26]: cat binary_buffer_size.txt
hellohelloworldhelloworld111
```
