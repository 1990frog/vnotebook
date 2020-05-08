[TOC]

# 安装包
```
$ sudo pacman -S shadowsocks-libev
$ sudo pacman -S simple-obfs
```

# config
```
{
    "server":"******",
    "server_port":******,
    "local_port":******,
    "password":"******",
    "method":"aes-256-gcm"
}
```

# services
`/usr/lib/systemd/system/ss.services`
```
[Unit]
Description=Shadowsocks-Libev Client Service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=nobody
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
ExecStart=/usr/bin/ss-local -c /etc/shadowsocks/config.json --plugin obfs-local --plugin-opts "obfs=tls"

[Install]
WantedBy=multi-user.target
```