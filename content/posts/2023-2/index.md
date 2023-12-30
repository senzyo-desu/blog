---
title: "Rime配置"
date: 2023-04-03T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom]
featuredImagePreview: ""
summary: Rime 的安装、输入方案、词库与皮肤。
---

## 安装

### Arch

安装 fcitx5 包组和 rime: 

```bash
yay -S fcitx5-im fcitx5-rime
```

`sudo vim /etc/environment` 并添加以下几行, 然后重新登录:

```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

### Windows

由于 [原版](https://github.com/rime/weasel) 停更已久, 推荐使用其他作者 [二次开发版本](https://github.com/fxliang/weasel), 支持更多特性。

但是注意, 这个二次开发版本, 要从该仓库的 [GitHub Action](https://github.com/fxliang/weasel/actions) 构建中下载, 因为作者说不发 [Release](https://github.com/fxliang/weasel/releases) 了。

## 配置

刚开始用 Rime 的时候, 跟着文档做自定义配置, 了解了各个配置文件的作用, 后来发现 Rime 实在是太复杂, 与其自定义, 不如直接用大佬的方案: [雾凇拼音](https://github.com/iDvel/rime-ice)。

### 雾凇拼音方案

#### Arch

```bash
yay -S rime-ice-git
```

自定义 [default.yaml](https://ghproxy.net/https://raw.githubusercontent.com/iDvel/rime-ice/main/default.yaml) 和 [custom_phrase.txt](https://ghproxy.net/https://raw.githubusercontent.com/iDvel/rime-ice/main/custom_phrase.txt) 文件中的内容, 存放到 rime 的个人配置文件夹 `~/.local/share/fcitx5/rime/` 中。

右击托盘中的 Rime 图标, 重新启动以使配置生效。

#### Windows

将 [雾凇拼音](https://github.com/iDvel/rime-ice) 的所有文件复制到个人配置文件夹 `C:/Users/<UserName>/AppData/Roaming/Rime/` 中, 自定义 [default.yaml](https://ghproxy.net/https://raw.githubusercontent.com/iDvel/rime-ice/main/default.yaml) 和 [custom_phrase.txt](https://ghproxy.net/https://raw.githubusercontent.com/iDvel/rime-ice/main/custom_phrase.txt) 文件中的内容。

右击托盘中的 Rime 图标, 重新部署以使配置生效。

### 自定义方案

{{< admonition type=note title="提示" open=true >}}
以 Arch 为例, 按照文档来, 简单的自定义方案就是这样了。

仅作参考, 推荐直接使用上面的 [雾凇拼音](https://github.com/iDvel/rime-ice) 方案。
{{< /admonition >}}

创建个人配置文件夹: 

```bash
mkdir ~/.local/share/fcitx5/rime/
```

在该文件夹中创建 `default.custom.yaml` 文件, 以指定可选的输入法。例如添加下列内容以使用朙月拼音 (luna_pinyin) : 

```yaml
patch:
  "menu/page_size": 9  # 9个候选词
  schema_list:
    - schema: luna_pinyin
```

添加内容时注意行首缩进。此文件会覆盖默认的配置文件, 若只在文件中添加朙月拼音, 则只能用朙月拼音。

右击托盘中的 Rime 图标, 重新启动以使配置生效。

使用 Rime 时, 默认可以按 `F4` 或 <code>Ctrl+\`</code> 切换输入法。`default.yaml` 文件中的 `grave` 指的就是 `Tab` 键上方的 **`** 键。

#### 导入词库

