---
title: 常用Linux命令
date: 2021-01-21 14:12:10
tags: [Linux]
---

# 1.ln -s [源文件] [目标文件]

> * 它的功能是为某一个文件在另外一个位置建立一个同不的链接，这个命令最常用的参数是-s
>
> * ln的链接又分成_软链接_ 和_硬链接_两种，软链接就是ln -s xx xx ,它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间；而硬链接ln xx xx ，即没有参数-s, 它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持`同步变化`



```shell
ln -s /bin/less /usr/local/bin/less
```



# 2.find / -name [文件名]

# 3.more [文件名]

>下翻：空格
>
>上翻：b
>
>查找：/
>
>查找下一个：n



# 4.rpm 软件包管理器

```shell
#安装软件的命令格式
rpm -ivh filename.rpm

#升级软件的命令格式 
rpm -Uvh filename.rpm 

#卸载软件的命令格式 
rpm -e filename.rpm 

#查询软件描述信息的命令格式 
rpm -qpi filename.rpm

#列出软件文件信息的命令格式 
rpm -qpl filename.rpm 

#查询文件属于哪个RPM的命令格式 
rpm -qf filename
```

# 5.jps

> jps是jdk提供的一个查看当前java进程的小工具， 可以看做是JavaVirtual Machine Process Status Tool的缩写。非常简单实用。

  

命令格式：jps [options ] [ hostid ] 

[options]选项 ：
 -q：仅输出VM标识符，不包括classname,jar name,arguments in main method 
 -m：输出main method的参数 
 -l：输出完全的包名，应用主类名，jar的完全路径名 
 -v：输出jvm参数 
 -V：输出通过flag文件传递到JVM中的参数(.hotspotrc文件或-XX:Flags=所指定的文件 
 -Joption：传递参数到vm,例如:-J-Xms512m