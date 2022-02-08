---
layout: post
title: Arch&Win10双系统安装及相关知识
categories: 操作系统
description: Arch&Win10双系统安装及相关知识
keywords: Archlinux 双系统
---
## 双系统

#### 前置

win10 安装在第一块磁盘且有第二块磁盘

#### 分区

fdisk /dev/sdb
mkfs.ext4 /dev/sdb1
mkdir /mnt/boot
mount /dev/sdb1 /mnt
mount /dev/sda2 /mnt/boot

#### Install

vim /etc/pacman.d/mirrorlist   OR
cat /etc/pacman.d/mirrorlist awk "/China/,/Server/" > mirrorlist
mv mirrorlist /etc/pacman.d/mirrorlist

pacstrap /mnt base base-devel linux linux-firmware sudo vim git dhcpcd
genfstab -U /mnt >> /mnt/etc/fstab

#### Grub

arch-chroot /mnt
pacman -S grub 
pacman -S os-prober efibootmgr
os-prober
grub-install --target=x86_64-efi --efi-directory=/boot
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg

#### Reboot

pacman -S intel-ucode
passwd
systemctl enable dhcpcd.service
exit
umount -R /mnt
reboot

#### Configuration

vim /etc/pacman.conf ## 添加如下语句

```shell
[archlinuxcn]
Include = /etc/pacman.d/archlinuxcn-mirrorlist
```

\#\# 在 /etc/pacman.d/ 下新建 archlinux-mirrorlist 文件并添加阿里源等，如下

```shell
## 阿里源
Server = http://mirrors.aliyun.com/archlinuxcn/$arch
```

pacman -Syy
pacman -S archlinuxcn-keyring  ## 如果报错“could not be locally signed...”执行下面命令

```shell
pacman -Syu haveged
systemctl start haveged
systemctl enable haveged

rm -fr /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinuxcn
```

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
vim /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "chappy" > /etc/hostname
vim /etc/hosts

```shell
127.0.0.1	localhost
::1	localhost
127.0.1.1	chappy.localdomain chappy
```

pacman -S wqy-microhei
\#\# 添加用户
useradd -m -G wheel tdl
passwd tdl
\#\# 默认没有安装vi，无法执行sisudo命令，解决办法为将 vim 软链接到 vi
ln -s /usr/bin/vim /usr/bin/vi
visudo ## 解注掉 %wheel 行，保存退出
\#\# 安装xorg、窗口管理器（dwm）
pacman -S xorg-server xorg-xinit xorg-apps

\#\# 切换到普通用户
mkdir Desktop
git clone http://git.suckless.org/dwm
git clone http://git.suckless.org/st
git clone http://git.suckless.org/dmenu
cd ~/Desktop/dwm
make
make clean insall
cd ~/Desktop/st
make 
make clean install
cd ~/Desktop/dmenu
make
make clean install
cp /etc/X11/xinit/xinitrc ~/.xinitrc
vim ~/.xinitrc ## 删除最后xterm相关的几行，添加下面语句后保存退出

```shell
exec dwm
```

startx

#### Q&A

1.安装以后重新启动，发现Grub引导界面只有 Arch 引导项，没有 windows ！而且在安装Grub的过程（grub-mkconfig操作）中，确实没有类似“Found Microsof Windows...”的输出信息，所以在 Grub 引导界面可能会看不到 Windows Manager 选项，需要手动修复一下。

首先查看ESP分区的uuid（即windows 的EFI分区，如/dev/sd2：UUID="2841-1E44"）

```shell
blkid ##或输入blkid /dev/<ESP所在磁盘，例如sda、sdb、nvm0>
```

使用root用户登录Archlinux，通过以下命令编辑 Grub 的配置文件

```shell
vim /boot/grub/grub.cfg
```

追加如下内容：

```shell
menuentry 'Microsoft Windows 10' {
	innmod part_get
	insmod fat #一般填fat，不要填vfat
	insmod chain
	search --fs-uuid --set=root xxxx-xxxx ## 这里填上ESP分区的UUID
	chainloader (${root})/EFI/Microsoft/Boot/bootmgfw.efi
}
```

保存后重启，成功！

2.正常安装是没有问题的，上面提到的执行 os-prober 未找到 windows 系统的问题原因是因为 Grub 默认配置 GRUB_DISABLE_OS_PROBER=false这行被注释掉了，即默认不进行系统探测，在/etc/default/grub文件中找到这一行，将前面的“#”去掉，然后再次执行 os-prober 和 grub-mkconfig -o /boot/grub/grub.cfg 即可！

```shell
## /etc/default/grub
GRUB_DISABLE_OS_PROBER=false
```

## 小知识：$XDG_CONFIG_HOME

您*不需要*在任何地方定义它，除非您想更改默认值。
[XDG基本目录规范](http://standards.freedesktop.org/basedir-spec/latest)明确指出：

> 如果`$XDG_CONFIG_HOME`未设置或为空，`$HOME/.config`则使用默认值 。

因此将其定义为默认值是多余的。所有兼容的应用程序将已经使用`$HOME/.config`
但是，如果您*确实*想在Debian / Ubuntu系统中更改默认设置，最好的地方是：

* 对于系统范围的更改，影响到所有用户： `/etc/profile`

- 仅针对您的用户： `~/.profile`