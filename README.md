## Xray VLESS-TCP-XTLS-Vision 带有回落 (fallbacks) 手动安装指南

已有SSL证书

- 如果你之前用acme申请了SSL证书，将证书文件改名为`fullchain.cer`，将私钥文件改名为`private.key`，使用WinSCP登录你的VPS，将它们上传到`/etc/ssl/private`目录，执行下面的命令。

```
chown -R nobody:nogroup /etc/ssl/private
```

- [使用证书时权限不足](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates-zh-Hans-CN)

#### 用[acme](https://github.com/acmesh-official/acme.sh)申请SSL证书

- 你先要购买一个域名，然后添加一个子域名，将子域名指向你VPS的IP。等待5-10分钟，让DNS解析生效。你可以通过ping你的子域名，查看返回的IP是否正确。确认DNS解析生效后，再执行下面的命令（每行命令依次执行）。
- 注意：将`chika.example.com`替换成你的子域名。

<details><summary>点击查看详细步骤</summary> 

```
apt install -y socat
```

```
curl https://get.acme.sh | sh
```

```
alias acme.sh=~/.acme.sh/acme.sh
```

```
acme.sh --upgrade --auto-upgrade
```

```
acme.sh --set-default-ca --server letsencrypt
```

```
acme.sh --issue -d chika.example.com --standalone --keylength ec-256
```

```
acme.sh --install-cert -d chika.example.com --ecc \
```

```
--fullchain-file /etc/ssl/private/fullchain.cer \
```

```
--key-file /etc/ssl/private/private.key
```

```
chown -R nobody:nogroup /etc/ssl/private/
```

</details>

- 备份已申请的SSL证书：使用WinSCP登录你的VPS，进入`/etc/ssl/private`目录，下载证书文件`fullchain.cer`和私钥文件`private.key`。
- SSL证书有效期是90天，每隔60几天会自动更新。[速率限制](https://letsencrypt.org/zh-cn/docs/rate-limits/)，超过次数会报错。

1. 安装[Xray](https://github.com/XTLS/Xray-core/releases)

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

2. 安装[Nginx](http://nginx.org/en/linux_packages.html)

- Debian 10/11

```
apt install -y gnupg2 ca-certificates lsb-release debian-archive-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring.gpg && printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" > /etc/apt/sources.list.d/nginx.list && printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900" > /etc/apt/preferences.d/99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && printf "[Service]\nExecStartPost=/bin/sleep 0.1" > /etc/systemd/system/nginx.service.d/override.conf
```

- Ubuntu 18.04/20.04

```
apt install -y gnupg2 ca-certificates lsb-release ubuntu-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring.gpg && printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" > /etc/apt/sources.list.d/nginx.list && printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900" > /etc/apt/preferences.d/99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && printf "[Service]\nExecStartPost=/bin/sleep 0.1" > /etc/systemd/system/nginx.service.d/override.conf
```

3. 下载[配置](https://github.com/chika0801/Xray-examples/tree/main/VLESS-TCP-XTLS-Vision)

```
curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS-Vision/config_server_with_fallbacks.json && curl -Lo /etc/nginx/nginx.conf https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS-Vision/nginx.conf
```

4. 启动程序

```
systemctl daemon-reload && systemctl restart xray && systemctl restart nginx
```

```
systemctl status xray && systemctl status nginx
```

| 项目 | |
| :--- | :--- |
| 程序 | /usr/local/bin/xray |
| 配置 | /usr/local/etc/xray/config.json |
| 检查 | xray -test -config /usr/local/etc/xray/config.json |
| 路由规则文件 | /usr/local/share/xray/ |
| 查看日志 | journalctl -u xray --output cat -e |
| 实时日志 | journalctl -u xray --output cat -f |


## v2rayN 6.x 配置指南

`服务器` ——> `添加[VLESS服务器]`

![1](https://user-images.githubusercontent.com/88967758/213372857-49306ebe-f2fc-4426-91df-fd54e096456a.jpg)

## v2rayN 5.x 配置指南

`服务器` ——> `添加[VLESS服务器]`

![1](https://user-images.githubusercontent.com/88967758/212540248-043ab1ed-af87-4e48-87b7-895018f4a52d.jpg)

## v2rayNG 配置指南

| 选项 | 值 |
| :--- | :--- |
| 地址(address) | chika.example.com |
| 端口(prot) | 443 |
| 用户ID(id) | chika |
| 流控(flow) | xtls-rprx-vision |
| 传输协议(network) | tcp |
| 传输层安全(tls) | tls |
| SNI | 留空 |
| uTLS | chrome |
