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


---

# 切换国内最快源
```
#1. 第一部分使用 pacman-mirrors 更新官方软件源
##1.1  按照地区自动更新为最快最稳定的软件源镜像地址
  sudo pacman-mirrors --country China
##1.2. 恢复默认软件源操作
  sudo pacman-mirrors --interactive --default
##1.3 软件源更新之后，我们一般会进行系统更新
  sudo pacman -Syyu # 软件源更新完成之后进行系统软件更新操作
##1.4 查看所有可用的地区信息
  sudo pacman-mirrors -l
```

# 使用 pacman 管理软件
```
#2. 第二部分使用 pacman 管理软件
##2.1 同步并且更新你的系统
  sudo pacman -Syyu
##2.2 在软件仓库中搜索软件
  sudo pacman -Ss [software package name]
##2.3 查看已安装软件
  sudo pacman -Qs [software package name]
  sudo pacman -Qi [software package name] # 附带详细信息
  sudo pacman -Qii [software package name] # 附带更加详细的包信息
  sudo pacman -Ql # 列出所有安装的软件包
##2.4 查看软件的详细依赖
  sudo pactree [software package name]
##2.5 查看系统中那些没有被使用软件依赖包（orphans）
  sudo pacman -Qdt
##2.6 自动移除那些系统中没有被使用的依赖包【类似于Debian下的 sudo apt autoremove --purge】
  sudo pacman -Rs $(pacman -Qdtq)
##2.7 下载并安装软件包
  sudo pacman -Syu [software package name] # 从软件仓库安装
  yay -S [software package name]  # Packages from the AUR
  sudo pacman -U [/package_path/][software package name.pkg.tar.xz] # 从本地安装
  pacman -U http://www.examplepackage/repo/examplepkg.tar.xz # 从网络安装【非官方仓库】
##2.8 卸载软件
  sudo pacman -R [software package name] 
  sudo pacman -Rs [software package name] # 同时删除依赖
  sudo pacman -Rns [software package name] # 删除软件及其依赖，还有pacman生成的配置文件，即更彻底的删除
##2.9 清空缓存【默认情况下安装软件会先来缓存中查看是否已经下载过，没有再去下载，软件安装后通常下载缓存还在】
  sudo pacman -Sc
  sudo pacman -Scc # 更彻底的清理
  关于 pacman 常用就这些了，更多请使用 man pacman OR pacman -h 去查看
```



清理系统中无用的包
sudo pacman -R $(pacman -Qdtq)
清除已下载的安装包
sudo pacman -Scc

查看一下系统信息
sudo screenfetch