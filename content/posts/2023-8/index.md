---
title: "两种歌词格式的相互转换"
date: 2023-12-13T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Python]
featuredImagePreview: ""
summary: 使用 ` / ` 分隔 `歌词原词` 和 `歌词翻译` 的格式 A, `歌词原词` 和 `歌词翻译` 使用相同 `时间标签` 的格式 B。
---

## 格式介绍

### 格式A

```
[12:34.56]选ぶ色も服も违って / 喜好的衣服与配色都不一样
```

格式 A 可以分为这样几部分: 

1. `时间标签`: 以 `[` + `数字` 开头, 以 `]` 结尾。比如 `[12:34.56]`。
2. `分隔符`: ` / ` 是分隔符。
3. `歌词原词`: `时间标签` 和 `分隔符` 之间的部分。比如 `选ぶ色も服も违って`。
4. `歌词翻译`: `分隔符` 右侧的部分。比如 `喜好的衣服与配色都不一样`。

所以, 格式 A 的组成可以看作: 

```
时间标签+歌词原词+分隔符+歌词翻译
```

### 格式B

```
[12:34.56]选ぶ色も服も违って
[12:34.56]喜好的衣服与配色都不一样
```

格式 B 可以分为这样几部分: 

1. `时间标签`: 以`[`+`数字`开头, 以`]`结尾。比如`[12:34.56]`。
2. `歌词原词`: 对于上下相邻的两行, 如果它们的 `时间标签` 相同, 则第 1 行的 `时间标签` 右侧的部分是 `歌词原词`。比如 `选ぶ色も服も违って`。
3. `歌词翻译`: 对于上下相邻的两行, 如果它们的时间标签相同, 则第 2 行的 `时间标签` 右侧的部分是 `歌词翻译`。比如 `喜好的衣服与配色都不一样`。

所以, 格式 B 的组成可以看作: 

```
时间标签+歌词原词
时间标签+歌词翻译
```

## 脚本

### 格式A转为格式B

让 ChatGPT 写一个 Python 脚本 `A2B.py`, 把 `./Input` 文件夹中, 所有使用格式 A 的歌词文件转换为使用格式 B 的新歌词文件: 

{{< admonition type=quote title="要求" open=true >}}
逐行读取一个歌词文件的内容, 检查每行中是否有 `分隔符`。

如果该行中有 `分隔符`, 则把该行的 `时间标签` 和 `歌词原词` 作为一行内容输出到列表, 然后把该行的 `时间标签` 和 `歌词翻译` 作为一行内容输出到列表的下一行。

如果该行中没有 `分隔符`, 则把该行保持原样直接输出到列表。

如果文件夹 `./Output/Format-B` 不存在, 就创建它们。

在文件夹 `./Output/Format-B` 中, 创建一个与当前歌词文件同名的新歌词文件, 使用 UTF-8 编码把列表中的数据写入新歌词文件中。

检查新歌词文件的最后一行是否为空白行。

如果是空白行, 就删掉这个空白行。
{{< /admonition >}}

脚本 `A2B.py`: 

```python
import os

def convert_format_A_to_B(input_path, output_path):
    with open(input_path, 'r', encoding='utf-8') as input_file:
        lines = input_file.readlines()

    output_lines = []
    for line in lines:
        if ' / ' in line:
            timestamp, rest = line.split(']')[0] + ']', line.split(' / ')[0].split(']')[1]
            output_lines.append(f"{timestamp}{rest}")
            output_lines.append(f"{timestamp}{line.split(' / ')[1].strip()}")
        else:
            output_lines.append(line.strip())

    # Remove the last empty line if it exists
    if output_lines and not output_lines[-1]:
        output_lines.pop()

    # Create output directory if it doesn't exist
    output_directory = os.path.join(output_path, 'Format-B')
    os.makedirs(output_directory, exist_ok=True)

    # Write the output to a new file in Format-B directory
    output_file_path = os.path.join(output_directory, os.path.basename(input_path))
    with open(output_file_path, 'w', encoding='utf-8') as output_file:
        output_file.write('\n'.join(output_lines))

if __name__ == "__main__":
    input_directory = "./Input"
    output_directory = "./Output"

    # Iterate through all .lrc files in the input directory
    for file_name in os.listdir(input_directory):
        if file_name.endswith(".lrc"):
            input_file_path = os.path.join(input_directory, file_name)
            convert_format_A_to_B(input_file_path, output_directory)

print("Conversion completed.")
```

