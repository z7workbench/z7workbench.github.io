---
layout:     article
title:      "Linux Note / Linux折腾笔记"
author:     ZeroGo
date:       2020-11-24 10:48:21 +0800
# article_header:
  # type:     cover
  # image:
    # src:    assets/articles_imgs/2020-10-26-android-notes/android_banner.webp
---
为了防止自己反复折腾，搞一个折腾笔记还是有必要的！本篇笔记会持续更新。

# 安装NVIDIA驱动
众所周知，在Linux操作系统中安装NVIDIA驱动是一件非常麻烦的事情，而且很容易崩溃。下面介绍如何使用官方的RunFile文件进行安装驱动。  
## 准备工作
你需要将驱动程序提前放入到你的电脑中，否则到命令行中复制还得需要挂载或者下载等操作浪费时间。  
安装驱动安装程序所需要的软件：``gcc cmake make``，例如
{% highlight bash %}
sudo apt install gcc cmake make
{% endhighlight %}

## 禁用Linux通用图形界面驱动nouveau
打开``/etc/modprobe.d/blacklist.conf``文件
{% highlight bash %}
sudo nano /etc/modprobe.d/blacklist.conf
{% endhighlight %}
在文件的末尾添加
```
blacklist nouveau
options nouveau modeset=0
```
保存后退出，输入指令
{% highlight bash %}
sudo update-initramfs -u
{% endhighlight %}
更新配置，重启（下列指令都可以重启）
{% highlight bash %}
sudo reboot
sudo shutdown -r now
{% endhighlight %}
## 安装驱动
重启之后应该无法打开图形界面（通过图形界面登录应该进不去桌面），打开tty超级终端界面进行登录。  
记得给你的驱动程序加执行权限，并运行
{% highlight bash %}
chmod a+x ./NVIDIA-Linux-x86_64-xxx.xx.xx.run
sh ./NVIDIA-Linux-x86_64-xxx.xx.xx.run
{% endhighlight %}
剩下的就可以按照安装程序的步骤安装即可。

# 永久挂载硬盘
挂载硬盘也是一个基本操作，但是不能总是开机的时候使用mount指令挂载吧，有些硬盘还是让他永久挂载一下。
## 准备工作
挂载硬盘需要硬盘的UUID信息和文件系统，两个信息可以通过1个指令进行查看
{% highlight bash %}
sudo blkid
{% endhighlight %}
它会打印出每个分区(partition)的信息，free space不会显示在上面。每条信息都会形如
```
/dev/sdXn: LABEL="xxx" UUID="xxx" TYPE="xxx" ...
```
LABEL有的盘有Label有的盘没有，这取决于你在使用mkfs指令的时候指没指定Label，对于挂载硬盘来说不太需要这个，我们要的是UUID和TYPE，前者就是UUID啦，后者是磁盘的文件系统类型。  
``lsblk -f``指令也可以达到上面命令的效果，而且更直观，你可以根据你的使用习惯来决定使用哪个命令。
## 永久挂载硬盘
编辑``/etc/fstab``文件
{% highlight bash %}
sudo nano /etc/fstab
{% endhighlight %}
可以看到，所有的硬盘的挂载都写在了这个文件中，包括你的系统盘、swap分区等等。在此处你可以挂载硬盘，包括NTFS文件系统的磁盘也可以挂载。  
具体的格式是
```
<file system> <mount point> <type>  <options> <dump>  <pass>
UUID=xxx  /mnt/xxx  xxx defaults  0 0
```
中间使用空格或者Tab(\t)分隔。具体参数的意思是
|字段|解释|
|:---:|---|
|file system|要挂载的分区或设备，不要被英文所迷惑|
|mount point|挂载点，说白了你要在哪里访问你的设备|
|type|挂载的分区或设备是什么文件系统类型，常见的有ext4、ntfs、auto等|
|options|挂载时的参数，一般使用defaults即可|
|dump|该挂载后的文件系统能否被dump备份命令作用，0为不能，1为每天都进行dump备份，2为不定期进行dump备份，一般选择0|
|pass|开机时是否检查磁盘，0为不检查，1为优先校验（根目录会使用这个），2为校验，一般选择0|

