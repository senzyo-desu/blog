---
title: "Docker和Nftables"
date: 2022-10-02T12:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Debian,Docker,Linux,Nftables]
featuredImagePreview: ""
summary: 在 Debian 系 Linux 中安装 Docker。禁止 Docker 使用 IPtables, 改用 Nftables。
---

## Docker

### 安装

参考[Install Docker on Debian](https://docs.docker.com/engine/install/debian/)和[镜像加速器-Docker-从入门到实践](https://yeasy.gitbook.io/docker_practice/install/debian): 

安装必要组件: 

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

设置Docker源: 

```bash
# 腾讯云镜像源
curl -fsSL https://mirrors.cloud.tencent.com/docker-ce/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
# 国内网络不推荐使用官方源
# curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://mirrors.cloud.tencent.com/docker-ce/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

安装Docker: 

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

查看Docker版本: 

```bash
docker version
```

将某一普通用户添加到Docker用户组中: 

```bash
sudo usermod -aG docker <UserName>
```

设置Docker开机自启: 

```bash
sudo systemctl enable docker.service
```

### 镜像源

参考[镜像加速器-Docker-从入门到实践](https://yeasy.gitbook.io/docker_practice/install/mirror): 

编辑文件`/etc/docker/daemon.json`并添加以下内容, 如不存在就新建: 

```json
{
   "registry-mirrors": [
       "https://mirror.ccs.tencentyun.com"
  ]
}
```

重启Docker: 

```bash
sudo systemctl restart docker.service
```

执行`docker info`, 如结果中有镜像源地址, 则说明配置成功。

```
Registry Mirrors:
 https://mirror.ccs.tencentyun.com/
```

## 禁止Docker使用iptables

编辑文件`/etc/docker/daemon.json`并添加以下内容: 

```json
{
   "iptables":false
}
```

重启Docker后生效: 

```bash
sudo systemctl restart docker.service
```

但这样会使docker很多地方都会受到影响, 甚至会出现不能用的情况。

### 解决方法

参考[Nftables#Working with Docker](https://wiki.archlinux.org/title/Nftables#Working_with_Docker): 

> 一种可靠的方法是让docker在单独的网络名称空间中运行, 在那里它可以做任何它想做的事情。

编辑`/etc/systemd/system/docker.service.d/netns.conf`并写入以下内容: 

```
[Service]
PrivateNetwork=yes

# cleanup
ExecStartPre=-nsenter -t 1 -n -- ip link delete docker0

# add veth
ExecStartPre=nsenter -t 1 -n -- ip link add docker0 type veth peer name docker0_ns
ExecStartPre=sh -c 'nsenter -t 1 -n -- ip link set docker0_ns netns "$$BASHPID" && true'
ExecStartPre=ip link set docker0_ns name eth0

# bring host online
ExecStartPre=nsenter -t 1 -n -- ip addr add 10.0.0.1/24 dev docker0
ExecStartPre=nsenter -t 1 -n -- ip link set docker0 up

# bring ns online
ExecStartPre=ip addr add 10.0.0.100/24 dev eth0
ExecStartPre=ip link set eth0 up
ExecStartPre=ip route add default via 10.0.0.1 dev eth0
```

如果`10.0.0.*`IP地址不适合你的设置, 请调整它们。

在nftables的nat表的postrouting链中添加以下规则为`docker0`启用IP转发并设置NAT: 

```bash
iifname docker0 oifname eth0 masquerade
```

然后, 确保[Enable packet forwarding](https://wiki.archlinux.org/title/Internet_sharing#Enable_packet_forwarding)。

现在, 你可以使用nftables为`docker0`接口设置防火墙和端口转发, 而不会产生任何干扰。

--------------------

{{< admonition type=quote title="更多文档" open=true >}}
1. [Docker Documentation](https://docs.docker.com/)
2. [Docker-从入门到实践](https://yeasy.gitbook.io/docker_practice/)
{{< /admonition >}}
