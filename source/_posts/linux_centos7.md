---
title: Linux_centos7  知识点
date: 2024-10-10 20:33:28
layout: post
categories: [Linux_centos7 知识点整理]
tags: [linux]
comments: true
cover: /img/linux_logo.png
description: Linux_centos7 整理知识点
---



# 1. Linux 的应用领域

- 企业环境的利用：最热门的当然是网络服务器；关键任务的应用（金融数据库、大型企业网管环境）；学术机构的高效能运算任务。
- 个人环境的使用：桌面计算机；手机（安装其实就是Linux核心的一支）；嵌入式系统（操作系统直接嵌入在产品上，如一些手机，数字相机，包括router，switch，Firewalls）。
- 云端运用：云程序（底层就是 Linux，云程序搭建出来的虚拟机也是 Linux）；端设备。



# 2. 基础知识

## 目录配置和含义

- /bin：存放二进制可执行的文件

- /boot：存放系统引导时使用的各种文件

- /dev：存放设备文件

- /etc：存放系统配置文件

- /home：存放系统用户的文件

- /lib：存放程序运行所需的共享库和内核模块

- /opt：额外安装的可选应用包所放置的位置

- /(root, 根目录)：与开机系统有关，超级用户目录；

- /sbin：存放二进制可执行文件，只有root用户才能访问

- /tmp：存放临时文件

- /usr(unix software resource)： 与软件安装 / 执行有关，存放系统应用程序；

- /var(variable)：与系统允许过程有关，存放运行时需要改变数据的文件，例如日志文件。

  



# 3. 常见的操作命令

![](/img/0009-linux常见指令总结.jpg )

> 不会就 ----->   **help**
>
> 也可以 **man**  然后输入需要想要的指令

> 查看所有进程：**ps -ef**
>
> 查看端口占用情况：**netstat -ntlp**
>
> 杀死进程：**kill -QUIT 进程号** || **kill -9 进程号**

## 磁盘文件与文件系统管理

- df 和 du
  - df ：列出文件系统的整体磁盘使用量
  - du：评估文件系统的磁盘使用量（常用在推估目录所占容量）

## 防火墙

- 检查防火墙状态：sudo systemctl status firewalld
- 停止防火墙服务：sudo systemctl stop firewalld
- 禁止防火墙开机启动：sudo systemctl disable firewalld
- 验证防火墙状态：sudo systemctl status firewalld

> 只开放防火墙指定的端口：
>
> **sudo firewall-cmd --permanent --add-port=80/tcp**
>
> **sudo firewall-cmd --reload**



## 目录文件相关

> **.**    代表此层目录
>
> **..**    代表上一层目录
>
> **-**     代表前一个工作目录
>
> **~**    代表【目前用户身份】所在的家目录
>
> **~account**   代表 account 这个用户的家目录（account 是个账号名称）

-  cd：变换目录
- pwd：显示当前目录
- touch：创建文件
-  mkdir：建立个新的目录
   - 语法：mkdir [-p] dirName
   - 说明：-p ：确保目录名称存在，不存在的就创建一个。
-  rmdir：删除一个空的目录
   - 语法：rmdir [-p] dirName
   -  说明：-p ： 当子目录被删除后使父目录为空目录的话，则一并删除
      - rmdir itcast  删除名为 itcast 的空目录
      - rmdir -p itcast/test 删除 itcast 目录中名为 test 的子目录
      - rmdir itcast* 删除名称以 itcast 开始的空目录
