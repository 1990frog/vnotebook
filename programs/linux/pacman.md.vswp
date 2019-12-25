vnote_backup_file_826537664 /home/cai/Documents/vnote_notebooks/vnotebook/programs/linux/pacman.md

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
