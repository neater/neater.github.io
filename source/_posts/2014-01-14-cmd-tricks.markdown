---
layout: post
title: "CMD Tricks"
date: 2014-01-14 15:47:05 +0800
comments: true
categories: 命令行
---

####删除一个大文件
```
/path/to/file.log (清空)
rm /path/to/file.log
```

####变量CDPATH定义了目录的搜索路径
cd */var/www/html/ 这样长了，我可以直接输入下面的命令进入 /var/www/html

```
export CDPATH=/var/www
cd html
```

#### 输出文件的指定行
```
sed -n 'x,yp' file;   awk 'NF > x && NF > y' file
```
从x行开始，输出y行

```
tail +x file | head -y
```
head和tail 参数 -n的不同：

- head -n -10 除去最后10行
- tail -n +10 从第10行开始
- tail -n 1000：显示最后1000行；
- tail -n +1000：从1000行开始显示，显示1000行以后的
- head -n 1000：显示前面1000行

从第3000行开始，显示1000行。即显示3000~3999行

```
cat filename | tail -n +3000 | head -n 1000
```
显示1000行到3000行

```
cat filename| head -n 3000 | tail -n +1000
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
查看某个选项的当前值

```
:set option?
```

###Readline Key Bindings(man bash中的查找字段)

键盘宏命令

- 开始录制宏 `ctrl + x (`
- 结束录制宏 `ctrl + x )`
- 运行宏 `ctrl + x e`

其他命令

- 显示当前shell的版本信息 `ctrl + x ctrl + v`
- 设置标记 `ctrl + @`
- 交换标记和光标的位置: `ctrl + x ctrl + x`
- 启动编辑器编辑命令，编辑完成后执行	`ctrl + x ctrl + e`
- 撤销前面的操作	`ctrl + x ctrl + u`
- 重新读取配置文件	`ctrl + x ctrl + r`

###Vimperator
y: 复制当前页的url到剪贴板。Y：复制选中文字到剪贴板

"B显示书签列表
:map B :dia bookmarks<cr>