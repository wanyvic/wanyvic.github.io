---
title: 科学上网trojan-go和strongswan-ikev2搭建
tags: vpn
abbrlink: e1da7b5b
date: 2021-10-08 11:48:59
---

# 生成ssl证书
## 首次安装
```
sudo -iu root
apt-get install socat
curl https://get.acme.sh | sh -s email=test1@test.com #安装完成
yourdomain=xxx.xxx.xxx.xxx #你的域名
```
## 签名证书
因为mac、ios系统vpn不支持ec-256证书格式，在此申请RSA证书
```
/root/.acme.sh/acme.sh --issue -d ${yourdomain} --debug --standalone
```
## 证书续期   
写个`autoCa.sh`放入`.acme.sh`文件夹中
```
#!/bin/bash   
yourdomain="xxx.xxx.xxx.xxx"
systemctl stop trojan-web                                       #停掉web服务
/root/.acme.sh/acme.sh --cron --home /root/.acme.sh             #续期
cert_file="/root/.acme.sh/${yourdomain}/${yourdomain}.cer"
key_file="/root/.acme.sh/${yourdomain}/${yourdomain}.key"
ca_file="/root/.acme.sh/${yourdomain}/ca.cer"
cp -f $cert_file /usr/local/etc/ipsec.d/certs/server.cert.pem   #拷贝证书到ipsec目录下
cp -f $key_file /usr/local/etc/ipsec.d/private/server.pem
cp -f $ca_file /usr/local/etc/ipsec.d/cacerts/ca.cert.pem
systemctl start trojan-web                                      #启动trojan web 服务
/usr/local/sbin/ipsec restart                                   #重启  ipsec
```
## 定时任务更新证书
bash输入 `crontab -e` 修改定时任务
```
#每个月2号执行
0 0 2 * * bash /root/.acme.sh/autoCa.sh > /dev/null
```


# 安装 strongswan  
## 复制生成好的证书到当前目录下
```
yourdomain="xxx.xxx.xxx.xxx"
cert_file="/root/.acme.sh/${yourdomain}_ecc/${yourdomain}.cer"
key_file="/root/.acme.sh/${yourdomain}_ecc/${yourdomain}.key"
ca_file="/root/.acme.sh/${yourdomain}_ecc/ca.cer"
cp -f $cert_file server.cert.pem
cp -f $key_file server.pem
cp -f $cert_file client.cert.pem
cp -f $key_file client.pem
cp -f $ca_file ca.cert.pem
```
## 安装 strongswan
```
source <(curl -sL https://raw.githubusercontent.com/wanyvic/one-key-ikev2-vpn/master/one-key-ikev2.sh)
```

提示: Would you want to import existing cert? You NEED copy your cert file to the same directory of this script 选 `yes`

## 配置 ikev2密码

```
vim /usr/local/etc/ipsec.secrets
```


# trojan-go
## 安装Jrohy的一键trojan面板脚本
```
source <(curl -sL https://git.io/trojan-install)
```
## 删除
```
source <(curl -sL https://git.io/trojan-install) --remove
```
## 切换 trojan-go
```
trojan
# 交互输入1 和 6
1
6
```
或通过`{https://yourdomain}`打开网页手动切换内核
## 更改trojan-go配置文件以支持websocket


```
vim /usr/local/etc/trojan/config.json
``` 

```
#在mysql后面追加
"websocket": {
        "enabled": true,
        "path": "/DFE4545DFDED/", # ws路径
        "host": "你的域名"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    }
```

```
trojan restart #重启trojan
```

## 更改trojan-go配置的证书路径
安装脚本默认生成ec-256证书，在此不使用ec-256证书。使用之前生成的RSA证书代替  
```
#删除证书路径中的ec-256证书
rm -rf /root/.acme.sh/${yourdomain}_ecc
sed -i "s/${yourdomain}_ecc/${yourdomain}/g" /usr/local/etc/trojan/config.json
```
# 客户端
## ipsec
ipsec 直接使用系统ikev2 vpn连接即可。
## trojan-go
GUI使用[Qv2ray](https://qv2ray.net/) + [trojan-go 内核程序](https://github.com/p4gefau1t/trojan-go/releases) + [trojan-go qv2ray 插件](https://github.com/Qv2ray/QvPlugin-Trojan-Go/releases)

[Qv2ray插件文档](http://qv2ray.net/lang/zh/plugins/usage.html)
### 运作
- Qv2ray 设置`sock5/http`代理到本地

- 浏览器使用`SwitchyOmega`插件连接本地代理端口

- 终端使用
设置环境变量  
```
export http_proxy=http://127.0.0.1:8082
export https_proxy=http://127.0.0.1:8082
# 或者
# export http_proxy="socks5://127.0.0.1:1080"
# export https_proxy="socks5://127.0.0.1:1080"
```
