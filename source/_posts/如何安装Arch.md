---
title: 如何安装Arch
date: 2017-07-07 00:00:00
tags: 
- Linux
categories: 
- Linux
---

# 如何安装Arch
Linux是每个人都绕不过去的坑，在之前为了装Arch作者也踩了好多好多好多好多坑，所以装好arch之后就有这么一个想法，写一篇小白都能看懂的教程，那么过去遇到的困难也就有那么一点意义，同时也是为了防止自己以后忘了，再回来看看。
<!-- more -->

# 如何安装Arch


## 为什么是Arch

​    装arch就像是组装积木，相比于Deepin和Ubuntu以及其他的一些发行版，arch并没有把所有的组件都给你装在一起，让你开箱即用，而是给你提供了不同的零件，让你一个一个把玩，选出自己喜欢的零件来组装自己的系统，我觉得这也是用linux的最大的一个乐趣。但是相对与Deepin和Ubuntu，arch安装的学习成本很高，而且网上的教程参差不齐，所以如果自己在网上找教程的话非常麻烦，本文也正是从这个出发点开始，想用一篇教程讲完所有的安装过程，希望能够大幅度的减少小白踩坑的时间，提高效率。

## 几点建议

1. 善用 tab，tab 在 linux 终端里有自动补全的功能，按两下 tab 则会出现所有可能的结果;
2. 善用上下方向键，bash 会记录你之前的命令，当你输入两条类似命令时，请充分利用用
3. 强大的心态，计算机这东西是死的，别认为它有黑魔法，它说你错了，那你肯定有地方错了，你如果每步都没错，你 arch 装不好是不可能的
4. 善用备份（.bak）与注释（#号），这样在你出问题之后可以改回原样
5. 多学英文，无英语，不 linux    

## 注

 这篇文章很大一部分参考了 **@Levi** 大神的文章，在此特别感谢。

## 使用说明

- 这篇文章字会非常非常非常多，而且会有很多的说明，所以希望你能够耐心看完。
- 请看清是否有**空格** 。
- **下文适用于全新安装arch** ，也就是说如果你的机子上啥系统也没有，或者说你想删掉现有系统只安装arch，就往下看，**如果你已经有Windows并且想安装Windows和arch双系统的话，请拉到最后面**，会有专门的介绍。
- **错误的操作可能会造成数据丢失**，所以务必要看清每一步是怎么做的。
- 本文中所有命令用#开始，但是实际输入时**不用输入#**，仅作为是一个标志。

## 开始安装吧

### 准备工作

#### 关闭BIOS 关闭安全启动

- 推荐 Windows 下用 ISO2USB 写入 iso 到 U 盘，U 盘格式选择 FAT32（也可以使用usbwriter，**不要使用ultraiso**，ultriso写入之后貌似不能启动）
- 找一个输完密码就能上网的 wifi，输入命令`# wifi-menu -o` 然后在列表里选择你的 wifi，然后按两次回车，输入密码，再按回车。

### 分区

​这里主要使用 ext4 分区格式，因为这是现存的最好的文件系统，使用`lsblk`命令查看分区挂载情况，你会看到诸如/dev/sda1 之类的文字
​    
用 parted 工具分区
```
#parted /dev/sda
(parted) #mklabel gpt#建立 gpt 分区表
(parted) #mkpart esp fat32 1MiB 513MiB#建立esp 分区，大小为 512MiB
(parted) #mkpart root ext4 513MiB 100%#建立根区，大小为剩余所有空间
(parted) #set 1 boot on #设置第一个分区即 sda1 可启动
#quit#退出
（如果你是固态加机械的组合的话，请把下面这一段看完，认 sda 为固态，sdb为机械，请根据自己的实际情况调整）
#parted /dev/sdb
(parted) #mklabel gpt#建立 gpt 分区表
(parted) mkpart home ext4 1MiB 100% #建立 home分区，大小为所有空间
quit #退出
lsblk #再次查看分区挂载情况
```

### 格式化
```
#mkfs.vfat -F 32 /dev/sda1 && mkfs.ext4 /dev/sda2 && mkfs.ext4 /dev/sdb1
```

### 挂载
```
（使用 chroot 的必要步骤，和 arch 的维修有关）
#mkdir /mnt/esp #建立 esp 分区挂载目录（#）
#mkdir /mnt/home #建立 home 分区挂载目录（如果你是双硬盘请跳过此步）
#mount /dev/sda2 /mnt #挂载根分区到 sda2 （#）
实际在挂载下面两个分区时可能会报错，请重复执行有#号注的两步
#mount /dev/sda1 /mnt/esp#挂载 esp 分区到 sda1
#mount /dev/sdb1 /mnt/home #挂载 home 分区到sdb1
注意挂载顺序，根分区一定要先挂载
```