-  cp：可以复制文件和目录 （复制目录可以用参数-r ，但是权限可能会变，一般用-a就好了）
   - 语法： cp [-r] source dest
   - 说明：-r ：如果复制的目录需要使用此选项，此时将复制该目录下所有的子目录和文件
   -  举例：
      - cp hello.txt itcast/         将hello.txt复制到itcast目录中
      - cp hello.txt ./hi.txt         将hello.txt复制到当前目录，并改名为hi.txt
      - cp -r itcast/ ./itheima/    将itcast目录和目录下所有文件复制到itheima目录下
      - cp -r itcast/* ./itheima/   将itcast目录下所有文件复制到itheima目录下
-  rm：移除文件或目录（移除目录加 -f 强制，-r是递归删除 （**`rm -rf 这个命令慎用`**））
   - 语法：rm [-rf] name
   -  说明：
      - -r：将目录及目录中所有文件（目录）逐一删除，即递归删除
      - -f：无需确认，直接删除
   -  举例：
      -  rm -r itcast/      删除名为 itcast 的目录和目录中所有文件，删除就需确认
      -  rm -rf itcast/    无需确认，直接删除名为 itcast 的目录和目录中所有文件
      -  rm -f hello.txt   无需确认，直接删除 hello.txt 文件
-  mv：（移动文件与目录，或改名）就相当于剪切
   -  语法：mv source dest
   -  举例：
      -  mv hello.txt hi.txt         将 hello.txt 改名为 hi.txt
      -  mv hi.txt itheima/         将文件 hi.txt 移动到itheima目录中
      -  mv hi.txt itheima/hello.txt       将hi.txt移动到itheima目录中，并改名为hello.txt
      -  mv itcast/ itheima/       如果 itheima 目录不存在，将 itcast 目录改名为 itheima
      -  mv itcast/ itheima/       如果 itheima 目录存在，将 itcast 目录移动到 itheima 目录
-  tail： 查看文件末尾的内容
   - 语法： tail [-f] fileName
   -  说明：-f ：动态读取文件末尾内容并显示，通常用于日志文件的内容输出。
   -  举例：
      -  tail  fileName ：显示 文件末尾 10 行的内容
      -  tail -20 fileName：显示文件末尾 20 行内容
      -  tail -f fileName：动态读取 文件末尾内容并显示
-  cat：查看文件内容
-  find ：在指定的目录下查找文件
   -  语法：find dirName -option fileName
   -  举例：
      -  find   .    -name  "*.java"     在当前目录及其子目录下查找 .java 结尾文件
      -  find   /itcast  -name  "*.java"    在 /itcast 目录及其子目录下查找 .java 结尾的文件
-  grep：从指定文件中查找指定的文本内容
   -  语法：grep word fileName
   -  举例：
      -  grep Hello HelloWorld.java    查找HelloWorld.java文件中 Hello 文本
      -  grep hello *.java      查找当前目录中所有 .java  文件中的 hello 文本

## 文件相关，vim编辑器

### 压缩指令

> 常见的压缩文件扩展名（**`常用的gzip、bzip和最新的xz`**）：
>
> - *.Z          compress 程序压缩的文件
> - *.zip       zip 程序压缩的文件
> - *.gz        gzip 程序压缩的文件
> - *.bz2      bzip2 程序压缩的文件
> - *.xz         xz 程序压缩的文件
> - *.tar        tar 程序打包的数据，并没有压缩过：|
> - *.tar.gz    tar 程序打包的文件，其中并且经过 gzip 的压缩
> - *.tar.bz2  tar 程序打包的文件，其中并且经过 bzip2 的压缩
> - *.tar.xz     tar 程序打包的文件，其中并且经过 xz 的压缩

- gzip这个压缩指令应用比较广:(1~9的压缩等级，越大压缩比越好，默认6）

```bash
[root@agent tmp]$ cd /tmp
#du查看文件大小单位（k）
[root@agent tmp]$ du /etc/services 
656	/etc/services
 
[root@agent tmp]$ cp /etc/services .
#压缩.
[root@agent tmp]$ gzip -v services 
services:	 79.7% -- replaced with services.gz
[root@agent tmp]$ ll /etc/services /tmp/services.gz 
-rw-r--r--. 1 root root 670457 Jul 31  2019 /etc/services
-rw-r--r--. 1 root root 136133 Mar 30 04:57 /tmp/services.gz
#查看压缩
[root@agent tmp]$ zcat services.gz 
[root@agent tmp]$ zmore services.gz 
[root@agent tmp]$ zless services.gz 
#过滤压缩文件中http
[root@agent tmp]$ zgrep -n 'http' services.gz 
 
#解压缩，但会将gz压缩包删掉
[root@agent tmp]$ gzip -d services.gz 
 
#以最好的压缩比，并保留原来文件
[root@agent tmp]$ gzip -9 -c services >services.gz
```

- 同样的，bzip2，bzcat/bzmore/bzless/bzgrep；这个压缩效果更好，都差不多。

- xz，xzcat/xzmore/xzless/xzgrep。这个压缩比更好，用法差不多

### 打包指令

![](/img/0010-linux_tar_1.jpg )

![](/img/0011-linux_tar_简记.jpg )

要懂得经常备份/etc下的东西，打包压缩是个很好的方式

```bash
#检查压缩/etc 到root下所需的时间 很明显第一个花时间短，花时间长说明压缩比好
[root@agent tmp]$ time tar -zcvf /root/etc.tar.gz /etc
[root@agent tmp]$ time tar -jcvf /root/etc.tar.bz /etc
[root@agent tmp]$ time tar -Jcvf /root/etc.tar.xz /etc
```

### vim 编辑器

vim可以看成vi的进阶版，可以有颜色，可以判断文本格式，很好用。（三种模式及互相切换方式如图）

![](/img/0012-vim.png )

```bash
#wget是下载网络是上文档的命令，这个文件是鸟哥给的一个host文件
[root@agent tmp]$ wget http://linux.vbird.org/linux_basic/0310vi/hosts
[root@agent tmp]$ vim hosts
```

基本操作：（区块选择要熟悉，挺好使的）

- i 在光标前插入；o 在下行插入，（进入插入模式）

- 按ESC键可以退出到命令行模式：在这边，u 撤销、CTRL+r 恢复、y 是复制，p 是粘贴，gg 到首行，G 到最后一列

- x：删除光标所在的字符；dd 删除当前行；想删除全部  gg—>V—>G，然后d删除

- 在命令行模式下输入：（冒号），然后输入【（wq  保存退出）、q!  强制退出，不保存）、 w（保存）、  q（离开）】

- :/string快速定位；   :1,$ s/str1/str2/g  用字符串 str2 替换正文中所有出现的字符串 str1

![](/img/0013-vim_vim.png )



## shell

```shell
#查看shell环境变量
[user1@agent ~]$ env
#查看环境变量及自定义变量
[user1@agent ~]$ set
#将自定义变量转成环境变量
[user1@agent ~]$ export name
```

```shell
[user1@agent ~]$ alias lm='rm -i'
[user1@agent ~]$ alias |grep lm
alias lm='rm -i'
[user1@agent ~]$ unalias lm
[user1@agent ~]$ alias |grep lm
 
#history查看历史命令，可以尝试下
[user1@agent ~]$ alias h='history'
[user1@agent ~]$ h
 
#source命令：读入环境配置文件的指令
```

![](/img/0014-shell.jpg )



## 系统服务（daemons）

![](/img/0015-系统服务.png )



## yum

```bash
#列出yum服务器上所提供的所有软件名称，
[root@www packages]# yum list
#已经安装的软件
Installed Packages
GConf2.x86_64                             3.2.6-8.el7                  @anaconda
GeoIP.x86_64                              1.5.0-13.el7                 @anaconda
ModemManager.x86_64                       1.6.10-1.el7                 @anaconda
#还可以安装的软件
Available Packages
0ad.x86_64                                0.0.22-1.el7                 epel     
0ad-data.noarch                           0.0.22-1.el7                 epel     
0install.x86_64 
#注意利用通配符*（记不清软件名字的情况下）
[root@www packages]# yum list  pam*                   
 
#列出目前服务器可供本机升级的软件
[root@www packages]# yum list updates
 
#搜寻磁盘阵列（raid）相关的软件
root@www packages]# yum search raid
 
#找出mdadm这个软件的功能，查出的信息要会看哦 不会的单词百度哦
[root@www packages]# yum info mdadm
 
Installed Packages
Name        : mdadm
Arch        : x86_64
Version     : 4.1
Release     : rc1_2.el7
Size        : 1.0 M
Repo        : installed
From repo   : anaconda
Summary     : The mdadm program controls Linux md devices (software RAID arrays)
URL         : http://www.kernel.org/pub/linux/utils/raid/mdadm/
License     : GPLv2+
Description : The mdadm program is used to create, manage, and monitor Linux MD (software
            : RAID) devices.  As such, it provides similar functionality to the raidtools
            : package.  However, mdadm is a single program, and it can perform
            : almost all functions without a configuration file, though a configuration
            : file can be used to help with some common tasks.
 
#找出提供passwd这个文件的软件有哪些，这样就可以针对性安装软件啦
[root@www packages]# yum provides passwd
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
passwd-0.79-5.el7.x86_64 : An utility for setting or changing passwords using PAM
Repo        : base
```

```bash
#可以不用加参数，这边加参数的话，下载默认就是yes了。
[root@www packages]# yum install pam-devel -y
[root@www packages]# yum update pam-devel
[root@www packages]# yum remove pam-devel
#假如要全系统升级呢
[root@www ~]# yum -y update
```



## 其余

- 查看目录内容：**ll**  或 **ls**

  - **`ls`**：基础命令，用于列出目录内容，可以通过添加选项来改变输出格式。
  - **`ll`**：`ls -l` 的别名，用于以长格式列出文件详情。

  > #### 常用选项：
  >
  > - `-a`：显示所有文件，包括隐藏文件（以`.`开头的文件）。
  > - `-l`：以长格式（long listing format）显示文件详情，包括权限、链接数、属主、属组、大小、时间戳等。
  > - `-h`：以人类可读的形式显示文件大小（例如，1K、234M、2G等）。
  > - `-R`：递归地列出子目录中的文件。
  > - `-t`：按时间排序文件。

- 语言系统
  - 查看当前支持的语系：**local**
  - 修改：**LANG=en_US.utf8**
  - 第三个要更新LC_ALL： **export LC_ALL=en_US.utf8**
- 显示日期 |  日历  | 计算机：**date** |  **cal**  |  **bc**
- 正常关机（有的系统需要切换 root 才可以执行，用命令： **su -**  ）
  - 将数据同步写入硬盘中的指令： **sync** （建议多执行几次）
  - 惯用的关机指令： **shutdown** | **init 0** | **systemctl [shutdown -参数]**
  - 重新启动，关机：**reboot**  |  **halt**   | **poweroff**s

```bash
#建议这样多保存再关机
sync;sync;sync;reboot
```

- 环境变量

```bash
#查看shell环境变量
[user1@agent ~]$ env
#查看环境变量及自定义变量
[user1@agent ~]$ set
#将自定义变量转成环境变量
[user1@agent ~]$ export name
```







------



# 4. Linux 常见问题

## Linux 无法通过curl获得服务器主页数据如何排查？

- 关闭防火墙、看host文件中是否 ip 和域名绑定



## 两个同网段Linux服务器在不安装客户端情况下如何传递文件？

- scp 命令



## Linux 命令

1. 查看文本文件头部 n 行

- head -n 200 filename  // 200 可替换为任一数字

2. 产看文本文件末尾 n 行

- tail -n 200 filename // 200 可替换为任一数字

3. 查看文本文件行数

- wc -l filename

