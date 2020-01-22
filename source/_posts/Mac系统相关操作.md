---
title: 终端命令
author: 靳宏财
top: true
cover: true
toc: true
summary: 常用终端操作，文件权限修改
categories: Mac
tags:
  - 系统操作
date: 2020-01-22 14:09:55
---

### 常用终端命令

**只列举常用vim命令，用来提升生产力**

| 命令                                                         | 作用                            |
| ------------------------------------------------------------ | ------------------------------- |
| say xxx                                                      | 读单词                          |
| pwd                                                          | 显示当前目录的路径名            |
| ls                                                           | 查看当前目录下的文件            |
| ls -al                                                       | 查看所有文件，包含隐藏文件      |
| clear                                                        | 清除屏幕或窗口内容              |
| cd /xx/yy                                                    | 跳转到目录/xx/yy                |
| cd ..                                                        | 返回上一级目录                  |
| cd /                                                         | 返回根目录                      |
| cd -                                                         | 返回到上一步操作目录            |
| cat xx                                                       | 查看xx文件的内容                |
| man xx                                                       | 查看命令的详细帮助，比如 mac ls |
| killall Finder                                               | 重启Finder                      |
| touch xxx                                                    | 创建xxx文件                     |
| mkdir xxx                                                    | 创建xxx文件夹                   |
| rm xxx                                                       | 删除文件                        |
| rm -rf xxxx                                                  | 删除文件夹                      |
| defaults write com.apple.finder AppleShowAllFiles TRUE  killall Finder | 查看隐藏文件                    |
| defaults write com.apple.finder AppleShowAllFiles FALSE  killall Finder | 隐藏文件                        |
| ↑ ↓                                                          | 读取上一条或者下一条的命令记录  |
| sudo vi /private/etc/hosts                                   | 编辑hosts文件                   |

---

### 文件修改权限

**通过hexo new page 'xxx' 创建的文档通过编辑器打开后，发现是锁定的不可编辑文件。原因是用户只有read权限，那么我们如何通过终端来修改文件的权限呢**

**通过google后，会得到一条修改文件权限的命令：chmod -R 777 file，执行后确实修改了文件权限，但是针对这条命令里的关键字/语法，多少会有一些疑问，-R和777都是什么意思，对于疑问我们应该去给自己找一个答案**

1. 各字段含义：sudo chmod -R（更改文件夹及其子文件夹） 
              7（所有者权限）6（组用户权限）4（其他用户权限）xxx（目标文件）
2. 首先了解以下权限对应关系（执行权限字母表示  权限含义 执行权限数值表示）：
     r 读取权 4； 
     w 写入权 2； 
     x 执行权 1；
     rwx（读、写、执行）
     rw-（读、写）
     .......
3. 7、6、4的由来
    若要rwx：4+2+1=7； 若要rw-：4+2=6； 若要r-x：4+1=5