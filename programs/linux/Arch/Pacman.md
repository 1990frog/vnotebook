# 添加archlinuxCN源
```
#编辑文件 sudo vi /etc/pacman.conf   末尾追加
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
[blackarch]
SigLevel = Never
Server = https://mirrors.ustc.edu.cn/blackarch/$repo/os/$arch
```
# 选择pacman中文镜像
```
sudo pacman-mirrors -c China​
sudo pacman-mirrors -i -c China -m rank
sudo pacman -Syyu
```
# pacman命令
```
安装 pacman -S
删除 pacman -R 
移除已安装不需要软件包 pacman -Rs 
删除一个包,所有依赖 pacman -Rsc 
升级包 pacman -Syu 
查询包数据库 pacman -Ss 
搜索以安装的包 pacman -Qs 
显示包大量信息 pacman -Si 
本地安装包 pacman -Qi 
清理包缓存 pacman -Sc

pacman -S package_name #安装软件包
pacman -R package_name #删除软件包

pacman -Rs package_name #顺便删除软件包相关依赖
pacman -Syu #升级系统中的所有包
pacman -Ss package #查询软件包
pacman -Qs package #查询已安装的包
pacman -Qi package #显示查找的包的信息
pacman -Ql package #显示你要找的包的文件都安装的位置
pacman -Sw package #下载但不安装包
pacman -U /path/package.pkg.tar.gz #安装本地包
pacman -Scc #清理包缓存，下载的包会在/var/cache 这个目录
pacman -Sf pacman #重新安装包

```
# 解决签名错误，安装软件包报错问题
```
#导入GPG Key
sudo pacman -S archlinuxcn-keyring
```


要删除软件包和所有依赖这个软件包的程序
警告: 此操作是递归的，请小心检查，可能会一次删除大量的软件包。
```
pacman -Rsc package_name
```

要罗列所有明确安装而且不被其它包依赖的软件包
pacman -Qet

检查一个安装的软件包被那些包依赖，可以使用 pkgtools[AUR]中的whoneeds:
whoneeds package_name
或者
pactree -r package_name

清理软件包缓存
pacman -Sc
