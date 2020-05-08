Manjaro安装各种坑

[TOC]
# 切换tty
按ctrl+alt+F2(F1-F6都可以一试，有的F1可能是图形界面)，等一会就可以进入tty，用户名密码登录后用top查看高负载的进程，kill PID即可
# sysctl.conf
manjaro和ubuntu系还是很多不一样的，比如没有了sysctl.conf，直接在sysctl.d中创建即可。等等吧，可以从archlinux中找答案
# 系统配置
## 更新引导分区表
```
更新引导分区表，之后重启可以看到有manjaro和win10的条目
sudo update-grub

修改 grub2 的等待时间
无论你的电脑是否有 2个或更多的操作系统，只要安装了 LinuxMint/Ubuntu，就必然会安装grub2作为引导管理器。grub2 启动时，会在默认的启动项上停留数秒（默认 10秒），等待用户选择。我们可以把这个时间改的更短。如果是 LinuxMint/Ubuntu 单系统，可以直接改为0，即直接进入，无需等待。
以管理员身份编辑 grub 配置文件，修改 GRUB_TIMEOUT 项后的数字。
sudo vi /etc/default/grub

```
## 切换英文主目录
```
sudo pacman -S xdg-user-dirs-gtk
export LANG=en_US
xdg-user-dirs-gtk-update
export LANG=zh_CN.UTF-8
```
# N卡画面撕裂
## 页面滚动屏幕撕裂
sudo nvidia-xconfig
```
保存配置文件至
/etc/X11/xorg.conf
```
[官方解决方案](https://devtalk.nvidia.com/default/topic/957814/linux/prime-and-prime-synchronization/)
```
sudo nvim /etc/modprobe.d/nvidia-drm-1.conf
options nvidia_drm modeset=1
```
## 屏幕缩放导致konsole出现横线
设置行间距为1

# 软件安装
## 安装搜狗输入法
```
sudo pacman -S fcitx-sogoupinyin
sudo pacman -S fcitx-im
sudo pacman -S fcitx-configtool # 图形化的配置工具

修改配置文件
~/.xprofile
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```
我用的是Manjaro系统，突然有一天搜狗就不能用了，总是提示上述语句。删除了相关文件并且重启还是没有用。后来在终端中输入 
```
sogou-qimpanel
```
提示找不到libfcitx-qt.so，于是找到原因，安装fcitx-qt4就可以成功解决上述问题。
```
yaourt -S fcitx-qt4
```

## fcitx打中文经常漏英文出来
将fcitx-im全部的包都安装

## 安装mlocate
```
sudo pacman -S mlocate
updatedb
locate file
```
## 安装QQ
```
https://github.com/countstarlight/deepin-wine-tim-arch
```
## 安装windows虚拟机
```
sudo pacman -S virtio-win
```
## vargent谁用谁香

## shadowsocks
config.json
```json
{
    "server":"",
    "server_port":,
    "local_port":,
    "password":"",
    "method":""
}
```
启动命令
```
/usr/bin/ss-local -c /etc/shadowsocks-libev/config.json
```
服务启动
/usr/lib/systemd/system/ss.service
```
[Unit]
Description=SS Service
After=multi-user.target
 
[Service]
Type=idle
#ExecStart=/usr/bin/python /opt/shadowsocksr/shadowsocks/local.py -c /etc/shadowsocks/config.json
ExecStart=/usr/bin/ss-local -c /etc/shadowsocks/config.json
 
[Install]
```
systemctl daemon-reload

systemctl start ss

## 缺失ifconfig
```
sudo pacman -S net-tools
```

# 常用命令screenfetch