### 安装
```
#nano /etc/pacman.d/mirrorlist
你将会看到一系列的服务器地址，请重点寻找[china]字样的
配置源，这个配置会自动复制到安装好的系统中
使用 alt+6 复制行 ctrl+u 粘贴行ctrl+k剪切行
复制需要的源到最上面的注释（#号）下边，如 163 源ustc 源，清华源（推荐只保留163和ustc的源，如果你所在校也维护了镜像源的话，也可以用这个方法把本校镜像源加上这样下载速度会很快），请去掉你复制内容里的#号
#pacstrap /mnt base base-devel #安装基本系统
```

### 配置
```
#genfstab -U -p /mnt > /mnt/etc/fstab #生成fstab
#cp /mnt/etc/fstab /mnt/etc/fstab.bak#备份fstab
#nano /mnt/etc/fstab #编辑 fstab
把 esp 的相关内容移到最上边，ctrl+k 剪切行，ctrl+u粘贴行
#arch-chroot /mnt #chroot 到目标系统
```

#### 本地化设置
```
#nano /etc/locale.gen #编辑 locale.gen 文件
#en_US.UTF-8 UTF-8 #去掉注释
#zh_CN.UTF-8 UTF-8 #去掉注释
#locale-gen#生成指定的本地化文件
#echo LANG=en_US.UTF-8 > /etc/locale.conf #提交本地化选项（注意大写）
```

#### 时区设置
```
#tzselect #查看可用的时区配置 选 asia 序号，然后选 shanghai 或者 xinjiang的序号
#ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime #设置当前时区为 Shanghai
#hwclock --systohc --utc #设置硬件时钟为 UTC（前面的两条横线每个均由两个减号构成）
设置 root 密码
#passwd 然后两次输入自己想设定的密码，输入时屏幕不会有任何显示，这是linux 一种安全措施
```

### 设置主机名
```
#echo X > /etc/hostname #X 替换为需要的名字，区分大小写
#cp /etc/hosts /etc/hosts.bak #备份 host 文件
#nano /etc/hosts #编辑/etc/hosts 添加同样的主机名
127.0.0.1 localhost.localdomain localhost X
::1 localhost.localdomain localhost X
红色标注的 X 是你实际要输入的内容，其他内容文件中已经自带
X 替换为需要的名字，区分大小写（注意留空格）
```

### 配置启动

#### 复制内核文件到 esp 分区
```
#cp /boot/vmlinuz-linux /esp/vmlinuz-linux
#cp /boot/initramfs-linux.img /esp/initramfs-linux.img
#cp /boot/initramfs-linux-fallback.img
#/esp/initramfs-linux-fallback.img
```

#### 安装并更新 efi 启动文件
```
#bootctl --path=/esp install
#bootctl --path=/esp update
```

#### 编辑启动文件

`#nano /esp/loader/loader.conf`
​    文件中有一行注释，不需要动，直接添加如下内容即可
```
default arch-*
timeout 3（这个数值可以改为 0）
editor 0
nano /esp/loader/entries/arch.conf`
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=PARTUUID= 14420948-2cea-4de7-b042-40f67c618660 rw
```
使用 blkid -s PARTUUID -o value /dev/sdxY 找到某个分区的 PARTUUID, ‘x’和 ‘Y’ 分别是磁盘和分区编号（实际上这是万里长征的最后一步，但务必细心，注意是 root 分区的磁盘号，如果是按照教程来的话应该是/dev/sda2,请用笔记录，仔细校对，替换上面的14420948-2cea-4de7-b042-40f67c618660，并且注意不要忘了最后面的rw）
    

#### 启用 ntp 自动更新时间
```
#timedatectl set-ntp true
#pacman -Syu （更新系统的命令，常用）
```

#### 添加用户
```
#useradd -m X X 改成自己喜欢的名字
#passwd X #设置密码
```

#### 配置 sudo
```
EDITOR=nano visudo
用户名添加到 root 下，格式和 root 一样
形如：
root ALL=(ALL) ALL
X ALL=(ALL) ALL
```

#### 安装 GNOME （建议新手安装，原因是基础软件齐全，开箱即用，稳定高效可定制，推荐的做法是先安装gnome，熟悉之后可以替换为kde或者是xfce，xfce占用资源比较少，但是不如kde漂亮）
```
#pacman -S gnome #只安装 GNOME 的基本环境
#pacman -S gedit file-roller gnome-tweak-tool p7zip #安装文本编辑器，
```

#### 归档管理器，优化工具，7zip 解压缩支持
```
#pacman -S wqy-zenhei 或者 wqy-microhei #安装文泉驿正黑字体
#pacman -S ibus-rime #安装 ibus 小狼毫输入法
```

#### 输入法配置
```
#nano ./.xinitrc
export LANG=zh-CN.UTF-8
export GTK_IM_MODULE=ibus
export QT_IM_MODULE=ibus
export XMODIFIERS="@im=ibus"
```

### 安装显卡驱动

`#pacman -S xf86-video-***` #安装显卡驱动 ati,intel,nouveau,A 卡 I 卡推荐开源驱动，N 卡推荐闭源驱动（#）

