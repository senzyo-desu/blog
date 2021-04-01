---
title: "Linux的cp与mv"
date: 2021-03-18T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Bash,Linux]
featuredImagePreview: ""
summary: 关于使用 `cp` 与 `mv` 的一点细节备忘。
---

## cp

### 复制目录下所有文件

用于测试的目录树如下: 

```
.
├── folder-1
│   └── file
└── folder-2
```

```bash
cp folder-1/* folder-2
等效于
cp folder-1/* folder-2/
```

```
.
├── folder-1
│   └── file
└── folder-2
    └── file
```

### cp -r

`cp -r`递归复制。

#### 例1

用于测试的目录树如下: 

```
.
├── folder-1
│   └── file
└── folder-2
```

```bash
cp -r folder-1 folder-2
# 等效于
cp -r folder-1/ folder-2
# 等效于
cp -r folder-1 folder-2/
# 等效于
cp -r folder-1/ folder-2/
```

```
.
├── folder-1
│   └── file
└── folder-2
    └── folder-1
        └── file
```

#### 例2

用于测试的目录树如下: 

```
.
└── folder-1
    └── file
```

注意, `folder-2`是不存在的目录。

```bash
cp -r folder-1 folder-2
# 等效于
cp folder-1 folder-2
```

```
.
├── folder-1
│   └── file
└── folder-2
    └── file
```

#### 例3

用于测试的目录树如下: 

```
.
├── folder-1
│   └── file
└── folder-2
```

注意, `folder-3`是不存在的目录。

```bash
cp -r folder-1 folder-2/folder-3
```

```
.
├── folder-1
│   └── file
└── folder-2
    └── folder-3
        └── file
```

### cp --parents

#### 目标目录存在

用于测试的目录树如下: 

```
.
├── folder-1
└── folder-a
    └── folder-b
        └── file
```

```bash
cp --parents folder-a/folder-b/file folder-1
```

```
.
├── folder-1
│   └── folder-a
│       └── folder-b
│           └── file
└── folder-a
    └── folder-b
        └── file
```

{{< admonition type=note title="效果不同于cp -r" open=true >}}

```bash
cp -r folder-a/folder-b/file folder-1
```

```
.
├── folder-1
│   └── file
└── folder-a
    └── folder-b
        └── file
```

{{< /admonition >}}


#### 目标目录不存在

用于测试的目录树如下: 

```
.
├── folder-1
└── folder-a
    └── folder-b
        └── file
```

注意, `folder-2`是不存在的目录。

```bash
cp --parents folder-a/folder-b/file folder-1/folder-2
```

结果是, 并不会自动创建不存在的目标目录, 所以目标目录必须存在才能进行复制过程。

## mv

### 移动目录

用于测试的目录树如下: 

```
.
├── folder-1
│   └── file
└── folder-2
```

```bash
mv folder-1 folder-2
等效于
mv folder-1/ folder-2
等效于
mv folder-1 folder-2/
等效于
mv folder-1/ folder-2/
```

```
.
└── folder-2
    └── folder-1
        └── file
```

### 移动目录下所有文件

用于测试的目录树如下: 

```
.
├── folder-1
│   └── file
└── folder-2
```

```bash
mv folder-1/* folder-2
等效于
mv folder-1/* folder-2/
```

```
.
├── folder-1
└── folder-2
    └── file
```

