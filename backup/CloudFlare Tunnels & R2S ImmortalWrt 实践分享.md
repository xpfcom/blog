## cloudflare：点击Zero Trust→点击Networks→点击Tunnels→点击+Create a tunnel→选择Cloudflared→点击Next
_________________________________

## 隧道名称：Tunnel 1

#### 1、隧道握手码（备注）：
```
++eyJhIjoiNWVjMDk2OGYyNTUzMmJhZDg4ODRhZTI5ZjhNThjNjAiLCJ0IjoiNTNmZWEzOGEtNDMwMy00YzgzLThkNDMtNzE3ZTlkYTNjYjI3IiwicyI6IlpUWTFNMlJoWkRRdFpEVmtOUzAwTVRFMkxXRmhaVGt0WmpKbU1UUXhOMk5tTnpOayJ9++
```

_________________________________

#### 2、OpenWrt安装程序
项目地址：https://github.com/cloudflare/cloudflared/releases/（如更新 替换下方代码日期）
```
VERSION="2024.6.1"
curl -O -L \
https://github.com/cloudflare/cloudflared/releases/download/${VERSION}/cloudflared-linux-arm64 \
&& chmod +x cloudflared-linux-arm64 \
&& mv cloudflared-linux-arm64 /usr/bin/cloudflared
```

_________________________________

#### 3、创建文件 赋予可执行权限
```
touch /etc/init.d/cloudflared
chmod +x /etc/init.d/cloudflared
```

_________________________________

#### 4、SSH替换 /ect/init.d/cloudflare 文件代码
```
#!/bin/sh /etc/rc.common
USE_PROCD=1
START=95
STOP=01
cfd_init="/etc/init.d/cloudflared"
cfd_token="++eyJhIjoiNWVjMDk2OGYyNTUzMmJhZDg4ODRhZTI5ZjhNThjNjAiLCJ0IjoiNTNmZWEzOGEtNDMwMy00YzgzLThkNDMtNzE3ZTlkYTNjYjI3IiwicyI6IlpUWTFNMlJoWkRRdFpEVmtOUzAwTVRFMkxXRmhaVGt0WmpKbU1UUXhOMk5tTnpOayJ9++"
boot()
{
ubus -t 30 wait_for network.interface network.loopback 2>/dev/null
rc_procd start_service
}
start_service() {
if [ $("${cfd_init}" enabled; printf "%u" ${?}) -eq 0 ]
then
procd_open_instance
procd_set_param command /usr/bin/cloudflared --no-autoupdate tunnel run --token ${cfd_token}
procd_set_param stdout 1
procd_set_param stderr 1
procd_set_param respawn ${respawn_threshold:-3600} ${respawn_timeout:-5} ${respawn_retry:-5}
procd_close_instance
fi
}
stop_service() {
pidof cloudflared && kill -SIGINT `pidof cloudflared`
}
```

_________________________________

##### 5、设置 cloudflared 文件开机启动 运行 cloudflared 文件
```
/etc/init.d/cloudflared enable
/etc/init.d/cloudflared start
```