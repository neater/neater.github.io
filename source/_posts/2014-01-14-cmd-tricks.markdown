---
layout: post
title: "CMD Tricks"
date: 2014-01-14 15:47:05 +0800
comments: true
categories: 
---
虽然CMD中有很多快捷键，但无法大范围移动光标，OS X 下，按住option，然后鼠标点击当前行，光标会移动到该处。
***
使用pgrep查找进程id，而不用 ps -ef | grep XXX
***
echo $0，当前使用的 shell

输出文件的指定行
sed -n 'x,yp' file;   awk 'NF > x && NF > y' file

从x行开始，输出y行
tail +x file | head -y

head和tail 参数 -n的不同，head -n -10 除去最后10行 tail -n +10 从第10行开始