### 格式B转为格式A

让 ChatGPT 写一个 Python 脚本 `B2A.py`, 把 `./Input` 文件夹中, 所有使用格式 B 的歌词文件转换为使用格式 A 的新歌词文件: 

{{< admonition type=quote title="要求" open=true >}}
逐行读取一个歌词文件的内容, 检查每行中是否有 `时间标签`。

如果有 `时间标签`, 则检查该行的 `时间标签` 和下一行的 `时间标签` 是否相同。

如果该行的 `时间标签` 和下一行的 `时间标签` 相同, 则拼接该行内容 (`时间标签` + `歌词原词`) + `分隔符` + 下一行的 `歌词翻译`, 作为一行内容输出到列表。

如果该行的 `时间标签` 和下一行的 `时间标签` 不相同, 则把该行保持原样直接输出到列表。

如果该行中没有 `时间标签`, 则把该行保持原样直接输出到列表。

如果文件夹 `./Output/Format-A` 不存在, 就创建它们。

在文件夹 `./Output/Format-A` 中, 创建一个与当前歌词文件同名的新歌词文件, 使用 UTF-8 编码把列表中的数据写入新歌词文件中。

检查新歌词文件的最后一行是否为空白行。

如果是空白行, 就删掉这个空白行。
{{< /admonition >}}

脚本 `B2A.py`: 

```python
import os

def convert_format_B_to_A(input_path, output_path):
    with open(input_path, 'r', encoding='utf-8') as input_file:
        lines = input_file.readlines()

    output_lines = []
    i = 0
    while i < len(lines):
        line = lines[i].strip()
        if '[' in line:
            timestamp = line.split(']')[0] + ']'
            if i + 1 < len(lines) and '[' in lines[i + 1]:
                next_line_timestamp = lines[i + 1].split(']')[0] + ']'
                if timestamp == next_line_timestamp:
                    lyrics_original = line.split(']')[1].strip()
                    lyrics_translation = lines[i + 1].split(']')[1].strip()
                    output_lines.append(f"{timestamp}{lyrics_original} / {lyrics_translation}")
                    i += 1  # Skip the next line as it has been processed
                else:
                    output_lines.append(line)
            else:
                output_lines.append(line)
        else:
            output_lines.append(line)
        i += 1

    # Remove the last empty line if it exists
    if output_lines and not output_lines[-1]:
        output_lines.pop()

    # Create output directory if it doesn't exist
    output_directory = os.path.join(output_path, 'Format-A')
    os.makedirs(output_directory, exist_ok=True)

    # Write the output to a new file in Format-A directory
    output_file_path = os.path.join(output_directory, os.path.basename(input_path))
    with open(output_file_path, 'w', encoding='utf-8') as output_file:
        output_file.write('\n'.join(output_lines))

if __name__ == "__main__":
    input_directory = "./Input"
    output_directory = "./Output"

    # Iterate through all .lrc files in the input directory
    for file_name in os.listdir(input_directory):
        if file_name.endswith(".lrc"):
            input_file_path = os.path.join(input_directory, file_name)
            convert_format_B_to_A(input_file_path, output_directory)

print("Conversion completed.")
```

## 测试用例

使用格式 A 的 `時の魔法.lrc`: 

