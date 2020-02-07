---
title: "Cheatsheet"
date: 2020-01-27T21:45:37+08:00
single: true
---

{{< expand "`pip`如何使用清华源">}}

### 临时使用

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

注意，simple 不能少, 是 https 而不是 http

### 设为默认

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

```bash
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

如果您到 pip 默认源的网络连接较差，临时使用本镜像站来升级 pip：

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```

{{< /expand >}}

{{< expand "设置`git`使用代理">}}

`git`主要使用两种协议：`HTTPS`和`ssh`。需要分别设置代理。

为`git`设置`HTTPS`代理: 

```bash
git config --global http.proxy "socks5h://127.0.0.1:1080"
# 仅为github设置代理：
git config --global http.https://github.com.proxy socks5h://127.0.0.1:1080
```

为`git`设置`SSH`代理有两种思路方法：

```git_config
Host github.com
    User git
    # 使用nc，ssh走代理
    ProxyCommand nc -v -x 127.0.0.1:1080 %h %p
    # 使用vps作为ssh跳板做代理
    ProxyCommand ssh -q -W %h:%p vps
```

同时，Github支持`SSH over HTTPS`，即：[Using SSH over the HTTPS port](https://help.github.com/en/github/authenticating-to-github/using-ssh-over-the-https-port)。

通过如下设置：

```git_config
Host github.com
    Hostname ssh.github.com
    Port 443
```

`SSH`协议流量走`HTTPS`协议。
再继续设置终端https代理即可。(目前访问`ssh.github.com`这个域名似乎没什么问题)

{{< /expand >}}

{{< expand  "终端设置代理" >}}

```bash
export ALL_PROXY=socks5h://127.0.0.1:1080
```

{{< /expand >}}

{{< expand "更换docker仓库镜像" >}}

from [Registry as a pull through cache](https://docs.docker.com/registry/recipes/mirror/)

> The first time you request an image from your local registry mirror, it pulls the image from the public Docker registry and stores it locally before handing it back to you. On subsequent requests, the local registry mirror is able to serve the image from its own storage.

对于使用`systemd`的系统，编辑`/etc/docker/daemon.json`:

```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://dockerhub.azk8s.cn",
    "https://hub-mirror.c.163.com"
  ]
}
```

接着重启服务:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

就可以用`docker info`查看是否更改。


{{< /expand >}}
