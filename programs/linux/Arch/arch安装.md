[TOC]

# 烧盘
rufus dd模式

# 磁盘分区
```
// 查看磁盘
$ fdisk -l
// 分区
$ cfdisk /dev/nvme0n1
// 设置EFI分区
$ mkfs.fat -F 32 /dev/nvme0n1p1
// 设置主分区
$ mkfs.ext4 /dev/nvme0n1p2
```

# 挂载至安装U盘
因为目前只能在U盘上操作，所以需要将安装目标盘挂载到当前U盘
```
$ mount /dev/nvme0n1p2 /mnt
$ mkdir /mnt/boot
$ mount /dev/nvme0n1p1 /mnt/boot
```

# 设置pacman国内源
```
$ vim /etc/pacman.d/mirrorlist
```

# 安装核心组件
```
$ pacstrap /mnt base linux linux-firmware
```

# 设置fstab
```
$ genfstab -U /mnt >> /mnt/etc/fstab
```

# 进入已安装的Arch
```
$ arch-chroot /mnt
```

# 设置时区
```
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ hwclock --systohc
```

# 设置wheel用户组
```
$ pacman -S vim neovim sudo
$ visudo // 取消[%whell All = (All) All]的注释
$ useradd -G wheel -m [username]
$ passwd [username]
$ ls /home
```
# 引导
```
$ pacman -S grub efibootmgr intel-ucode os-prober
$ grub-mkconfig > /boot/grub/grub.cfg
$ grub-install --target = X86_64-efi --efi-directory = /boot
```

# 安装常用软件
```
// 网络
$ pacman -S dhcpcd networkmanager net-tools
$ systemctl enable dhcpcd
$ systemctl enable NetworkManager
// 显示
$ pacman -S xorg plasma-meta sddm kdebase-meta
$ systemctl enable enable sddm
// 音频
$ pacman -S alsa-utils pulseaudio pulseaudio-alsec
// 中文字体
$ pacman -S noto-fonts-cjk noto-fonts noto-fonts-emoji
```

# 本地化
```
$ vim /etc/locale.gen //选择zh_CN = UTF-8
$ echo LANG = zh_CN.utf8 > /etc/locale.conf
```

# [Archlinuxcn]源
```
$ sudo nvim /etc/pacman.conf
>> [archlinuxcn]
>> Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
$ sudo pacman -Syyu
$ sudo pacman -S archlinuxcn-keyring
```


清理系统中无用的包
sudo pacman -R $(pacman -Qdtq)
清除已下载的安装包
sudo pacman -Scc

查看一下系统信息
sudo screenfetch

