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

***
使用pgrep查找进程id，而不用 ps -ef | grep XXX
***
echo $0，当前使用的 shell

vim打开剪贴板内容
```
alias vipb='xclip -selection clipboard -o | vim -' // Linux
alias vipb="pbpaste | vim -" 	// Mac
```
显示sqlite文件内容
```
sqlite3 -line database.db 
```
修改SSH的RSA密码短语
```
ssh-keygen -f ~/.ssh/id_rsa -p 修改SSH的RSA密码短语
```
man -w/--path 
```
打印相关帮助文档的位置
```
使用-i参数默认的前面输出用{}代替，-I参数可以指定其他代替字符，如例子中的[] 

```
find . -name "file" | xargs -I [] cp []
```