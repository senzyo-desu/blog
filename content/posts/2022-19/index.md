---
title: "Twikoo的自建后端与自动更新"
date: 2022-10-11T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Blog,Linux]
series: [Deploy Blog on Own Server]
series_weight: 2
featuredImagePreview: ""
summary: 使用 systemd 管理 twikoo 后端的运行。通过 crontab 定时运行脚本, 检测当前版本与最新版本是否一致, 如不一致, 则更新到最新版本, 并 push 到 Git Remote 仓库。
---

## 自建twikoo后端

1. 全局安装twikoo: 

    ```bash
    sudo npm i -g tkserver@latest
    ```

2. 创建twikoo工作目录: 

   ```bash
   mkdir /home/senzyo/project/twikoo/
   ```

   twikoo会在此目录下生成`data`文件夹存放数据文件`db.json*`。`db.json.0`存储具体评论, `db.json.1`存储twikoo前端控制面板的配置, `db.json.2`存储文章属性。

3. 创建`/etc/systemd/system/twikoo.service`并写入以下内容: 

    ```
    [Unit]
    Description=Twikoo

    [Service]
    Type=simple
    WorkingDirectory=/home/senzyo/project/twikoo
    ExecStart=nohup tkserver
    Restart=on-failure
    RestartSec=10s
    ExecStop=/bin/kill $MAINPID

    [Install]
    WantedBy=multi-user.target
    ```

4. 重新加载守护程序: 

    ```bash
    sudo systemctl daemon-reload
    ```

5. 设置`twikoo.service`自启动并立刻启动: 

    ```bash
    sudo systemctl enable --now twikoo.service
    ```

6. 检查运行状态: 

    ```bash
    sudo systemctl status twikoo.service
    ```

7. 注意`/home/senzyo/project/twikoo`的权限归属, 最好是分配给当前普通用户。

    ```bash
    sudo chown -R senzyo:senzyo /home/senzyo/project/twikoo
    ```

## 自动更新后端和前端

首先参考[Nginx部署Hugo站点](../2022-8/)设置一系列仓库, 然后按照以下步骤操作: 

1. 创建更新脚本`/home/senzyo/project/twikoo/upgrade.sh`并写入以下内容: 

    ```bash
    #!/bin/bash
    
    PROD="/home/senzyo/project/blog"
    WORK="/home/senzyo/project/twikoo"
    
    LATEST=$(npm view tkserver version)
    LOCAL=$(npm ls tkserver -g | grep @ | cut -d '@' -f 2)
    
    if [ "$LATEST" != "$LOCAL" ]; then
        echo "==========更新Twikoo前端=========="
        cd $WORK
        wget -O - https://mirrors.cloud.tencent.com/npm/twikoo/-/twikoo-$LATEST.tgz | tar -zxf -
        mv package/dist/twikoo.all.min.js $PROD/themes/DoIt/assets/lib/twikoo/twikoo.all.min.js
        rm -rf package
    
        echo "==========正在提交到Git仓库=========="
        cd $PROD
        git pull
        git add -A
        git commit -m "更新Twikoo至$LATEST"
        git push -uf origin master
    
        echo "==========更新Twikoo后端=========="
        systemctl stop twikoo.service
        npm i -g tkserver@latest
        systemctl start twikoo.service
    else
        echo "==========当前Twikoo已是最新版本$LATEST=========="
    fi
    ```

2. 授予可执行权限: 

    ```bash
    chmod +x /home/senzyo/project/twikoo/upgrade.sh
    ```

3. `sudo crontab -e`设置crontab, 添加以下内容: 

    ```bash
    0 1 * * 1 flock -xn /tmp/twikoo-upgrade.lock -c '/home/senzyo/project/twikoo/upgrade.sh > /dev/null 2>&1 &'
    ```

    {{< admonition type=note title="注意" open=true >}}
这里要为`root`用户创建定时任务而不是为普通用户创建, 因为`upgrade.sh`中需要通过`systemd`管理twikoo后端服务, 普通用户会因权限不足而不能执行这一过程。
    {{< /admonition >}}

4. 由于[Nginx部署Hugo站点](../2022-8/#服务器中的remote仓库)中设置的`.git/hooks/post-receive`在接收到`push`后会在`/home/senzyo/project/blog`自动构建Hugo站点, 所以闭环成功🎉。