安装 [中文维基百科词库](https://archlinux.org/packages/community/any/rime-pinyin-zhwiki/) 和 [萌娘百科词库](https://aur.archlinux.org/packages/fcitx5-pinyin-moegirl-rime) 为例: 

```bash
yay -S rime-pinyin-zhwiki fcitx5-pinyin-moegirl-rime
```

{{< admonition type=tip title="提示" open=true >}}
可以将自定义词库文件放入 `~/.local/share/fcitx5/rime/`, 文件名(filename.dict.yaml)应与词库名统一。
{{< /admonition >}}

编辑用于朙月拼音的自定义配置文件: 

```bash
vim ~/.local/share/fcitx5/rime/luna_pinyin.custom.yaml
```

指定词库配置文件: 

```yaml
# encoding: utf-8

patch:
  "translator/dictionary": luna_pinyin.extended  # 词库名字可自定义,与文件名保持一致即可
```

编辑用于朙月拼音的词库配置文件: 

```bash
vim ~/.local/share/fcitx5/rime/luna_pinyin.extended.dict.yaml
```

指定具体的词库以及其他配置: 

```yaml
# encoding: utf-8

---
name: luna_pinyin.extended
version: "2023.04.03"
sort: by_weight  # 字典初始排序, 可選original或by_weight
use_preset_vocabulary: false  # 是否启用默认的“八股文”词库及词频系统

import_tables:
  - zhwiki
  - moegirl
```

#### Emoji

从 [rime-emoji](https://github.com/rime/rime-emoji) 下载 opencc 文件夹, 放入用户配置文件夹 `~/.local/share/fcitx5/rime/` 中。

将 `emoji_suggestion.yaml` 中的内容写入用于朙月拼音的自定义配置文件中: 

```bash
vim ~/.local/share/fcitx5/rime/luna_pinyin.custom.yaml
```

```yaml
# encoding: utf-8

patch:
  "translator/dictionary": luna_pinyin.extended
  switches/@next:
    name: emoji_suggestion
    reset: 1
    states: [ "🈚︎", "🈶️" ]
  'engine/filters/@before 0':
    simplifier@emoji_suggestion
  emoji_suggestion:
    opencc_config: emoji.json
    option_name: emoji_suggestion
    tips: none
    inherit_comment: false
```

#### 自定义短语

编辑 `~/.local/share/fcitx5/rime/custom_phrase.txt`, 

{{< admonition type=warning title="注意" open=true >}}
使用 `UTF-8 ` 编码。

各字段以制表符 `Tab` 分隔, 注意不要被编辑器自动替换为了 `空格`。

字段顺序为: 文字、编码、权重 (可选)。
{{< /admonition >}}

例如: 

```
中州韻輸入法引擎	rime	2
http://rime.im/	rime	1
Rime	rime	3
```

{{< admonition type=tip title="提示" open=true >}}
虽然文本码表编辑较为方便, 但不适合导入大量条目。可以自定义词库文件来实现这一目的。
{{< /admonition >}}

## 皮肤

### Linux

因为 Linux 中, Rime 的前端是通过 Fcitx5 或者 IBus 来实现的, 没法像 Windows 和 macOS 那样使用 YAML 文件来自定义皮肤。

推荐直接用自带的 `KDE Plasma (实验性) `, 这是一款亮色主题, 圆角和高亮还比较耐看。或者安装主题包, 参考 [Fcitx5#主题](https://wiki.archlinuxcn.org/wiki/Fcitx5#主题)。

不嫌麻烦的话, 可以使用桌面环境的配置界面来自定义皮肤, 比如在 KDE 的“设置->语言和区域设置->输入法->配置附加组件->经典用户界面->主题”中选择主题, 或者点击右边的齿轮进行自定义。

### Windows

我模仿 Win11 的输入法样式自定义了亮色和暗色皮肤, 使用该配置要注意自己的字体文件, 最好安装适用于 Windows 的 [Noto Color Emoji](https://fonts.google.com/noto/specimen/Noto+Color+Emoji), 不然候选项中不显示国旗。虽然 Windows 中不支持显示国旗, 但在输入法中显示多少算个安慰。

将以下文件保存为 `styles.yaml`, 放入个人配置文件夹 `C:/Users/<UserName>/AppData/Roaming/Rime/` 中。

```yaml
patch:
  __include: my_color_schemes
  __include: font_face_settings
  __include: font_size_settings
  __include: win11                  # 使用本文件最后面自定义的 Win 11 方案
  style/use_argb_value: true        # 支持 argb
  # __include: _vertical_text       # 取消注释即启用竖排文本
  style/mark_text: "|"              # 在候选项左侧显示标记, 模仿 Win 11 样式
  #style/capture_by_click: true     # 捕获功能, 通过单击“候选窗口/高亮的候选”
  style/layout/max_width: 800       # 水平布局的最大窗口宽度, 设置 0 可禁用最大宽度; 如果长度过长, 则为候选者设置多行
  style/layout/max_height: 400      # 带换行的竖排文本布局的最大窗口高度, 设置 0 可禁用最大高度

my_color_schemes:
  preset_color_schemes/win11light:
    name: "Win11 亮色 / Win11 Light"
    # color format, argb: 0xaarrggbb; rgba: 0xrrggbbaa。也可以使用 argb
    color_format: rgba
    border_color: 0xF9F9F9FF                      # 整体外边框颜色
    back_color: 0xF9F9F9FF                        # 整体底色
    hilited_candidate_back_color: 0xF0F0F0FF      # 选中项整体底色
    # hilited_candidate_border_color: 0xF0F0F0FF  # 选中项整体外边框颜色
    # candidate_back_color: 0xF9F9F9FF            # 候选项整体底色

    # hilited_back_color: 0xF0F0F0FF              # 选中拼音底色 (inline_preedit: false)
    hilited_text_color: 0x0067C0FF                # 选中拼音颜色 (inline_preedit: false)
    hilited_label_color: 0x505050FF               # 选中序号颜色
    hilited_candidate_text_color: 0x505050FF      # 选中文字颜色
    hilited_comment_text_color: 0x505050FF        # 选中注音颜色
    hilited_mark_color: 0x0067C0FF                # 标记颜色

    text_color: 0x000000FF               # 拼音颜色 (inline_preedit: false)
    label_color: 0x000000FF              # 序号颜色
    candidate_text_color: 0x000000FF     # 文字颜色
    comment_text_color: 0x000000FF       # 注音颜色
    shadow_color: 0x2C2C2CFF             # 阴影颜色
 
  preset_color_schemes/win11dark:
    name: "Win11 暗色 / Win11 Dark"
    # color format, argb: 0xaarrggbb; rgba: 0xrrggbbaa
    color_format: rgba        # 也可以使用 argb
    border_color: 0x2C2C2CFF                      # 整体外边框颜色
    back_color: 0x2C2C2CFF                        # 整体底色
    hilited_candidate_back_color: 0x383838FF      # 选中项整体底色
    # hilited_candidate_border_color: 0x383838FF  # 选中项整体外边框颜色
    # candidate_back_color: 0x2C2C2CFF            # 候选项整体底色

    # hilited_back_color: 0x383838FF              # 选中拼音底色 (inline_preedit: false)
    hilited_text_color: 0x4CC2FFFF                # 选中拼音颜色 (inline_preedit: false)
    hilited_label_color: 0xDEDEDEFF               # 选中序号颜色
    hilited_candidate_text_color: 0xDEDEDEFF      # 选中文字颜色
    hilited_comment_text_color: 0xDEDEDEFF        # 选中注音颜色
    hilited_mark_color: 0x4CC2FFFF                # 标记颜色

    text_color: 0xFFFFFFFF               # 拼音颜色 (inline_preedit: false)
    label_color: 0xFFFFFFFF              # 序号颜色
    candidate_text_color: 0xFFFFFFFF     # 文字颜色
    comment_text_color: 0xFFFFFFFF       # 注音颜色
    shadow_color: 0x2C2C2CFF             # 阴影颜色

font_size_settings:
  style/font_point: 12
  style/label_font_point: 12
  style/comment_font_point: 12

# 所有字体都可以通过设置范围来设置; 
# 在逗号和冒号周围加上空格都可以; 
# 每个字体集合单元的第一个必须是字体名称; 
# 起始码点必须在最后码点之前, 使用十六进制; 
# 如果要定义起始和结束点, 可以这样做: fontname: start_code_point: end_code_point; 
# 如果只定义起始码点, 可以这样做: fontname: start_code_point; 
# 如果只定义结束码点, 可以这样做: fontname:: end_code_point; 
# 字体的粗细和样式必须在第一个字体集合单元中设置, 使用[: weight_set] [:style_set]; 
# 字体的粗细可以是以下内容, 大小写不敏感: thin、extra_light、ultra_light、light、semi_light、medium、demi_bold、semi_bold、bold、extra_bold、ultra_bold、black、heavy、extra_black、ultra_black、normal

# 字体的样式可以是以下内容, 大小写不敏感: italic、oblique、normal

# 在一些符号字体之后设置主字体, 以确保符号在正确的字体中显示。如果要指定符号字体, 则需要在 font_face_settings 中添加字体设置
font_face_settings:
  # 普通文字字体
  style/font_face: "Segoe UI Emoji:20:39, Segoe UI Emoji:1f51f:1f51f, Noto Color Emoji:80, Segoe UI Emoji:80, Sarasa UI SC, Microsoft Yahei"
  style/label_font_face: "Sarasa UI SC, Microsoft Yahei"             # 序号字体
  style/comment_font_face: "Sarasa UI SC, Microsoft Yahei"           # 注音字体

_vertical_text:
  style/vertical_text: true                 # 设置 true 以使用竖排文本布局, 古代书籍的排版
  style/vertical_text_with_wrap: false      # 竖排文本布局, 具有换行整体候选功能, 生效前提: style/vertical_text: true 且 style/layout/max_height: not_zero_value
  style/vertical_text_left_to_right: false  # 竖排文本布局的流向设置
  style/layout/max_height: 0

_vertical_text_wrap:
  __include: _vertical_text
  style/vertical_text_with_wrap: true
  style/layout/max_height: 600

win11:
  # style/blur_window: true     # 仅支持 Win 10 or Win 11 (build number <= 22000), 其余未测试; 另外要求外阴影透明, 关键背景色 alpha 不大于 7f
  style/horizontal: false       # 行文向: 横 horizontal | 纵 vertical
  style/inline_preedit: false   # 拼音位于: 候选框 false | 输入框内 true
  # color_scheme settings
  style/color_scheme: win11light
  style/color_scheme_dark: win11dark
  #style/preedit_type: preview_all    # composition
  # layout settings
  style/layout/align_type: center     # 选项: top, center, bottom 
  style/layout/min_height: 0
  style/layout/border: 0              # 边框粗细
  style/layout/spacing: 20            # 拼音与候选项的间距(inline_preedit: false)
  style/layout/candidate_spacing: 15  # 候选项间距
  style/layout/shadow_radius: 3       # 阴影半径。设置 0 可以禁用阴影功能
  # 如果 shadow_offset_x 和 shadow_ooffset_y 都为 0, 则为圆形阴影。负片向左投阴影。
  style/layout/shadow_offset_x: 2     # 阴影边缘横向轮廓
  style/layout/shadow_offset_y: 2     # 阴影边缘竖向轮廓
  style/layout/hilite_padding: 5      # 选中项的填充(高亮面积)
  style/layout/hilite_spacing: 5      # 序号与候选词间距
  style/layout/margin_x: 13           # 整体的横向外边距
  style/layout/margin_y: 13           # 整体的竖向外边距
  style/layout/round_corner: 10       # 选中项高亮的圆角半径
  style/layout/corner_radius: 10      # 整体的圆角半径
```

右击托盘中的 Rime, 点击重新部署; 再点击“输入法设定”, 先选择输入方案, 再选择相应的皮肤。

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. https://wiki.archlinuxcn.org/wiki/Fcitx5
2. https://wiki.archlinuxcn.org/wiki/Rime
3. https://rime.im/docs/
{{< /admonition >}}
