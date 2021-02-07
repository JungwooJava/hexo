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

>



# 3.rpm 软件包管理器

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

# 4.jps

> jps是jdk提供的一个查看当前java进程的小工具， 可以看做是JavaVirtual Machine Process Status Tool的缩写。非常简单实用。

  

```lua
命令格式：jps [options ] [ hostid ] 

[options]选项 ：
 -q：仅输出VM标识符，不包括classname,jar name,arguments in main method 
 -m：输出main method的参数 
 -l：输出完全的包名，应用主类名，jar的完全路径名 
 -v：输出jvm参数 
 -V：输出通过flag文件传递到JVM中的参数(.hotspotrc文件或-XX:Flags=所指定的文件 
 -Joption：传递参数到vm,例如:-J-Xms512m
```



# 5. 查看日志技巧





> “当日志存储文件很大时，我们就不能用 **vi** 直接去查看日志了，就需要Linux的一些内置命令去查看日志文件.

> **系统Log日志位置：**
>
> **/var/log/message 系统启动后的信息和错误日志， 是 Red Hat Linux 中最常用的日志之一**
>
> **/var/log/secure 与安全相关的日志信息**
>
> **/var/log/maillog 与邮件相关的日志信息**
>
> **/var/log/cron 与定时任务相关的日志信息**
>
> **/var/log/spooler 与UUCP和news设备相关的日志信息**
>
> **/var/log/boot.log 守护进程启动和停止相关的日志消息**
>
> 

### 一、cat命令：

```shell
参数：

-n 或 --number 由 1 开始对所有输出的行数编号
-b 或 --number-nonblank 和 -n 相似，只不过对于空白行不编号
-s 或 --squeeze-blank 当遇到有连续两行以上的空白行，就代换为一行的空白行
-v 或 --show-nonprinting
-E --show-ends 在每行结束处显示 $
-e --等价于-vE

cat主要有三大功能：
1. $ cat filename 一次显示整个文件。
2. $ cat > filename 从键盘创建一个文件。（只能创建新文件,不能编辑已有文件）
3. $ cat filename1 filename2 > filename 将几个文件合并为一个文件（如果原本file文件中有内容，会被覆盖掉）
 
例：
把 file1 的内容加上行号后输入到 file2 这个文件里  
cat -n filename1 > filename2
把 file1 和 file2 的内容加上行号（空白行不加）之后将内容追加到 file3 里
cat -b filename1 filename2 >> filename3  
 
把test.txt文件扔进垃圾箱，赋空值test.txt
cat /dev/null > /etc/test.txt   
注意：>意思是创建，>>是追加。千万不要弄混了。
```

### 二、more命令：

>==more== 命令是一个基于==vi==编辑器文本过滤器，它以全屏幕的方式按页显示文本文件的内容，支持vi中的关键字定位操作。
>
>该命令一次显示一屏文本信息，满屏后停下来，以百分比的形式，以上下翻页，以上下行移动显示查看日志并且在屏幕的底部给出一个提示信息，从开始至当前己显示的该文件的百分比：–More–（XX%） 

#### 按键说明

 按<kbd>Space</kbd>键：显示文本的下一屏内容，即下翻。
 按<kbd>B</kbd>键：显示上一屏内容，即上翻。
 按<kbd>Enter</kbd>键：只显示文本的下一行内容。
 按<kbd>/</kbd>斜线符：接着输入一个模式，可以在文本中寻找下一个相匹配的模式，即查找。
 按<kbd>H</kbd>键：显示帮助屏，该屏上有相关的帮助信息。
 按<kbd>Q</kbd>键：退出more命令。

查找下一个：<kbd>n</kbd>





### 三、less命令：

```
less 命令查看日志，和more命令类似，只不过less支持上下键前后翻阅文件。
```





### 四、head命令：

```
参数：
-q 隐藏文件名
-v 显示文件名
-c 显示字节数
-n 显示的行数
从文本文件的头部开始查看，head 命令用于查看一个文本文件的开头部分。
例：
head filename 或 head -n 10 显示文本文件 file 的前十行内容，然后退出命令
head -n 20 filename 显示文本文件 file 的前二十行内容
head -n -10 filename 显示文本文件除了最后10行的其他所有文本文件信息
```

### 五、tail命令：

```
tail 命令用于显示文本文件的末尾内容（默认10行，相当于增加参数 -n 10），并且实时不断有内容被打印出来，
  若想中断进程，使用命令 Ctrl-C
参数：
tail [ -f ] [ -c Number | -n Number | -m Number | -b Number | -k Number ] [ File ] 
参数解释：
-f 该参数用于监视File文件增长。
-c Number 从 Number 字节位置读取指定文件 
-n Number 从 Number 行位置读取指定文件。
-m Number 从 Number 多字节字符位置读取指定文件，比方你的文件假设包括中文字，假设指定-c参数，可能导致
   截断，但使用-m则会避免该问题。
-b Number 从 Number 表示的512字节块位置读取指定文件。
-k Number 从 Number 表示的1KB块位置读取指定文件。
File 指定操作的目标文件名称 
上述命令中，都涉及到number，假设不指定，默认显示10行。Number前面可使用正负号，表示该偏移从顶部还是从尾
  部开始计算。
tail 可运行文件一般在/usr/bin/以下。
tail -f filename 监视filename文件的尾部内容（默认10行，相当于增加参数 -n 10）
tail -100f filename 监视filename文件的尾部内容（默认从底部往前100行，相当于增加参数 -n 100）
tail -n 20 filename 显示filename最后20行
tail -r -n 10 filename 逆序显示filename最后10行
```

