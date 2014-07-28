---
layout: post
title: "CMD Tricks"
date: 2014-01-14 15:47:05 +0800
comments: true
categories: 命令行
---



#### 输出文件的指定行
```
sed -n 'x,yp' file;   awk 'NF > x && NF > y' file
```
从x行开始，输出y行
```
tail +x file | head -y

head和tail 参数 -n的不同，head -n -10 除去最后10行 tail -n +10 从第10行开始
```

#### Mac下获取UDID
```
ioreg -w 0 -rc IOUSBDevice -k SupportsIPhoneOS | sed -n 's/.*USB Serial Number[^0-9a-z]*\([0-9a-z]*\).*/\1/p'

system_profiler SPUSBDataType | sed -n -e '/iPhone/,/Serial/p' | grep "Serial Number:" | awk -F ": " '{print $2}'

system_profiler SPUSBDataType | grep "Serial Number:.*" | sed s#".*Serial Number: "##
```
#### 清空文件
```
> filename

cat /dev/null > filename
```
给机器增加Load
```
cat /dev/nll/unandom | gzip -9 > /dev/null & 
```
虽然CMD中有很多快捷键，但无法大范围移动光标，OS X 下，按住option，然后鼠标点击当前行，光标会移动到该处。
***
使用pgrep查找进程id，而不用 ps -ef | grep XXX
***
echo $0，当前使用的 shell