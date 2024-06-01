---
title: "Linux中Lunacy不遵循系统缩放"
date: 2024-01-13T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Help,Linux]
featuredImagePreview: ""
summary: 编辑 `lunacy.desktop`, `Exec=env AVALONIA_GLOBAL_SCALE_FACTOR=1.25 /opt/icons8/lunacy/Lunacy %u`。然后刷新菜单缓存 `update-desktop-database ~/.local/share/applications/`。
toc:
  enable: false
---

参考 [App not respect system scaling settings](https://community.icons8.com/t/app-not-respect-system-scaling-settings/3290), 编辑 `~/.local/share/applications/lunacy.desktop`, 在 `Exec=` 一行添加 `env AVALONIA_GLOBAL_SCALE_FACTOR=<倍数>`, 比如缩放 `125%`:

```
Exec=env AVALONIA_GLOBAL_SCALE_FACTOR=1.25 /opt/icons8/lunacy/Lunacy %u
```

然后刷新菜单缓存 (该命令的参数是存放已修改过的 `desktop` 文件的目录):

```bash
update-desktop-database ~/.local/share/applications/
```