### 六、tac命令：

```
tac (反向查看日志，会打开整个文件，倒序显示，不常用)
tac 是将 cat 反写过来，所以他的功能就跟 cat 相反。
cat 是由第一行到最后一行连续显示在屏幕上，而 tac 则是由最后一行到第一行反向在萤幕上显示出来
```

### 七、echo命令：

```
echo 命令用来在标准输出上显示一段字符
echo [ -n ] 字符串其中选项n表示输出文字后不换行；字符串能加引号，也能不加引号
echo "the echo command test!"
echo "the echo command test!">filename 输出内容到文件
用 echo 命令输出加引号的字符串时，将字符串原样输出
用 echo 命令输出不加引号的字符串时，将字符串中的各个单词作为字符串输出，各字符串之间用一个空格分割
```

### 八、grep命令：

```
grep 同时满足多个关键字和满足任意关键字，是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 
  配的行打印出来。grep全称是Global Regular Expression Print，表示全局正则表达式版本，显示完自动退
  出命令
grep [options]  
参数:  
[options]参数：
-c：只输出匹配行的计数
-I：不区分大 小写(只适用于单字符)
-h：查询多文件时不显示文件名
-l：查询多文件时只输出包含匹配字符的文件名
-n：显示匹配行及 行号
-s：不显示不存在或无匹配文本的错误信息
-v：显示不包含匹配文本的所有行
-A: 显示匹配行及前面多少行, 如: -A3, 则表示显示匹配行及前3行
-B: 显示匹配行及后面多少行, 如: -B3, 则表示显示匹配行及后3行
-C: 显示匹配行前后多少行, 如: -C3, 则表示显示批量行前后3行
pattern正则表达式主要参数：
：忽略正则表达式中特殊字符的原有含义
^：匹配正则表达式的开始行
$: 匹配正则表达式的结束行
<：从匹配正则表达 式的行开始
>：到匹配正则表达式的行结束
[ ]：单个字符，如[A]即A符合要求 
[ - ]：范围，如[A-Z]，即A、B、C一直到Z都符合要求 
。：所有的单个字符
- ：有字符，长度可以为0


例
grep -n "word" filename 查看文件包含条件的日志，全部显示出来（单引号或者双引号都可以，不区分）
grep -E "word1|word2|word3" filename 满足任意条件（word1、word2和word3之一）将匹配的内容全部打印出来

grep word1 filename | grep word2 |grep word3 必须同时满足三个条件（word1、word2和word3）才匹配多管道，多次筛选

使用正则表达式 -E 选项
grep -E "[1-9]+" 或 egrep "[1-9]+"
grep -A100 'word' filename 显示匹配行往后100行
grep -B100 'word' filename 显示匹配行往前100行
grep -C100 'word' filename 显示匹配行往前往后100行
```

### 九、sed命令：

> sed 本身是一个管道命令，主要是以行为单位进行处理，可以将数据行进行替换、删除、新增、选取等特定工作。

```shell

参数


-n∶ 使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的资料一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过sed特殊处理的那一行(或者动作)才会被列出来。

-p ∶列印，亦即将某个选择的资料印出。通常 p 会与参数 sed -n 一起运作

-s ∶取代，可以直接进行取代的工作,通常这个 s 的动作可以搭配正规表示法
例如1 sed s/old/new/g
例如2
sed -n '5,10p' filename 只查看文件的第5行到第10行
sed -n '/2019-01-04 21:30:00/,/2019-01-04 22:30:30/p' filename 只查看文件包含时间段的区间内容
```

### 十、混合命令：



```shell
tail -n +92表示查询92行之后的日志
tail filename -n 300 -f 查看底部即最新300条日志记录，并实时刷新
tail -f filename | grep -E 'word1|word2|word3' 实时打印出匹配规则的文件内容（注意或符号前后最好
  不要有空格）
cat -n filename |grep “地形” | more 得到关键日志的行号
cat -n filename |tail -n +92|head -n 20
grep 'nick' | tail filename -C 10 查看字符‘nick’前后10条日志记录, 大写C
head -n 20 则表示在前面的查询结果里再查前20条记录
```

### 十一、附加：

```shell
vi filename 查看或编辑文件

查找文件内容关键字方法：
先执行命令>： vi filename
然后输入>:    /查找字符串 
按n查找下一个

例如1 查找nohup.out日志文件的error关键字
执行命令：vi  nohup.out
输入以下回车：/error
按n查找下一个

例如2 将实时日志打印到指定文件
将实时日志打印到文件newlog.log内，方便查找
执行命令：tail  -f  nohup.out   >newlog.log
备注：newlog.log文件可以不存在，命令执行时会自动新建
```

