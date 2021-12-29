# Arch&Win10双系统安装及相关知识

## $XDG_CONFIG_HOME

您*不需要*在任何地方定义它，除非您想更改默认值。
[XDG基本目录规范](http://standards.freedesktop.org/basedir-spec/latest)明确指出：

> 如果`$XDG_CONFIG_HOME`未设置或为空，`$HOME/.config`则应使用默认值 。

因此将其定义为默认值是多余的。所有兼容的应用程序将已经使用`$HOME/.config`
但是，如果您*确实*想在Debian / Ubuntu系统中更改默认设置，最好的地方是：

* 对于系统范围的更改，影响到所有用户： `/etc/profile`

- 仅针对您的用户： `~/.profile`

## 双系统

#### 前置

win10 安装在第一块磁盘且有第二块磁盘

#### 分区

fdisk /dev/sdb
mkfs.ext4 /dev/sdb1
mkdir -p /mnt/boot/EFI
mount /dev/sdb1 /mnt
mount /dev/sda2 /mnt/boot/EFI

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
grub-install --target=x86_64-efi --efi-directory=/boot/EFI
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg

#### Reboot

pacman -S intel-ucode
passwd
systemctl enable dhcpcd.service
exit
umount -R /mnt

#### Q&A

安装以后重新启动，发现Grub引导界面只有 Arch 引导项，没有 windows ！而且在安装Grub的过程（grub-mkconfig操作）中，确实没有类似“Found Microsof Windows...”的输出信息，所以在 Grub 引导界面可能会看不到 Windows Manager 选项，需要手动修复一下。

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