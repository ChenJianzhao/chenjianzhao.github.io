---
categories: Linux
date: 2016-09-07 00:20:00 +0800
status: public
title: Linux下环境变量设置
---

## Linux下环境变量设置的三种方法：
***
如想将一个路径加入到$PATH中，可以像下面这样做：
1) 控制台中设置，不赞成这种方式，因为他只对当前的shell 起作用，换一个shell设置就无效了：
```bash
$PATH="$PATH":/NEW_PATH  (关闭shell Path会还原为原来的path)
```
2) 修改 /etc/profile 文件，如果你的计算机仅仅作为开发使用时推存使用这种方法，因为所有用户的shell都有权使用这个环境变量，可能会给系统带来安全性问题。这里是针对所有的用户的，所有的shell

在最下面添加：
```bash
export  PATH="$PATH:/NEW_PATH"
```
3) 修改 ~/.bashrc文件，这种方法更为安全，它可以把使用这些环境变量的权限控制到用户级别，这里是针对某一特定的用户，如果你需要给某个用户权限使用这些环境变量，你只需要修改其个人用户主目录下的 .bashrc文件就可以了。

在下面添加：
```bash
 export PATH="$PATH:/NEW_PATH"
 ```
 >**推荐第三种方法**