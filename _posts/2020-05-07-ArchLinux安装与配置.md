---
layout: post
title: Arch Linux安装与配置
categories: Linux
description: 基于archlinux wiki总结一下Arch Linux的安装与配置
keywords: Arch
---
从被Arch的理念吸引，到真正动手安装，中间经历了好几次失败。从开始相信arch wiki到对自己的英语水平失望转而搜索各种博客，最后再回到arch wiki。一来一回渐渐对arch wiki理解得更明了，结合博客与官方wiki，总结一下整个安装过程。由于这次安装是基于vmware 14，所以官方中提到的准备阶段不包含本文中，下载镜像很简单，也不在本文涉及，arch版本用的是archlinux-2021.04.01-x86_64.iso。

创建虚拟机的过程不一步步列出，硬件兼容选择Workstation 14.x，客户机选择Linux（其他Linux 4.x或更高版本内核 64 位），处理器参数根据自己电脑配置进行选择，内存建议够用即可，不必追求过高内存，否则会影响宿主机性能。我的宿主电脑16G物理内存，选择2G作为arch使用。网络选择“使用网络地址转换（NAT）”，I/O为“LSI Logic”，磁盘“SCSI”。磁盘空间不要太小，尤其是你要创建交换（swap）分区的情况下，因为交换分区的作用是要用来跟内存进行数据交换，内存为2G，尽量大于等于内存空间，这里我选择磁盘容量40G。

虚拟机创建完成后挂载archlinux-2021.04.01-x86_64.iso镜像到CD/DVD，启动虚拟机。一路进到arch的安装程序界面。来到关键的安装过程。

### Pre-installation（准备）
#### Verify the boot mode（检验启动模式）
其实在启动虚拟机后的第一个界面就可以看到，我们创建的虚拟机是基于BIOS启动的。BIOS是固件，是一开始是由它来引导bootloader，再由bootloader引导arch系统。但是我们这里还是输入如下命令来检验一下到底是基于BIOS，还是UEFI。

```shell
ls /sys/firmware/efi/efivars
```

输出结果若是具体的路径则证明是UEFI模式，否则会报错“no such file or dictionary”。由于我们这里是BIOS模式，所以会出现类似错误信息。

#### Connect to the internet（联网）
由于具体的宿主机不同，宿主机的网络环境也不尽相同。所以这里对联网这块不做详细阐述。我这里配置虚拟机时选择的是NAT，同宿主机是相同网络环境，启动后可以直接联网。但还是ping一下试试。

```shell
ping www.baidu.com
```

#### Update the system clock（更新系统时钟）
说实话我不太清楚官方wiki中这一步的作用，如果不同步时钟可能会对后续安装有影响，暂且先同步一下。

```shell
timedatectl set-ntp true
```

#### Partition the disks（磁盘分区）
看wiki写的是用fdisk命令进行分区，但网上也有用cfdisk，两个都用过，道理一样，cfdisk提供伪图形接口，看自己需要。分区包含根分区（/）36G，交换分区（/swap）4G，其中根分区是必须要有的，交换分区可以没有。使用cfdisk分区时要选择dos类型，根分区选择Linux，交换分区选择Linux Swap。最后不要忘了写入分区表。

![分区]({{assets_base_url}}/images/blog/Arch/安装与配置/分区表.jpg)

#### Format the partitions（格式化分区）
分区创建成功后，必须将其格式化为一定的文件格式，这样操作系统才能识别。Arch支持的文件系统格式有很多，详情可以去官方wiki查看，这里我们选择将根分区格式化为ext4。

```shell
mkfs.ext4 /dev/sda1
```

关于交换分区（swap）官方的说法是，swap分区的格式为82，尽管任何格式都可以，但还是推荐使用82这种格式，因为大多数情况下systemd会自动检测并挂载该分区。使用以下命令将交换分区格式化为82。

```shell
mkswap /dev/sda2
```

#### Mount the file systems（挂载分区）
将根分区（/）挂载到/mnt

```shell
mount /dev/sda1 /mnt
```

如果创建了交换分区（swap）则开启交换分区

```shell
swapon /dev/sda2
```