```
[ti:時の魔法]
[ar:小木曽雪菜]
[al:WHITE ALBUM2 Original Soundtrack ～setsuna～]
[00:21.00]人は苦しみも微笑みも / 人的悲伤和幸福
[00:28.00]つないでゆける / 总是彼此联结着的
[00:34.10]あなたといるときの私なら / 与你在一起的日子 就算是我
[00:39.40]そんなことも思える / 也会时常这么想
[00:45.50]こんな 一枚の葉もつけない / 就算所有的绿叶都已凋零
[00:52.60]枯れた木にも キセキはおきる / 腐朽的枯木也会诞生出奇迹
[00:59.00]ほら 白いたくさんの光が / 看吧 那无数的白色光芒
[01:06.00]真冬のサクラ咲かせてゆくよ / 让冬日的樱花也能尽情绽放
[01:16.00]きっと叶うよ / 愿望总是会实现的
[01:22.30]願いは 目を閉じて / 只要闭上双眼
[01:29.30]時の魔法を唱えよう / 咏唱着时间的魔法
[01:38.80]ゼロから once again / 从零开始 重新来过吧
[01:46.00]
[01:56.30]忙しく日々を過ごしてた / 虽然忙忙碌碌地度过每一天
[02:03.40]だけど今は / 然而现在
[02:09.40]優しいまなざしで この世界 / 我终于能用温柔的眼光
[02:14.60]見渡せると思える / 眺望这个世界
[02:20.90]こんな 一枚の葉もつけない / 就算所有的绿叶都已凋零
[02:27.60]枯れた木にも キセキはおきて / 腐朽的枯木也会诞生出奇迹
[02:34.00]ほら 私の中にも輝く / 看吧 我心中闪耀的希望
[02:41.00]真冬のサクラ咲かせてゆくよ / 让冬日的樱花也能尽情绽放
[02:51.30]きっと叶うよ 願いは / 愿望总是会实现的
[03:00.80]届くから / 只要把心愿传达给你
[03:04.60]時の魔法を唱えたら / 咏唱着时间的魔法
[03:13.90]ここから once again / 从这里开始 重新来过吧
[03:20.30]
[03:31.70]あなたといる未来が / 与你在一起的未来
[03:38.10]キラキラと降ってくる / 闪烁着光芒从天空飘落
[03:44.70]いつまでもいつまでも / 无论何时 无论多久
[03:50.20]大切にしたいな / 都想去珍惜它呢
[04:00.60]きっと叶うよ 願いは / 愿望总是会实现的
[04:10.10]目を閉じて / 只要闭上双眼
[04:13.60]時の魔法を唱えよう / 咏唱着时间的魔法
[04:22.90]ゼロから once again / 从零开始 重新来过吧
[04:29.20]hm～ / hm～
[04:36.10]ゼロから once again / 从零开始 重新来过吧
[04:43.60]
```

使用格式 B 的 `POWDER SNOW.lrc`: 

```
[ti:POWDER SNOW]
[ar:冬馬かずさ]
[al:WHITE ALBUM2 Original Soundtrack ~kazusa~]
[00:06.80]粉雪が空から 優しく降りてくる
[00:06.80]点点细雪从天空彼端 温柔地飘落
[00:18.40]手のひらで受け止めた 雪が切ない
[00:18.40]用手心接住的雪 让人无法释怀
[00:29.30]どこかで見てますか あなたは立ち止まり
[00:29.30]好似看着哪里 你停下了脚步
[00:40.90]思い出していますか 空を見上げながら
[00:40.90]你想起来了吗 看着这片天空的同时
[00:53.90]嬉しそうに雪の上を歩くあなたが
[00:53.90]欢快着踩在雪面上的你
[01:05.00]私には本当に いとおしく見えた
[01:05.00]在我的眼中 是如此令人动情
[01:19.20]今でも覚えている あの日見た雪の白さ
[01:19.20]我至今还记忆犹新 那一天目睹的洁白雪花
[01:30.50]初めて触れた唇の温もりも忘れない
[01:30.50]还有初次碰触到的嘴唇的温暖
[01:40.45]I still love you
[01:40.45]我依然爱着你
[01:47.40]
[02:10.50]粉雪が私に いくつも降りかかる
[02:10.50]点点细雪在我的身上 纷纷地洒落
[02:21.80]暖かいあなたの 優しさに似ている
[02:21.80]就像温暖的你 带来的温柔一般
[02:33.00]楽しそうに話をしてくれたあなたが
[02:33.00]高兴地与我说着话的你
[02:43.90]私には心から恋しく思えた
[02:43.90]在我的心中 是如此令人心动
[02:57.70]今でも夢を見るの あの日見た白い世界
[02:57.70]我至今还如梦初醒 那一天目睹的纯白世界
[03:08.80]あの時触れた指先の冷たさも忘れない
[03:08.80]还有那时碰触到的指尖的冰凉
[03:18.35]I still love you
[03:18.35]我依然爱着你
[03:20.00]今でも覚えている あの日見た雪の白さ
[03:20.00]我至今还记忆犹新 那一天目睹的洁白雪花
[03:30.70]初めて触れた唇の温もりも忘れない
[03:30.70]还有初次碰触到的嘴唇的温暖
[03:40.50]粉雪のようなあなたは 汚れなく奇麗で
[03:40.50]如点点细雪般的你 如此纯白无垢
[03:52.60]私もなりたいと 雪に願う
[03:52.60]我也想变得像你一样 向着雪我如此祈愿道
[04:02.40]I still love you
[04:02.40]我依然爱着你
[04:07.70]
```
