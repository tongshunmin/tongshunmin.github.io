---
layout: post
title: 如何从dll文件导出对应的lib文件?
category: Windows
tags: Windows
keywords: dll,lib
description: 
---
## 引言

近期由于不再使用vs生成lib，考虑使用windows下gcc生成一个动态库，供第三方调用，发现编译之后只有dll，lib如何处理？
好吧，这就是本文的目的。

### 由dll导出lib

Visual C++ 开发工具提供了两个命令行工具，一个是dumpbin.exe,另一个是lib.exe。利用这两个工具即可从dll导出其对应的lib。

1、导出def文件
在命令行执行：
dumpbin /exports target.dll > target.def

2、编辑target.def 文件，使之格式与.def文件格式一致 。比如：
EXPORTS
fn @1
fn @2

3、导出lib文件
在命令行执行
````
lib /def:target.def /machine:i386 /out:target.lib
````

## 实践
用下面的源码验证下，保存为export.c

````c
/* 
 * compile with gcc 4.8.1
 * gcc export.c -c -o test.exe
 * gcc -Wall -shared export.c -o export.dll
 * 
*/

#include <stdlib.h>

const char *  get_version_string()
{
    return "tocy-gcc-dll-test-interface";
}

int select_mode(int mode)
{
    return mode;
}
````
gcc编译后生成export.dll，执行上面的第一步，输出的def格式如下：
````
Microsoft (R) COFF/PE Dumper Version 10.00.40219.01
Copyright (C) Microsoft Corporation.  All rights reserved.

Dump of file export.dll

File Type: DLL

  Section contains the following exports for export.dll

    00000000 characteristics
    5732C473 time date stamp Wed May 11 13:34:43 2016
        0.00 version
           1 ordinal base
           2 number of functions
           2 number of names

    ordinal hint RVA      name

          1    0 00001280 get_version_string
          2    1 0000128A select_mode

  Summary

        1000 .CRT
        1000 .bss
        1000 .data
        1000 .edata
        1000 .eh_fram
        1000 .idata
        1000 .rdata
        1000 .reloc
        1000 .text
        1000 .tls
````        
将其修改为标准的def文件格式，如下：
````
LIBRARY export

EXPORTS
get_version_string  @1
select_mode         @2
执行第三步就可以导出lib。
````

##  最后
关于dll和so的区别
windows下动态库和linux下动态库的区别。
基本机制差不多，比较奇怪的是为什么linux下的动态库*.so隐式链接的时候不需要符号表，而windows下的动态库*.dll却需要呢？
其实从文件格式上来说就知道，dll是PE格式（Portable Executable）；而so是ELF格式（Executable and Linkable Format）。顾名思义，就是由于文件格式确定的。有兴趣可以深入研究下二者的不同。

感谢收看，如果对大家有帮助，请[github上follow和star](https://github.com/tongshunmin)，本文发布在[佟顺民的技术博客](http://blog.mineki.cn/)，转载请注明出处

##  参考
使用 DEF 文件从 DLL 导出
[Module-Definition (.Def) Files](http://my.oschina.net/hondfy/blog/165675)