### Installation（安装）
#### Select the mirrors（配置镜像）
安装不仅需要联网，还要配置可访问的资源镜像。编辑/etc/pacman.d/mirrorlist文件，将（国内）镜像地址配置进去，其他不需要的（国外）镜像地址可以注释掉。关于如何找到某地区镜像地址，arch wiki给出自动生成镜像地址文件接口：[生成镜像源](https://archlinux.org/mirrorlist/)。

#### Install essential packages（安装必要的包）
使用pacstrap脚本安装基础包（base），Linux 内核（kernel）和通用硬件（firmware）驱动。安装目标路径为/mnt，即根分区。

```shell
pacstrap /mnt base linux linux-firmware
```

如果要安装其它包或包组，则只需要将相应的包或包组名追加到上面的命令后面即可。或者在安装完成后使用pacman命令安装，两种方式都是一样的。但这里建议将vim、dhcpcd两个包也一并装上，因为如果没有dhcpcd，安装完之后还要进行复杂的网络配置，方便起见安装上这两个工具。安装过程没有报错则表明系统安装成功，至此安装过程结束。

### Configure the system（配置系统）
#### Fstab（分区文件）
Linux系统中的分区、各种设备，或远程文件系统最终都会被映射到文件系统，而后为Linux识别。因此生成一个文件，文件中每一行记录一个分区或设备，并分配一个唯一的UUID标识，即fstab文件。该文件中的这些定义将在引导时以及重新加载系统管理器的配置时动态地转换为systemd挂载单元。默认设置将在启动需要挂载文件系统的服务之前自动fsck和挂载文件系统。之前我们将根分区（/）和交换分区（/swap）挂载到了/mnt，所以我们使用以下命令生成fstab文件。

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

生成的fstab文件位于新系统的/etc下面。其内容如下所示：

![fstab文件内容]({{assets_base_url}}/images/blog/Arch/安装与配置/fstab文件内容.jpg)

检查一下该文件，确认没有问题，否则需要进行修正。

#### Chroot（切换系统）
将root切换到新系统

```shell
arch-chroot /mnt
```

#### Time zone（设置时区）
设置新系统时区，这里亚洲/北京为例

```shell
ln -sf /usr/share/zoneinfo/Asia/Beijing /etc/localtime
```

上面命令只是建立联接，需使用以下命令生成/etc/localtime文件

```shell
hwclock --systohc
```

#### Localization（本地化）
编辑/etc/locale.gen文件，去掉en_US.UTF-8 UTF-8前面的注释，当然也可以去掉zh_CN.UTF-8 UTF-8（中文）前面的注释，但不建议现在这样做，因为后面可能要安装图形界面，如果现在去掉的话可能会导致图形界面中的中文乱码，出现类似“口口”字符。

编辑完成后重新生成/etc/locale.gen

```shell
locale-gen
```

创建locale.conf配置文件, 并设置对应的语言

```shell
LANG=en_US.UTF-8
```

#### Network configuration（网络配置）
创建/etc/hostname文件，文件中仅录入主机名，如“freeman”

```shell
freeman
```

同时hosts文件中添加对应主机名的入口，如果本机有固定IP地址则替换掉127.0.0.1。

```shell
127.0.0.1	localhost
::1		    localhost
127.0.1.1	freeman.localdomain	freeman
```

如果是新系统为了更完整地配置网络环境可能需要安装网络管理软件，前面我们已经安装了dhcpcd，这里不再重复安装。

#### Initramfs（初始化内存文件系统）
ramfs是内存文件系统，是为了提高系统运行速度，在内存中建立了一套文件系统，这样系统操作文件的速度就同读取内存中的数据一样。因为在使用pacstrap安装系统内核时已经默认（执行mkinitcpio脚本）初始化好了内存文件系统，所以就不再需要进行初始化。但是如果存在 LVM, system encryption or RAID，则需要修改 mkinitcpio.conf并使用以下命令重新生成initramfs 镜像。

```shell
mkinitcpio -P
```

#### Root password（修改Root用户密码）

```shell
passwd
```

#### Boot loader（安装引导程序）
新系统安装完成后仍需要安装一下引导程序（Boot Loader），引导程序是由固件（BIOS或UEFI）启动的一个软件。引导程序负责依据指定参数加载需要的内核，并依据配置文件（mkinitcpio.conf）初始化内存文件系统。常用的引导软件有systemd-boot、LILO、GRUB，这里我们选择GRUB。

安装grub官方wiki中没有给出，但也比较简单，进到刚安装好的系统（使用arch-chroot /mnt），输入以下命令并回车。

```shell
pacman -S grub
```

如果系统中已安装 grub-legacy，则会将其替换掉。安装完成后，执行以下命令。

```shell
grub-install --target=i386-pc /dev/sda
```

该命令是将grub安装在磁盘/dev/sda，切记/dev/sda后面不要跟数字，因为引导程序是要安装在磁盘分区之前的位置，而不是某分区上，否则固件无法找到该引导程序。

安装完成后，需要生成主配置文件“/boot/grub/grub.cfg”。生成过程可能受到/etc/default/grub中的各种选项和/etc/grub.d/中的脚本的影响。如果你没有进行额外的配置，那么自动生成程序将确定配置文件要引导的系统根文件系统。若要想成功，系统引导及根文件系统是很重要的。

配置文件默认路径为“/boot/grub/grub.cfg”，而不是“/boot/grub/i386-pc/grub.cfg”。如果试图在chroot或system -nspawn容器中运行grub-mkconfig，你可能会发现它并不起作用。

使用grub-mkconfig工具生成“/boot/grub/grub.cfg”文件。默认情况下，生成脚本会自动将所有安装的Arch Linux内核的菜单项添加到生成的配置中。安装或删除内核之后，您只需要重新运行下面的grub-mkconfig命令。

```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

Grub的入口菜单可以通过编辑配置文件进行调整，这里不做详细阐述，有兴趣的可以去看一下wiki。

### Reboot（重启）
通过exit或Ctrl + d命令退出chroot环境，通过umount -R /mnt解除挂载，当然这是可选的。最后输入reboot命令重启系统。

### Post-installation（后续安装）
有关系统管理和后续安装教程(如设置图形用户界面、声音或触控板)请转至[一般建议](https://wiki.archlinux.org/title/General_recommendations)。有关可能感兴趣的应用程序列表，请参阅[应用程序列表](https://wiki.archlinux.org/title/List_of_applications)。


本文结合官方wiki和其他博客校正认识后编写，结合自己电脑环境完成整个安装过程。中间发现所有博客中提到的内容包括细节其实都可以在官方wiki中找到，所以安装的关键在于仔细阅读并理解官方wiki中的内容，而不是简单的复制粘贴网上的命令。那样不也就达不到学习知识的目的了不是么。