双显卡请按照下面指示的部分执行，替换加（#）的这步
双显卡看这里（特指 N 卡加 I 卡）
```
#pacman -S intel-dri xf86-video-intel bumblebee nvidia
#gpasswd -a X bumblebee X 指你的用户名
```

下面几步命令相对复杂可以等有图形界面后复制粘帖
关闭独显：
下面的配置文件如果没有需要自行创建
```
#pacman -S bbswitch
#nano /etc/modules-load.d/bbswitch.conf // 写入下边内容，每次启动都会加载 bbswitch 模块
bbswitch
#nano /etc/modprobe.d/bbswitch.conf // 写入下边内容，关闭 bbswitch 默认加载参数
options bbswitch load_state=0
#nano /etc/modprobe.d/nouveau_blacklist.conf
```

写入下边内容，有时候 bbswitch 加载了，但是不能关闭显卡，因为有些模块正在占用着，因此要禁掉
```
blacklist nouveau
blacklist nvidiafb
#nano /usr/lib/systemd/system-shutdown/nvidia_card_enable.sh
```
需要运行权限，写入下边内容，每次 reboot 的时候，显卡都是关闭的，不管是重启到 windows 还是 linux，都会找不到设备，必须彻底关机才行。解决该问题的办法就是每次重启都启用显卡。
```
#!/bin/bash
case "$1" in
reboot)
echo "Enabling NVIDIA GPU"
echo ON > /proc/acpi/bbswitch
;;
*)
esac
```

### 重启前的最后操作

#### 可选：建立交换文件（如果内存足够大，8G+请跳过）

### 启用内核更新

### 裸机安装arch就到这里啦，下面是双系统部分

## Windows 和 Arch 双系统部分

​    一般来说，大部分是已经有 windows 的情况下想要装 arch，我也推荐这种先装 windows再装 arch 的方式，先装 arch 的话因为 windows 的分区的问题会比较麻烦。下面是在已经安装 windows 的情况下安装 arch。
安装之前要先进入 windows 给 arch 单独分一个区。

​    进入arch控制台之后，输入 fdisk –l ,此时应该会有一个 efi 分区。

`#mkfs.ext4 /dev/sdNX` 进行格式化，X 是要安装 arch 的分区（也就是windows下面搞出来的分区，找出这个分区最简单的办法是看分区大小）,N 是上面看的字母，a 或 b 或 c 等等（比方说sda1或者sdb2等等，仅作举例），**对别的分区不要进行格式化**（惨痛的教训）

`#mount /dev/sdNX /mnt` 挂载根分区

`#mkdir /mnt/boot` 创建启动分区挂载点

`#mount /dev/sdNX /mnt/boot` 挂载启动分区 注意这个/dev/sdNX 是上面那个标示 EFI 分区的磁盘号。

接下来输入

`#wifi-menu` 连接 wifi
     下面的步骤和上面是重复的，就是接着上面的【安装】部分一直做到配置启动之前，后面配置启动的部分不做
     下面注意，要进行配置启动的部分了，下面的步骤**直接关系到能不能成功引导双系统**。

`#pacman –S grub os-prober efibootmgr`

`#grub-install --recheck /dev/sdX`

​    如果提示 error: cannot find EFI directory，说明找不到 EFI 文件夹的位置，还需要加上-efi-directory 参数指明安装位置也就是下面这个命令

`#grub-install --recheck /dev/sdX --efi-directory=/boot`

​    如果没有错误说明安装成功。安装完毕之后还需要生成一个 grub 配置文件。这一步会探测系统上已经安装的系统并写入到配置文件中。但是由于在安装介质环境中，此时 Windows 系统可能会探测不到。等到一会重启真正进入 arch 系统之后还需要重新执行以下这个命令，这样就会正常探测到所有系统了。所以**重启之后看不到windows的启动项不用担心啦**。

`#grub-mkconfig –o /boot/grub/grub.cfg` 生成配置文件

​    下面从上文 **启用 ntp 自动更新时间**一直做到**重启前的最后操作**这一部分然后重启。
​    重启之后**再次执行**

`#grub-mkconfig –o /boot/grub/grub.cfg` 此 时 应 该 就可以 探测 到windows 系统，后面就是一些配置了。照着上面接下来的部分接着做就行啦。

**到此教程结束啦，撒花★,°:.☆(￣▽￣)/$:.°★***
