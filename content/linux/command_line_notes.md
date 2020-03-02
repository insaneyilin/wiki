---
title: "Linux/Unix 命令行 cheatsheet"
date: 2019-11-17 01:01
---

## 终端快捷键

- `Ctrl + A`    移动光标至行首
- `Ctrl + E`    移动光标至行尾
- `Ctrl + L`    清屏
- `Ctrl + U`    剪切光标前的所有字符
- `Ctrl + K`    剪切光标后的所有字符
- `Ctrl + W`    剪切光标前的内容，直到遇到空格为止
- `Ctrl + Y`    粘贴上一次剪切的字符
- `Ctrl + H`    与退格键相同
- `Ctrl + C`    终止当前执行的进程
- `Ctrl + D`    当没有进程在执行时退出当前终端，如果当前有进程就发送 EOF 命令给当前进程
- `Ctrl + Z`    将执行中的任何东西放入后台进程。fg 可以将其恢复。
- `Ctrl + _`    撤销最后一条命令（因为是下划线，所以实际上是 `Ctrl + Shift + _` ）
- `Ctrl + F`    将将光标向前移动一个字符
- `Ctrl + B`    将将光标向后移动一个字符
- `Option + →`  光标向前移动一个单词 (OS X下，iTerm2 需要特殊配置)
- `Option + ←`  光标向后移动一个单词

## 常用命令

- `cd ~`: 回到 $HOME 目录
- `cd -`: 回到上一次的目录
- `ls`: 列出文件
- `ls -l`: 详细列表
- `ls -a`: 列出隐藏文件
- `ls -lh`: 更可读的方式列出
- `ls -R`: 递归显示文件夹内的内容
- `top`: 实时显示进程信息
- `ps`: 查看进程信息（当前 session）
- `ps aux`: 查看所有进程信息
- `history n`: 列出最近执行过的 n 条命令
- `Ctrl + R`: 交互式检索之前执行的命令
- `!<command>`： 执行最近以 `<command>` 开头的命令
- `!!<command>:p`：显示最近执行的以 `<command>` 开头的命令
- `!!`: 执行上一条命令
- `!!:p`：将上一条命令打印到终端
- `rm -i <file>`：删除文件时进行确认
- `rm -f <file>`：强制删除
- `rm -r <dir>`：删除目录
- `find <dir> -name <pattern>`：搜索文件
- `grep <pattern> <file>`：在文件中搜索
- `grep -r <pattern> <dir>`：递归搜索目录中含有关键字的所有行
- `grep -v <pattern> <file>`：搜索文件中不含有关键字的所有行
- `grep -i <pattern> <file>`：搜索文件中含有关键字的所有行

## 命令链/管道

- `[command-a]; [command-b]`      不管命令 a 是否执行成功，执行完命令 a 后再执行命令 b
- `[command-a] && [command-b]`    如果命令 a 执行成功就执行命令 b
- `[command-a] || [command-b]`    如果命令 a 执行失败就执行命令 b
- `[command-a] &`                 在后台执行命令 a
- `[command-a] | [command-b]`     运行命令 a，然后将结果给命令 b，例如 `ps aux | grep python`
- `[command] > [file]`            将命令输出的内容覆盖到文件里
- `[command] >> [file]`           将命令输出的内容附加到文件里
- `[command] < [file]`            告诉命令从文件中读取内容
