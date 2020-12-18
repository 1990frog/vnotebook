[TOC]

# 其他的管理工具
+ distutils：仅用于打包和安装，严格来讲不算是包管理工具
+ setuptools：distutils的增强版，扩展了distutils，提供更多的功能，引入包依赖的管理，easy_install就是它的一个命令行工具，引入了 egg 的文件格式。
+ Pipenv：一个集依赖包管理（pip）及虚拟环境管理（virtualenv）的工具

# 查询
## 查询当前环境安装的所有软件包
`pip list`
## 查询pypi上含有某名字的包
`pip search pkg`
## 查看包详情
`pip show pkg`

# 下载
## 在不安装软件包的情况下下载软件包到本地
`pip download --destination-directory /local/wheels -r requirements.txt`
## 从本地目录安装包，而不从pypi上安装
`pip install --no-index --find-links=/local/wheels -r requirements.txt`
## wheel ？
```
pip install wheel
pip wheel --wheel-dir=/local/wheels -r requirements.txt
```

# 安装
## 使用 pip install <pkg> 可以很方便地从 pypi 上搜索下载并安装 python 包。
`pip install requests`
## 只从本地安装，而不从 pypi 安装
```
# 前提你得保证你已经下载 pkg 包到 /local/wheels 目录下
pip install --no-index --find-links=/local/wheels pkg
```
## 限定版本
```
# 所安装的包的版本为 2.1.2
pip install pkg==2.1.2

# 所安装的包必须大于等于 2.1.2
pip install pkg>=2.1.2

# 所安装的包必须小于等于 2.1.2
pip install pkg<=2.1.2
```
## 限制不使用二进制包安装
由于默认情况下，wheel 包的平台是运行 pip download 命令 的平台，所以可能出现平台不适配的情况。
比如在 MacOS 系统下得到的 pymongo-2.8-cp27-none-macosx_10_10_intel.whl 就不能在 linux_x86_64 安装。
使用下面这条命令下载的是 tar.gz 的包，可以直接使用 pip install 安装。
比 wheel 包，这种包在安装时会进行编译，所以花费的时间会长一些。
```
# 下载非二进制的包
pip download --no-binary=:all: pkg
#　安装非二进制的包
pip install pkg --no-binary
```
## 延长超时时间
`pip install --default-timeout=100 <packages>`
## 卸载软件
`pip uninstall pkg`
## 升级软件包
想要对现有的 python 进行升级，其本质上也是先从 pypi 上下载最新版本的包，再对其进行安装。所以升级也是使用 pip install，只不过要加一个参数 --upgrade。
`pip install --upgrade pkg`
在升级的时候，其实还有一个不怎么用到的选项 --upgrade-strategy，它是用来指定升级策略。
它的可选项只有两个：
+ eager ：升级全部依赖包
+ only-if-need：只有当旧版本不能适配新的父依赖包时，才会升级。
# 代理
当你身处在一个内网环境中时，无法直接连接公网。这时候你使用pip install 安装包，就会失败。面对这种情况，可以有两种方法：
+ 下载离线包拷贝到内网机器中安装
+ 使用代理服务器转发请求

第一种方法，虽说可行，但有相当多的弊端
+ 步骤繁杂，耗时耗力
+ 无法处理包的依赖问题

第二种方法
`pip install --proxy [user:passwd@]http_server_ip:port pkg`
每次安装包就发输入长长的参数，未免有些麻烦，为此你可以将其写入配置文件中：
`$HOME/.config/pip/pip.conf`

不同的操作系统不同
```
# Linux/Unix:
/etc/pip.conf
~/.pip/pip.conf
~/.config/pip/pip.conf

# Mac OSX:
~/Library/Application Support/pip/pip.conf
~/.pip/pip.conf
/Library/Application Support/pip/pip.conf

# Windows:
%APPDATA%\pip\pip.ini
%HOME%\pip\pip.ini
C:\Documents and Settings\All Users\Application Data\PyPA\pip\pip.conf (Windows XP)
C:\ProgramData\PyPA\pip\pip.conf (Windows 7及以后)
```
配置文件
```
[global]
time-out=60
index-url = http://mirrors.aliyun.com/pypi/simple/

# 替换出自己的代理地址，格式为[user:passwd@]proxy.server:port
proxy=http://xxx.xxx.xxx.xxx:8080

[install]
# 信任阿里云的镜像源，否则会有警告
trusted-host=mirrors.aliyun.com
```






