---
title: 腾讯云域名使用acme.sh签发letsencrypt的wildcard
date: 2019-04-05 23:41:36
tags:
  - SSL
  - 腾讯云
  - letsencrypt
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/AF9D13C8-BFDC-4111-930C-5D34A98DDF2F.jpg
---

之前薅阿里云羊毛的小🐔到期了，续费金额着实高贵，续不起。无奈转投腾讯云。包括万网域名转移、域名重新备案，cdn 迁移。这些都还顺利，最后在 issue letsencrypt ssl 证书的时候踩了一波坑。折腾了一天。<!--more-->

## 环境

- 腾讯云主机 S2
- CentOS 7.2 
- `acme.sh` v2.8.1
- nginx 1.14.1

## 问题复现

当使用 `acme.sh` 的 http 方式来 issue 单个域名的证书时，没有问题。但我有好几个二级域名，为每个域名单独 issue 证书，并且还要修改 nginx 配置，实在麻烦。

所以我尝试使用 `acme.sh` 的 http 方式来 issue wildcard 通配符证书，结果报错了，报错内容大致的意思时 **申请 wildcard 需要使用 dns 方式**。

ok ，找到 dnsapi 的[文档](https://github.com/Neilpang/acme.sh/wiki/dnsapi), 但在给出的文档里并没有发现支持腾讯云的 api 。

尝试使用 dns alias mode，但发现 alias mode 不支持自动 renew 证书，这意味着每三个月就要手动去做更新，太蠢了。

emmm，貌似陷入了僵局。

## 尝试解决问题

在 `acme.sh` 的 issue 里找关于腾讯云域名 dns 的问题。发现一位老哥的 [issue](https://github.com/Neilpang/acme.sh/issues/1904) 里提到了腾讯云，但他用的服务商 api 的参数写了 dns_dp（ DNSPod ） 。好奇心作祟，以 tencent 和 DNSPod 作为关键词查了一波。

结果，发现了 [腾讯为何要收购DNSPod？ - 知乎](https://www.zhihu.com/question/20176201)。

我枯辽。

原来是一家。

那问题就简单了。

直接

```shell
export DP_Id="1234"
export DP_Key="sADDsdasdgdsf"

acme.sh --issue --dns dns_dp -d example.com -d www.example.com
```

这里的 id 和 key 都需要登录 [dnspod 官网](https://www.dnspod.cn/Login?r=/console) 去申请，注意不是 **腾讯云官网**，腾讯云也有申请 api 密钥的地方，服务器和域名的服务是分开的。

这样不出意外就可以成功 issue wildcard 了。

![](https://cdn.axis-studio.org/wildcard.png)

## 本文参考

- thumbnail: 农大新区正门
- [acme.sh/issues](https://github.com/Neilpang/acme.sh/issues)



  
