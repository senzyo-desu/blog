---
title: "快速下载Snap软件"
date: 2022-04-10T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Linux,Download,Snap]
featuredImagePreview: "/2022-14/featuredImagePreview.svg"
summary: 通过 curl 得到返回链接, 手动下载并安装相应的 snap 软件, 可以作为快速下载 snap 软件的一种妥协方法。
toc:
  enable: false
---

在云服务器上使用snap时, 遇到了这样的问题: 

1. `snap install`的下载速度太慢。
2. snap没有镜像站。
3. [https://uappexplorer.com/snaps](https://uappexplorer.com/snaps)已经不能用了。
4. 运行环境不能够 (不敢) 使用代理。

参考Ask Ubuntu上的[How to get snap download url](https://askubuntu.com/a/1196449): 

1. 通过[snapcraft.io](https://snapcraft.io/)页面找到需要的snap包的名称, 以**Redis Desktop Manager**为例, 其`PackageName`是**redis-desktop-manager**。

2. 使用一个`HTTP Headers`中带有`Snap-Device-Series`的`GET`请求`api.snapcaft.io`获取相应snap包的下载链接, 比如: 

    ```bash
    curl -H 'Snap-Device-Series: 16' http://api.snapcraft.io/v2/snaps/info/redis-desktop-manager
    ```

    得到返回信息如下: 

    ```json
    {
        "channel-map": [{
            "channel": {
                "architecture": "amd64",
                "name": "stable",
                "released-at": "2022-04-11T08:42:55.242579+00:00",
                "risk": "stable",
                "track": "latest"
            },
            "created-at": "2022-04-07T06:42:10.696195+00:00",
            "download": {
                "deltas": [],
                "sha3-384": "bd46915580ab10393d290efd83d3bad25ce62afa8e60f72eb46b637c487ce56e027f82997d2c077a7f2f588175491cb4",
                "size": 274452480,
                "url": "https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_617.snap"
            },
            "revision": 617,
            "type": "app",
            "version": "2022.2+c787819a"
        }, {
            "channel": {
                "architecture": "arm64",
                "name": "stable",
                "released-at": "2019-05-08T16:06:17.202784+00:00",
                "risk": "stable",
                "track": "latest"
            },
            "created-at": "2018-11-02T13:22:00.292564+00:00",
            "download": {
                "deltas": [],
                "sha3-384": "da6611902f6e6e30f91c9f0e59148d5caf3c528b8f4ce0ca4927e9c46c9347bcb36472505568b0975e371eb6f7481ccb",
                "size": 405401600,
                "url": "https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_180.snap"
            },
            "revision": 180,
            "type": "app",
            "version": "0.9.8+git"
        }, {
            "channel": {
                "architecture": "armhf",
                "name": "stable",
                "released-at": "2019-05-08T16:06:18.459411+00:00",
                "risk": "stable",
                "track": "latest"
            },
            "created-at": "2018-11-23T11:25:27.569032+00:00",
            "download": {
                "deltas": [],
                "sha3-384": "db1b4288e4ef113427a529ed8c0c7eeb9a880c21d56d569e9345d5fd0d1562d4b24690692b68b1c3df8857f6c3b78fec",
                "size": 402309120,
                "url": "https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_187.snap"
            },
            "revision": 187,
            "type": "app",
            "version": "0.9.8+git"
        }, {
            "channel": {
                "architecture": "i386",
                "name": "stable",
                "released-at": "2019-05-08T16:06:17.796650+00:00",
                "risk": "stable",
                "track": "latest"
            },
            "created-at": "2018-11-23T11:24:37.772099+00:00",
            "download": {
                "deltas": [],
                "sha3-384": "07bc1be5689009fecb82fadc6f3359e9e4923817c5cfcca8e0fb27821b4366bcee9c54b1b91e70b2773d9c620cb2b8ea",
                "size": 413761536,
                "url": "https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_186.snap"
            },
            "revision": 186,
            "type": "app",
            "version": "0.9.8+git"
        }, {
            "channel": {
                "architecture": "amd64",
                "name": "edge",
                "released-at": "2022-05-31T07:31:31.925793+00:00",
                "risk": "edge",
                "track": "latest"
            },
            "created-at": "2022-05-31T07:29:29.187719+00:00",
            "download": {
                "deltas": [],
                "sha3-384": "95eb6cea54334d8337c5014c7d34a8793c50450c578946055a13bed1612eb3f8a6c7cded1918d7291c3b8b30e60e4d32",
                "size": 274501632,
                "url": "https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_625.snap"
            },
            "revision": 625,
            "type": "app",
            "version": "2022.3+8bd6667f"
        }, {
            "channel": {
                "architecture": "arm64",
                "name": "edge",
                "released-at": "2018-11-02T13:27:43.086596+00:00",
                "risk": "edge",
                "track": "latest"
            },
            "created-at": "2018-11-02T13:22:00.292564+00:00",
            "download": {
                "deltas": [],
                "sha3-384": "da6611902f6e6e30f91c9f0e59148d5caf3c528b8f4ce0ca4927e9c46c9347bcb36472505568b0975e371eb6f7481ccb",
                "size": 405401600,
                "url": "https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_180.snap"
            },
            "revision": 180,
            "type": "app",
            "version": "0.9.8+git"
        }, {
            "channel": {
                "architecture": "armhf",
                "name": "edge",
                "released-at": "2018-11-23T11:40:13.961025+00:00",
                "risk": "edge",
                "track": "latest"
            },
            "created-at": "2018-11-23T11:25:27.569032+00:00",
            "download": {
                "deltas": [],
                "sha3-384": "db1b4288e4ef113427a529ed8c0c7eeb9a880c21d56d569e9345d5fd0d1562d4b24690692b68b1c3df8857f6c3b78fec",
                "size": 402309120,
                "url": "https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_187.snap"
            },
            "revision": 187,
            "type": "app",
            "version": "0.9.8+git"
        }, {
            "channel": {
                "architecture": "i386",
                "name": "edge",
                "released-at": "2018-11-23T11:36:57.430143+00:00",
                "risk": "edge",
                "track": "latest"
            },
            "created-at": "2018-11-23T11:24:37.772099+00:00",
            "download": {
                "deltas": [],
                "sha3-384": "07bc1be5689009fecb82fadc6f3359e9e4923817c5cfcca8e0fb27821b4366bcee9c54b1b91e70b2773d9c620cb2b8ea",
                "size": 413761536,
                "url": "https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_186.snap"
            },
            "revision": 186,
            "type": "app",
            "version": "0.9.8+git"
        }],
        "default-track": null,
        "name": "redis-desktop-manager",
        "snap": {
            "license": "GPL-3.0",
            "name": "redis-desktop-manager",
            "prices": {},
            "publisher": {
                "display-name": "Igor Malinovskiy",
                "id": "EYGqoNaJFRPwVKQstqp8WYex02yg31Wj",
                "username": "uglide",
                "validation": "unproven"
            },
            "snap-id": "Iw3a6EauULwaud5DO0ixtrJg8o6VXaey",
            "store-url": "https://snapcraft.io/redis-desktop-manager",
            "summary": "RESP.app - GUI for Redis \u00ae (formerly RedisDesktopManager)",
            "title": "redis-desktop-manager"
        },
        "snap-id": "Iw3a6EauULwaud5DO0ixtrJg8o6VXaey"
    }
    ```

3. 从返回信息中找到对应平台的下载链接, 使用命令下载: 

    ```bash
    wget https://api.snapcraft.io/api/v1/snaps/download/Iw3a6EauULwaud5DO0ixtrJg8o6VXaey_617.snap
    ```

4. 下载完成后安装即可:

    ```bash
    sudo snap install xxx.snap --dangerous
    ```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [Snap info](https://api.snapcraft.io/docs/info.html)
2. [How to get snap download url](https://askubuntu.com/a/1196449)
{{< /admonition >}}
