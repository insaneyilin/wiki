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

---

## GitHub Standard Fork & Pull Request Workflow

主要思想：从主 repo fork 到自己账户下，设置 upstream，保持 remote master 和 local master 干净，新 feature 在分支开发，开发完同步主 repo （merge, rebase …），从开发分支直接发 pull request 到主 repo。

有一点要注意，**不要在 local master 分支上开发新 feature **，否则同步远端 upstream 分支的时候会乱掉。

### 1. Fork 主 repo 到自己账户

```
# Clone your fork to your local machine
git clone git@github.com:USERNAME/FORKED-PROJECT.git
```

### 2. 保持远端的 Fork 与上游同步

```
# Add 'upstream' repo to list of remotes
git remote add upstream https://github.com/UPSTREAM-USER/ORIGINAL-PROJECT.git

# Verify the new remote named 'upstream'
git remote -v

# Fetch from upstream remote
git fetch upstream

# View all branches, including those from upstream
git branch -va

# Checkout your master branch and merge upstream
git checkout master
git merge upstream/master
```

本地尽量不要用 master 开发，可以避免同步时出现冲突。

### 3. 创建分支开发新 feature

如果要进行开发（new feature or bugfix），建议创建一个新的分支开发，如 `dev_xxx` 或者 `bugfix_xxx` 。这样能够最大程度的控制自己的修改，同时保证修改同 master 分支的隔离。


### 4. 提交 Pull Request

先同步远端上游 master 到本地 master，再在开发分支上 rebase master，提 pr 的时候可以直接从分支提：

```
# Fetch upstream master and merge with your repo's master branch
git fetch upstream
git checkout master
git merge upstream/master

# If there were any new commits, rebase your development branch
git checkout newfeature
git rebase master
```

以上参考自 [Chaser324/GitHub-Forking.md](https://gist.github.com/Chaser324/ce0505fbed06b947d962)。

---

## git clone --depth=1 之后怎样获取其他分支？

```
git remote set-branches origin <remote_branch_name>
git fetch --depth 1 origin <remote_branch_name>
git checkout <remote_branch_name>
```

---

## 本地 repo 关联到远端 repo

```
git remote add origin git@github.com:your_name/your_project.git
git remote show origin

git branch --set-upstream-to=origin/<branch> <local_branch>
```

---

## GitHub 设置本地 repo 记住登录信息

github 针对本地 repo 设置保存用户名和密码，不用每次 pull/push 都重新输入：

```
git config --local credential.helper store
```


