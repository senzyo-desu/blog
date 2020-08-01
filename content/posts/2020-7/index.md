---
title: "Git Config"
date: 2020-07-11T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,Git]
featuredImagePreview: "/2020-7/featuredImagePreview.svg"
summary: 记录 `git config` 常用命令。
---

## 配置级别

`git config`有三个级别: **system**、**global** (用户, 全局生效) 和**local** (用户, 当前仓库), 优先级为**system**>**global**>**local**, 分别使用`--system/global/local`参数来指定级别。

- 查看系统 (system) 的git config: 

    ```shell
    git config --system --list
    ```

- 查看用户全局 (global) 的git config: 

    ```shell
    git config --global --list
    ```

- 查看当前仓库 (local) 的git config: 

    ```shell
    git config --local --list
    ```

{{< admonition type=note title="不使用参数时" open=true >}}
如果不使用`--system/global/local`参数, 直接使用`git config --list`命令, 则输出system+global+local的git config; 如果当前路径不是git仓库, 则只输出system+global的git config。
{{< /admonition >}}

## 配置用户

```shell
git config --global user.name "your_name"
git config --global user.email "your_email@example.com"
```

## 生成SSH

```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

如果你使用的是不支持Ed25519算法的旧系统, 请使用以下命令: 

```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

{{< admonition type=info title="注意" open=true >}}
这里的xxxxx@xxxxx.com只是生成的sshkey的名称, 并不约束或要求具体命名为某个邮箱。

现网的大部分教程均讲解的使用邮箱生成, 其一开始的初衷仅仅是为了便于辨识所以使用了邮箱。
{{< /admonition >}}

### 测试连通性

在Git仓库托管处 (GitHub、GitLab或Gitee等) 设置好刚才生成的公钥, 测试连通性: 

```shell
ssh -T git@github.com
or
ssh -T git@gitlab.example.com
or
ssh -T git@gitee.com
```

## 其他设置

### 行尾结束符

行尾结束符, 即EOL(End of Line)。

Linux和MacOS使用LF的系统, 设置为`input`: 

```shell
git config --global core.autocrlf input
```

Windows这个使用CRLF的, 设置为`true`: 

```shell
git config --global core.autocrlf true
```

设置文件名大小写敏感: 

```shell
git config --global core.ignorecase false
```

设置默认分支: 

```shell
git config --global init.defaultBranch master
```

输入错误命令时发出提示, 并等待3秒: 

```shell
git config --global help.autocorrect 30
```

### 端口号

由于有些学校、企业、机场会屏蔽SSH默认端口号22, 可以修改`~/.ssh/config`文件以指定端口号, 比如改用443端口:

```
# Github
Host github.com
    HostName ssh.github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github/id_ed25519
    Port 443
    User git

# Github Gist
Host gist.github.com
    HostName ssh.github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github/id_ed25519
    Port 443
    User git

# Gitlab
Host gitlab.com
    HostName altssh.gitlab.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitlab/id_ed25519
    Port 443
    User git
```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [Git - git-config Documentation](https://git-scm.com/docs/git-config)
2. [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
3. [生成/添加SSH公钥](https://gitee.com/help/articles/4181#article-header0)
4. [Git配置多个SSH-Key](https://gitee.com/help/articles/4229#article-header0)
5. [Using SSH over the HTTPS port](https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)
6. [GitLab.com now supports an alternate git+ssh port](https://about.gitlab.com/blog/2016/02/18/gitlab-dot-com-now-supports-an-alternate-git-plus-ssh-port/)
{{< /admonition >}}