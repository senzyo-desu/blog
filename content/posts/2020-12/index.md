---
title: "自定义GRUB"
date: 2020-07-20T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,Linux]
featuredImagePreview: ""
summary: 自定义 GRUB 主题和其他设置。
---

{{< admonition type=tip title="提醒" open=true >}}
每次编辑 `/etc/default/grub` 后都需要运行 `grub-mkconfig -o /boot/grub/grub.cfg` 命令重新生成 GRUB 主配置文件。
{{< /admonition >}}

## 主题

从 [pling](https://www.pling.com/browse?cat=109&ord=rating) 下载主题压缩包, 比如 [Vimix](https://www.pling.com/p/1009236)。

### 自定义界面

下面是 [Vimix](https://www.pling.com/p/1009236) 主题的 `theme.txt` 文件, 被我稍微改动了一些, 可供参考: 

```
# GRUB2 gfxmenu Linux theme
# Designed for any resolution

# Global Property
title-text: ""
desktop-image: "background.jpg"
desktop-color: "#000000"
terminal-font: "Sarasa Mono SC Regular 16"
terminal-box: "terminal_box_*.png"
terminal-left: "0"
terminal-top: "0"
terminal-width: "100%"
terminal-height: "100%"
terminal-border: "0"

# Show the boot menu
+ boot_menu {
left = 34%
top = 25%
width = 32%
height = 50%
item_font = "Sarasa Mono SC Regular 22"
item_color = "#cccccc"
selected_item_color = "#ffffff"
icon_width = 32
icon_height = 32
item_icon_space = 10
item_height = 50
item_padding = 5
item_spacing = 10
selected_item_pixmap_style = "select_*.png"
}

# Show a countdown message using the label component
+ label {
top = 80%
left = 35%
width = 30%
align = "center"
id = "__timeout__"
text = "将在 %d 秒钟后启动"
color = "#cccccc"
font = "Sarasa Mono SC Regular 16"
}
```

### 自定义字体

有些主题使用默认的 `Unifont` 字体, 即 `/usr/share/grub/unicode.pf2`, 它是 `grub` 包自带的 pf2 文件, 使用 `pacman -Ql grub | grep pf2` 可以查找更多。

要么使用自带的 pf2 字体, 要么自己转换指定的 ttf 文件为 pf2 文件以供 GRUB 使用: 

```bash
grub-mkfont -s 16 -o <font.pf2> <font.ttf>
grub-mkfont -s 22 -o <font.pf2> <font.ttf>
```

`-s` 参数指定字号大小。每个 pf2 字体文件的大小是固定的, 所以需要为要使用的字号分别生成相应的文件。

将生成的 pf2 字体文件放入主题文件夹下。编辑 `theme.txt` 文件内的字体选项: 

```
...
terminal-font: "Sarasa Mono SC Regular 16"
...
item_font = "Sarasa Mono SC Regular 22"
...
font = "Sarasa Mono SC Regular 16"
...
```

注意, 这里填写的是字体的 family 以及规格和大小。

查看字体的 family:

```bash
fc-scan <font.ttf> --format='%{family}\n'
```

### 安装

1. 将主题文件夹放入 `/usr/share/grub/themes/` 路径下。
2. `sudo vim /etc/default/grub` 指定主题路径, 比如: 

   ```
   GRUB_THEME="/usr/share/grub/themes/vimix/theme.txt"
   ```

3. 重新生成 `grub.cfg`: 

   ```bash
   sudo grub-mkconfig -o /boot/grub/grub.cfg
   ```

## 默认启动上次的启动项

`sudo vim /etc/default/grub`, 设置以下选项: 

```
GRUB_DEFAULT=saved
...
GRUB_SAVEDEFAULT=true
```

### 多内核

如果使用了 [多内核](../2023-4/#多内核可选), 可以取消显示子菜单, 方便选择。`sudo vim /etc/default/grub`, 设置以下选项: 

```
GRUB_DISABLE_SUBMENU=y
```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [GRUB](https://wiki.archlinux.org/title/GRUB)
2. [GRUB/Tips_and_tricks](https://wiki.archlinux.org/title/GRUB/Tips_and_tricks)
{{< /admonition >}}