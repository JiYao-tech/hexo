---
title: IDEA中的Git操作
tags: Git
copyright: true
abbrlink: 20484
date: 2020-03-24 17:35:07
---
> 大家在使用Git时，都会选择一种Git客户端，在IDEA中内置了这种客户端，可以让你不需要使用Git命令就可以方便地进行操作，本文将讲述IDEA中的一些常用Git操作。

## 环境准备

- 使用前需要安装一个远程的Git仓库和本地的Git客户端，具体参考：[10分钟搭建自己的Git仓库](https://mp.weixin.qq.com/s?__biz=MzU1Nzg4NjgyMw==&mid=2247483948&idx=1&sn=6562c995519a3b6cf0105e66620ed562&scene=21#wechat_redirect)。
- 由于IDEA中的Git插件需要依赖本地Git客户端，所以需要进行如下配置：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190434-233250.jpeg)

## 操作流程

> 我们这里使用mall-tiny项目的源代码来演示，尽可能还原一个正式的操作流程。

### 在Gitlab中创建一个项目并添加README文件

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190445-855425.jpeg)

<!--more-->

### clone项目到本地

- 打开从Git检出项目的界面：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190447-410655.jpeg)

- 输入Git地址进行检出：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190455-269364.png)

- 暂时不生成IDEA项目，因为项目还没初始化：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190457-393131.jpeg)

### 初始化项目并提交代码

- 将mall-tiny的代码复制到该目录中：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190459-45475.jpeg)

- 这里我们需要一个.gitignore文件来防止一些IDEA自动生成的代码被提交到Git仓库去：

```
# Maven #
target/

# IDEA #
.idea/
*.iml

# Eclipse #
.settings/
.classpath
.project
```

- 使用IDEA打开项目：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190502-322099.jpeg)

- 右键项目打开菜单，将所有文件添加到暂存区中：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190508-130940.jpeg)

- 添加注释并提交代码：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190511-637220.jpeg)

### 将代码推送到远程仓库

- 点击push按钮推送代码：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190516-689389.jpeg)

- 确认推送内容：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190517-542755.jpeg)

- 查看远程仓库发现已经提交完成：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190520-952697.jpeg)

### 从远程仓库拉取代码

- 在远程仓库添加一个README-TEST.md文件：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190523-130430.jpeg)

- 从远程仓库拉取代码：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190526-604464.jpeg)

- 确认拉取分支信息：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190531-590298.jpeg)

### 从本地创建分支并推送到远程

- 在本地创建dev分支，点击右下角的Git:master按钮：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190533-313696.jpeg)

- 使用push将本地dev分支推送到远程：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190535-800010.jpeg)

- 确认推送内容：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190537-150687.jpeg)

- 查看远程仓库发现已经创建了dev分支：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190540-894712.jpeg)

### 分支切换

- 从dev分支切换回master分支：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190542-779232.png)

### Git文件冲突问题解决

- 修改远程仓库代码：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190814-937523.jpeg)

- 修改本地仓库代码：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190544-499632.png)

- 提交本地仓库代码并拉取，发现代码产生冲突，点击Merge进行合并：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190545-988790.jpeg)

- 点击箭头将左右两侧代码合并到中间区域：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190547-10095.jpeg)

- 冲突合并完成后，点击Apply生效：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190549-813062.png)

- 提交代码并推送到远程。

### 从dev分支合并代码到master

- 在远程仓库修改dev分支代码：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190551-123887.jpeg)

- 在本地仓库拉取代码，选择从dev分支拉取并进行合并：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190552-573751.jpeg)

- 发现产生冲突，解决后提交并推送到远程仓库即可。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190555-514828.jpeg)

### 查看Git仓库提交历史记录

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/undefined/202004/30/190556-925372.jpeg)



> 转载自：https://mp.weixin.qq.com/s/zmw7FfF7vkgWD2rx-KfijA

