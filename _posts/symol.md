
---
title: iOS app引用static libraries的symbol分析
date: 2017-04-01
---

优化App的缩小题体积的时候，需要分析Mach-O 可执行文件各个库所占的大小，生成LiveSdk-LinkMap-normal-x86_64.txt(具体生产过程查看参考链接)文件，然后查看symbol符号表，最后统计出各个库的在可执行文件中所占的大小。

##  linkmap文本（解析的对象部分文件截取）  
Symbols:  
Address	Size    	File  Name  
0x100001D60	0x000000B2	[  2] -[JXLUImageEmptyFilter init]  
0x100001E20	0x00000330	[  3] -[H264HwEncoderFileDelegate initWithUrl:]  
0x100002150	0x000001B0	[  3] -[H264HwEncoderFileDelegate

## 脚本

`#!/bin/bash`

`#用于统计静态库在可执行文件中的大小（用于普通LiveSdk-LinkMap文件）`
`#参数说明：参数1：file_name.txt文件的文件路径`

`file_name=$1`


`# 条件 正则／计算／输出文件`
`# 例子`
`# awk 'BEGIN{ sum=0; print "总和：" } { print $1"+"; sum+=$1 } END{ print "等于"; print sum }'`
`# ^[[\s]*[0-9]{1,3}]$ 正则表达式查找比如   [ 21]`
`# --non-decimal-data  16进制相加`

`# GPUImageFilter [ 19]-[ 28] ffmpeg [ 29]-[166] libx264 [200]-[243] libkronos [167]-[199]`

`awk --non-decimal-data '/\[ *[0-9]{1,3}\]/{ if($2~/^0x/)sum+=$2;} END{ printf("total   0x%x \n", sum) }' >total.txt  $file_name`

`awk --non-decimal-data '/\[( 19| 2[0-8])\]/{ if($2~/^0x/)sum+=$2} END{ printf("GPUImageFilter 0x%x \n", sum) }' >>total.txt  $file_name`

`awk --non-decimal-data '/\[( 29| [3-9][0-9]|1[0-5][0-9]|16[0-6])\]/{ if($2~/^0x/)sum+=$2} END{ printf("ffmpeg 0x%x \n", sum) }' >>total.txt  $file_name`

`awk --non-decimal-data '/\[(2[0-3][0-9]|24[0-3])\]/{ if($2~/^0x/)sum+=$2} END{ printf("libx264 0x%x \n", sum) }' >>total.txt  $file_name`

`awk --non-decimal-data '/\[(16[7-9]|1[7-9][0-9])\]/{ if($2~/^0x/)sum+=$2} END{ printf("libkronos 0x%x \n", sum) }' >>total.txt  $file_name`

## 脚本说明：
awk表达式 用正则表达式过滤 符合条件的size列做累加，最后得出结果输出到文本
其中\[ *[0-9]{1,3}\]的空格用\s替换会有些问题

##  运行结果:    
total 0x4d5bf4  
GPUImageFilter 0x243bc  
ffmpeg 0x2de589   
libx264 0x177de7  
libkronos 0x241c7  
运行结果表示各个库在可执行文件中占的大小 结果用十六进制表示

##  参考地址：  
<http://blog.cnbang.net/tech/2296/>  
<http://wiki.jikexueyuan.com/project/objc/Build-tool/6-3.html>  
<http://man.linuxde.net/awk>
    
	

 
