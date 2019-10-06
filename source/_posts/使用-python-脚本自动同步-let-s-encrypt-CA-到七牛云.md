---
title: 使用 python 脚本自动同步 let's encrypt CA 到七牛云
date: 2019-09-28 11:49:16
tags:
  - ssl
  - let's encrypt
  - 七牛
  - python
thumbnail: https://cdn.axis-studio.org/DSC05401.JPG?imageMogr2/quality/50

---

# 起因

`axis-studio` 域名使用了 `Let's encrypt` 签发的 `wildcard`。好处是免费，坏处是有效期只有三个月。

在服务器上使用了 `acme.sh` 可以实现自动更新。但是网站部分资源使用了七牛云的 `OSS` 服务。由于该服务使用了 `cnd.axis-studio.org` 域名，所以希望每次服务器端的证书更新后，可以同步更新七牛云的证书。

思路有了，接下来就剩下 coding 了。

本来想用 shell 脚本写一个，但最近都在写 python。就尝试着用 python 写写看。

# 开始写码吧

## 环境

- centos 7.2
- python 3.6.8
- `acme.sh` 2.8.1

## 步骤

首先要确保你服务器上的 `acme.sh` 脚本正常工作，这是整个项目顺利进行的前提。

接着我们需要到七牛控制台去获得你的 `access_key` 和 `secret_key`。

然后我们需要找到服务器上 `nginx` 上读取你 `ssl` 证书的 path。

在控制台中输入：

```linux
find / -name fullchain.cer
```

或者：

```
cat /usr/local/nginx/conf/nginx.conf
```

找到 `path`

```
ssl_certificate /your/path/fullchain.cer;
ssl_certificate_key /your/path/*.your.domain.org.key;
```

接着进入到你的用户目录，新建一个目录 `pyss` ， 并创建一个 `venv`, 进入

```
cd

mkdir pyss

python3 -m venv env

. env/bin/activate
```

然后开始写脚本 `py_update_qiniu_ssl.py`

```python
# -*- coding: utf-8 -*-
import qiniu
from qiniu import DomainManager
import sys
import getopt
import time


access_key = '' # 这里填你的 qiniu access_key
secret_key = '' # 这里填写 secret_key

# domain 是你七牛储存绑定的域名
opts, args = getopt.getopt(sys.argv[1:], 'd:p:', ['domain=','path='])

for opt, arg in opts:
    if opt in ('-d', '--domain'):
        domain_name = arg
    elif opt in ('-p', '--path'):
        cert_path = arg


auth = qiniu.Auth(access_key=access_key, secret_key=secret_key)
domain_manager = DomainManager(auth)

# 下面的 path 改成你的 path
private_key = "{0}/*.axis-studio.org.key".format(cert_path)
ca = "{0}/fullchain.cer".format(cert_path)

with open(private_key, 'r') as f:
    private_key_str = f.read()

with open(ca, 'r') as f:
    ca_str = f.read()

ret, info = domain_manager.create_sslcert(
    "{0}/{1}".format(domain_name, time.strftime("%Y-%m-%d", time.localtime())), domain_name, private_key_str, ca_str)

ret, info = domain_manager.put_httpsconf(
    domain_name, ret['certID'], False)

```

最后修改 `acme.sh` 的 `--reloadcmd` 就可以了

```
acme.sh --reload "nginx -s reload && cd /pyss/ && . env/bin/activate && python py_update_qiniu_ssl.py -d your_domain -p you_cert_path"
```

# 其实可以更简单

`acme.sh` 有自己的 `--deploy` [文档](https://github.com/Neilpang/acme.sh/blob/693d692a472e9298c3bf3ee71ffc7d3328451887/deploy/README.md)

# 本文参考

- 封面图：chinajoy 2019 上的 [伊织萌](https://twitter.com/moe_five)