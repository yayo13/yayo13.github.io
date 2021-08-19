---
layout: post
title: ubuntu上使用clash
categories: [linux]
description: 
keywords: 
---

<style>
.img_txt{text-align:center}
</style>

# 下载clash
 
  [Dreamacro项目][1]下载最新clash linux 版本，将其解压缩到clash/目录下
 
# 配置

## 下载config_tmp.yaml 和 Country.mmdb

```bash
cd clash/
# [url]为机场给的链接
wget -O config_tmp.yaml [url]

wget -O Country.mmdb <https://www.sub-speeder.com/client-download/Country.mmdb>
```

## 生成config.yaml

上一步得到的config_tmp.yaml与clash不一定兼容，需要以[clash官方][2]提供的为模板，将订阅信息更新进去

```bash
## config.yaml 模板
# HTTP(S) and SOCKS5 server on the same port
mixed-port: 1090

# Set to true to allow connections to the local-end server from
# other LAN IP addresses
allow-lan: false

# This is only applicable when `allow-lan` is `true`
# '*': bind all IP addresses
# 192.168.122.11: bind a single IPv4 address
# "[aaaa::a8aa:ff:fe09:57d8]": bind a single IPv6 address
bind-address: '*'

# Clash router working mode
# rule: rule-based packet routing
# global: all packets will be forwarded to a single endpoint
# direct: directly forward the packets to the Internet
mode: rule

# Clash by default prints logs to STDOUT
# info / warning / error / debug / silent
log-level: debug

# When set to false, resolver won't translate hostnames to IPv6 addresses
ipv6: false

# RESTful web API listening address
external-controller: 127.0.0.1:9090

# Static hosts for DNS server and connection establishment (like /etc/hosts)
#
# Wildcard hostnames are supported (e.g. *.clash.dev, *.foo.*.example.com)
# Non-wildcard domain names have a higher priority than wildcard domain names
# e.g. foo.example.com > *.example.com > .example.com
# P.S. +.foo.com equals to .foo.com and foo.com
hosts:
  # '*.clash.dev': 127.0.0.1
  # '.dev': 127.0.0.1
  # 'alpha.clash.dev': '::1'

proxies:
  # vmess
  # cipher support auto/aes-128-gcm/chacha20-poly1305/none
  - name: "vmess1"
    type: vmess
    server: cdn-cn.123.cn
    port: 19083
    uuid: a381baf6-1122-3676-a364-8a90219e8b22
    alterId: 0
    cipher: auto
    network: ws
    ws-path: /aaff

  - name: "vmess2"
    type: vmess
    server: cdn-cn.123.cn
    port: 19049
    uuid: a381baf6-1122-3676-a364-8a90219e8b22
    alterId: 0
    cipher: auto
    network: ws
    ws-path: /ddefg

  - name: "vmess3"
    type: vmess
    server: cdn-cn.123.cn
    port: 19050
    uuid: a381baf6-1122-3676-a364-8a90219e8b22
    alterId: 0
    cipher: auto
    network: ws
    ws-path: /cat

  - name: "vmess4"
    type: vmess
    server: cdn-cn.123.cn
    port: 19042
    uuid: a381baf6-1122-3676-a364-8a90219e8b22
    alterId: 0
    cipher: auto
    network: ws
    ws-path: /dog

proxy-groups:
  # select is used for selecting proxy or proxy group
  # you can use RESTful API to switch proxy is recommended for use in GUI.
  - name: "group1"
    type: select
    # disable-udp: true
    proxies:
      - vmess1
      - vmess2

  - name: "group2"
    type: url-test
    proxies:
      - vmess1
      - vmess2
      - vmess3
      - vmess4
    # tolerance: 150
    lazy: true
    url: 'http://www.gstatic.com/generate_204'
    interval: 300

proxy-providers:


rules:
  - DOMAIN-SUFFIX,google.com,group1
  - DOMAIN-SUFFIX,qq.com,REJECT
  - DOMAIN-SUFFIX,cnblogs.com,DIRECT
  - IP-CIDR,127.0.0.0/8,DIRECT
  - IP-CIDR,192.168.0.0/16,DIRECT
  - IP-CIDR,10.0.0.0/8,DIRECT
  - MATCH,group2
```
从wget生成的config_tmp.yaml中，将<mark>proxies</mark>、<mark>proxy-groups</mark>、<mark>rules</mark>信息更新进模板，得到config.yaml

## 配置系统网络

![](/assets/images/articles/20210818_01/01.png)

<div class="img_txt">图1. 系统网络设置</div>

如图，将网络代理设为<mark>手动</mark>

设置**HTTP代理**、**HTTPS代理**、**Socks主机**的IP为`127.0.0.1`，端口分别设置为`1090`、`1090`、`9090`

设置**忽略主机**为`localhost, 127.0.0.0/8, ::1`

## 配置board

终端执行`./clash -f ./config.yaml` 开启clash

浏览器打开 `https://clash.razord.top/` 进入board，一切没问题就可以在board上选择节点，选择全局代理

![](/assets/images/articles/20210818_01/02.png)

<div class="img_txt">图2. clash board设置</div>

## 配置终端

为了让终端也能走proxy，在`~/.bashrc`内加入

```bash
export http_proxy=http://127.0.0.1:1090
export https_proxy=http://127.0.0.1:1090
```



---

[1]: https://github.com/Dreamacro/clash/releases
[2]: https://github.com/Dreamacro/clash/wiki/configuration

