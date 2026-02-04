---
title: "使用 GoReleaser 实现 Go 的自动化构建"
date: 2024-02-12T23:20:50+08:00
draft: false

# 文章短链接
slug: goreleaser-use
# 分类
categories: ["分享"]
# 标签
tags:
  - golang
  - 自动化构建
  - goreleaser
# 文章图片
featuredImage: https://img.ygxb.net/i/2024/02/06/65c24b4be3105.webp
---

## 前言

去年（准确来说应该是前年）我自己开发并开源了一个小项目 [lsky-upload](https://github.com/ygxbnet/lsky-upload)，主要功能是在 Typora 中把图片上传到自己搭建的 Lsky Pro 图床中，这样写文章时就不用考虑图片问题了（用 md 写文章的应该都懂）

开发并不难，我一边查资料一边写，很快就弄出来了，但程序构建打包就成了一个问题。

刚开始我是在自己的电脑上打包成可执行文件，再压缩成 zip 文件，然后上传到 GitHub Releases，这样持续了大概三个版本，并且因为我又很懒，所以就只打包了 Windows 的可执行文件，太懒了（笑

后来我在折腾 QQ 机器人时 看到 [go-cqhttp](https://github.com/Mrs4s/go-cqhttp) 项目使用 [GoReleaser](https://github.com/goreleaser/goreleaser) 作为自动化打包工具，然后我就研究了一下，欸，发现是真的好用

---

## 准备

- 本地 Golang 环境
- GitHub 账户
- Scoop（可选）

注意：GoReleaser 是 Golang 的专用构建工具，其它编程语言暂不支持

## 安装

### 使用 Scoop 安装

对于 Windows 用户，推荐直接用 [Scoop](https://scoop.sh/) 包管理安装。

安装好 Scoop 后只需执行以下命令即可完成 GoReleaser 安装

```shell
scoop install goreleaser
```

### 使用二进制文件安装

到 GoReleaser 的 [Releases](https://github.com/goreleaser/goreleaser/releases) 页面下载 `goreleaser_Windows_x86_64.zip` 文件

将其中的 `goreleaser.exe` 文件放在一个合适目录（例如：`C:\ProgramFiles\goreleaser`）

然后如下图将此目录添加到系统的环境变量

![image-20240212150218865](https://img.ygxb.net/i/2024/02/12/65c9c27fbafcf.webp)

### 验证

在终端或 Windows PowerShell 中输入：

```shell
goreleaser --version
```

如果有输出证明安装成功

## 初始化 GoReleaser

在你的 Golang 项目所在的目录执行

```shell
goreleaser init
```

它会生成一个 `.goreleaser.yaml` 这是 GoReleaser 的配置文件

这时你就可以运行以下命令在本地测试 GoReleaser

```shell
goreleaser release --snapshot
```

它默认会把构建的文件输出到项目的 `dist` 目录

其中就包含由多平台可执行文件组成的多个压缩文件：

```txt
lsky-upload_Darwin_arm64.tar.gz
lsky-upload_Darwin_x86_64.tar.gz
lsky-upload_Linux_arm64.tar.gz
lsky-upload_Linux_i386.tar.gz
lsky-upload_Linux_x86_64.tar.gz
lsky-upload_Windows_arm64.zip
lsky-upload_Windows_i386.zip
lsky-upload_Windows_x86_64.zip
```

现在已经完成了 GoReleaser 的基本配置

其实已经可以直接将这些文件上传到自己项目的 Releases 页，然后发布软件了

但这一点也不优雅，不但要上传这些文件，还要自己写更新日志，太麻烦了

这时就要请出大名鼎鼎的 GitHub Actions 了

## GitHub Actions 集成

首先你要有 GitHub 账户，并且已经把项目托管到了 GitHub

然后在你项目的根目录创建 `.github` 文件夹

在 `.github` 中再创建 `workflows`

再在其中创建 `release.yml` 文件（这文字描述也太麻烦了，算了直接上图）

![image-20240212215514523](https://img.ygxb.net/i/2024/02/12/65ca23446abe7.webp)

在 `release.yml` 文件中写入

```yaml
name: Release

on:
  push:
    tags:
      - v*

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 // 官方要求

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version-file: "go.mod" // 使用 go.mod 文件中的 golang 版本

      - name: Run GoReleaser
        uses: goreleaser/goreleaser-action@v5
        with:
          distribution: goreleaser
          version: latest
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} // 使用 github-actions 发布
```

这样当你给代码仓库加上以 `v` 开头的标签（例如：v1.0）并推送到 GitHub 时

它就会自动触发构建，根据 commit 信息生成更新日志，并且发布。这就非常的优雅了

### 注意

在 `release.yml` 文件中 如果把 `GITHUB_TOKEN` 设置成 `${{ secrets.GITHUB_TOKEN }}`

那 Release 会由 github-actions 来发布，如下图

![image-20240212221230792](https://img.ygxb.net/i/2024/02/12/65ca2751a83ed.webp)

如果想要由自己发布，就把 `GITHUB_TOKEN` 设置为 `${{ secrets.GH_ACCESS_TOKEN }}`

然后在仓库的 Settings\Secrets and variables\Actions 设置中添加一个叫 GH_ACCESS_TOKEN 的 Repository secrets

在 Value 中填入自己的 Access Token（获取教程参考：[GitHub 如何创建 Access Token](https://zhuanlan.zhihu.com/p/393441709)）设置好之后发布者就是你自己了

## 参考资料

- [GoReleaser 官方网站](https://goreleaser.com/)
- [lsky-upload 项目](https://github.com/ygxbnet/lsky-upload)

## 其它

哈！终于把这篇教程写完了，前前后后加起来差不多写了 4 天

这也算是我这个小小博客的第一篇文章吧，可喜可贺，终于克服懒癌写出来了（笑

说实话，其实这篇文章还有挺多地方我不是很满意，比如没有解释 `.goreleaser.yaml` 的具体配置，很多东西没表达清，排版比较乱

这些就以后慢慢改吧，现在先去偷一下懒吧 嘿嘿（先溜了~~~

最后，如果大家关于这篇文章还有什么问题，那就下方的评论区评论吧，我看到后都会解答
