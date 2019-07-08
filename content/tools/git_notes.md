---
title: "Git notes"
date: 2019-07-09 00:01
---

[TOC]

## <a name="gitconfig"></a>Git 配置

手动设置用户信息：

```
git config --global user.name "ABC"
git config --global user.email "abc@abc.com"
```

使用配置文件，`vim ~/.gitconfig`：

```
[user]
    name = ABC
    email = abc@abc.com

[core]
    editor = vim

[alias]
    st = status
    br = branch
    df = diff
    l = log
    lg = log --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s [%C(cyan)%an%Creset] %Cgreen(%cr)%Creset' --graph --decorate
    co = checkout
```

---

## Github 配置 SSH Key

### 1. 设置 git 的 username 和 e-mail

参考 [Git 配置](#gitconfig)。

### 2. 生成 ssh key

```
# 查看是否已存在 ssh key
ls ~/.ssh/id_rsa*

# 若没有，生成一个新的 key
ssh-keygen -t rsa -C "abc@abc.com"

# 查看生成的 ssh key
cat ~/.ssh/id_rsa.pub
```

### 3. Github 添加 ssh key

在 Github 用户设置里将 `~/.ssh/id_rsa.pub` 中的内容贴上去。

### 4. 验证

```
ssh -T git@github.com
```