执行完成之后进行重启，你的电脑即可自动挂载硬盘。  
**注意！**千万不要写错了，否则进不去系统。我之前曾经把``defaults``少写了一个s，就进不去图形界面了。不过写错了也不要担心，系统如果发现这个文件发生了错误，只要你的根目录挂载没问题，他就会进入到超级终端中，允许你再次更改fstab文件。  
## P.S.: 为磁盘分区并格式化成想要的文件系统类型
如果你的磁盘都是free space的话，挂载是挂载不了的，就相当于Windows中的Disk Manager（磁盘管理）中显示未分配是一样的效果。所以使用指令分区是一项重要的技能。  
那么如何分区呢？使用强大的``fdisk``指令吧！  
我们都知道，看磁盘信息一个重要的指令就是``sudo fdisk -l``，那么这个指令也可以用于分区。直接输入
{% highlight bash %}
sudo fdisk /dev/sdX
{% endhighlight %}
其中，``sdX``是你的具体的设备名称，可以通过``sudo fdisk -l``指令进行查看。  
在新的界面中，按m加回车可以看到每一个小指令的介绍，这里只介绍最基本的分区操作：将整个盘分成1个区。  
输入n和回车进入到新建分区中，再选择主分区（输入p和回车），输入1和回车，再输入两次回车（默认的设置）即可设置完成一个盘的一个分区的建立。  
当然还没有完成，因为这只是规划了一下，并没有实际应用。因此你仍然需要输入w和回车进行应用更改，然后``fdisk``就会帮助你进行应用更改。  
建完了分区就算搞定了吗？并不是，这里与Windows不同，Windows通过磁盘管理可以直接一步到位，Linux中是将这个过程拆成了两个。现在使用强大的``mkfs``指令进行创建文件系统的工作吧！
你如果不熟悉mkfs指令的操作，可以输入``man mkfs``来看一下这个操作。其实很简单
{% highlight bash %}
sudo mkfs -t <type> /dev/sdXn
{% endhighlight %}
``<type>``处写你想要的文件系统类型，``sdXn``为你要格式化的分区。举个例子吧：
{% highlight bash %}
sudo mkfs -t ext4 /dev/sdb1
{% endhighlight %}
当然你可以将上面的命令简单简写为
{% highlight bash %}
sudo mkfs.ext4 /dev/sdb1
{% endhighlight %}
也是一样的。  
## P.S.: 使用exFAT文件系统
文件配置表（File Allocation Table，FAT）是微软发明的一种文件系统类型，Windows和Linux均支持，可用于跨操作系统传输文件。经典的FAT有FAT16、FAT32，缺点就是占用空间大，容易产生文件碎片，磁盘读写效率会比较低。  
如果需要长时间跨越操作系统操作，那么选用FAT文件系统是很不错的选择。但是常见的FAT32不支持单个文件大小超过3.6G，因此exFAT(FAT64)孕育而出。  
Linux在内核5.4版本的时候支持了exFAT文件系统，如果用不了，需要下载软件包。在Debian/Ubuntu环境中可以执行命令来下载
{% highlight bash %}
sudo apt install exfat-fuse exfat-utils
{% endhighlight %}
这样你可以访问并创建exFAT文件系统了，也就是说你可以使用``mkfs.exfat``进行格式化了。  
这里**不建议**把大硬盘格式化成exFAT，因为会导致许多文件碎片。  
# Linux常用指令
|指令|类型|示例|解释|
|:---:|:---:|---|---|
|ls|文件夹操作|ls [dir]|查看对应目录中的内容，不写[dir]字段默认为当前目录|
|ll|文件夹操作|ll [dir]|等同于ls -l|
|la|文件夹操作|la [dir]|等同于ls -a|
|mv|文件操作|mv (-T) [file or folder]|移动或重命名文件或文件夹|
|cp|文件操作|cp (-T -r) [file or folder]|复制文件或文件夹，文件夹需要加-r|
|rm|文件操作|rm (-rf) [fire or folder]|删除文件或文件夹，文件夹需要加-r，-f为强制删除|
|du|文件夹操作|du -h [dir] --max_depth 1|查看子文件夹下的文件大小|
|df|磁盘操作|df -h|看磁盘占用|

