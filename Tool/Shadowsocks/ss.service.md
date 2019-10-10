# ss.service

## ss.service:

```txt
[Unit]
Description=SS Service
After=multi-user.target
 
[Service]
Type=idle
#ExecStart=/usr/bin/python /opt/shadowsocksr/shadowsocks/local.py -c /etc/shadowsocks/config.json
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks/config.json
 
[Install]
```

## config.json

```json
{
    "server":"c17s2.jamjams.net",
    "server_port":9795,
    "local_port":1080,
    "password":"yEvgoSN3T4",
    "method":"aes-256-gcm"
}
```