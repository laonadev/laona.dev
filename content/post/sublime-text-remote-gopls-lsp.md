---
title: "Sublime Text 集成远程 gopls 服务器"
date: 2019-07-31T13:07:33+08:00
lastmod: 2019-07-31T13:07:33+08:00
draft: false
keywords: ["Sublime Text", "LangServer", "LSP", "Golang", "gopls"]
description: "Sublime Text 安装 gopls 支持，连接到远程 gopls 服务器，解决 Go 开发使用 Samba 卡顿的问题。"
tags: ["Sublime Text", "Golang"]
categories: ["开发"]
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
postMetaInFooter: true
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

庙里的小和尚们在使用 Sublime Text 作 Go 开发时，一般都会安装 GoSublime 的扩展。

做开发的小和尚一般会人手配备一台 Linux 开发虚拟机，代码通过在 Linux 下搭建 Samba 服务映射为 Windows 的 Z: 盘。

由于庙里开发环境的这种特殊性，在使用 GoSublime 过程中总是会出现各种奇怪的问题：

* 又卡了……怎么回事？
* CPU使用率怎么这么高？
* 内存为什么吃了好几G？

老衲非常同情他们的遭遇，毕竟老衲也明白生产工具对生产效率的重要性，于是隔三差五都会研究研究怎么改进，但终究没有什么进展，不过大概知道原因：GoSublime 带的工具 margo 在 Go 代码变更时，会不断地从 Samba 中读取文件，造成很大的 IO 压力，在代码目录很大时这种现象就更明显，出现异常的机率也越大。

近日，老衲有缘结识了 `gopls`，深信它将是彻底这个问题的突破口。

<!--more-->


# gopls 是什么？

`gopls`是 Golang 官方开发的一个工具：https://github.com/golang/go/wiki/gopls

{{% admonition quote "gopls是什么？" %}}
gopls (pronounced: "go please") is an implementation of the Language Server Protocol (LSP) server for Go. The LSP allows any text editor to be extended with IDE-like features (see https://langserver.org/ for details). It is currently in alpha, so it is not stable.
{{% /admonition %}}

简单翻译如下：

{{% admonition quote "gopls是什么（译文）？" %}}
`gopls`（读作“go please”）是一个 Language Server Protocol（LSP，语言服务器协议），LSP 允许任意文本编辑器扩展类似IDE集成开发环境的功能（请参阅 https://langserver.org/ ）。`gopls` 目前在 alpha 开发状态，所以它可能还不稳定。
{{% /admonition %}}

## 为什么要用它？

老衲觉得这篇文章解释得很清楚了：统一。

[**GopherCon 2019 - Go, pls stop breaking my editor**](https://about.sourcegraph.com/go/gophercon-2019-go-pls-stop-breaking-my-editor)

来两张对比图，很直观地看出它们在 Go 语言 IDE 方面的支持复杂度。

未使用 Language Server：
![](sublime-text-remote-gopls-lsp0.png)

使用 Language Server：
![](sublime-text-remote-gopls-lsp1.png)

# 安装 gopls

```bash
go get golang.org/x/tools/gopls@latest
```

安装后查看下命令帮助：

```log
sunwind@sunwind-dev:~/LSP$ gopls version
version v0.1.3, built in $GOPATH mode
sunwind@sunwind-dev:~/LSP$ gopls --help   
The Go Language source tools.

Usage: gopls [flags] <command> [command-flags] [command-args]

Available commands are:
  serve : run a server for Go code using the Language Server Protocol
  bug : report a bug in gopls
  check : show diagnostic results for the specified file
  format : format the code according to the go standard
  query : answer queries about go source code
  version : print the gopls version information

gopls flags are:
  -debug string
        Serve debug information on the supplied address
  -listen string
        address on which to listen for remote connections
  -logfile string
        filename to log to. if value is "auto", then logging to a default output file is enabled
  -mode string
        no effect
  -ocagent string
        The address of the ocagent, or off (default "off")
  -port int
        port on which to run gopls for debugging purposes
  -profile.cpu string
        write CPU profile to this file
  -profile.mem string
        write memory profile to this file
  -profile.trace string
        write trace log to this file
  -remote string
        *EXPERIMENTAL* - forward all commands to a remote lsp
  -rpc.trace
        Print the full rpc trace in lsp inspector format
  -v    Verbose output
```

## 启动 gopls 服务器

```bash
gopls -v serve -listen :10888 -rpc.trace
```

这里为了方便调试，增加了 `-rpc.trace`，调试完成后可以不必再带这个参数。

调试完成后，可以使用 supervisor 托管这个进程。

# 在 Sublime Text 中安装 gopls 支持

安装 `gopls` 支持，其实就是安装 `gopls` 客户端，然后通过这个客户端连接到上一步启动的 `gopls` 服务器。

Wiki 页面中提供了多种编辑器的安装方法，其中就包括 Sublime Text 的扩展安装，但是由于目前官方提供的扩展还不支持连接到远程的 `gopls` 服务器，需要做一些修改，所以老衲直接使用 git clone 了代码到 Sublime Text 的扩展目录下，Win10下目录类似于：`C:\Users\sunwind\AppData\Roaming\Sublime Text 3\Packages\LSP\`。

修改后的代码老衲提交到了：https://github.com/laonadev/LSP ，各位施主可略为参考。

主要改动：

* 增加对连接到远程 `gopls` 服务器的支持
  也就是让 Sublime Text 连接到远程的 `gopls` 服务器，而不是在本地启动

* 增加对本地路径和远程路径的映射支持
  也就是让 Sublime Text 操作 Z: 盘下源文件时，远程的 `gopls` 服务器分析对应目录下的源文件。
  如：`Z:\myproj\main.go` => `/home/sunwind/myproj/main.go`

改动过的代码（注意要使用空格缩进）：

1. plugin/core/sessions.py

	```python
	            transport = start_tcp_transport(config.tcp_port)
	```
	替换为：
	```python
	            transport = start_tcp_transport(config.tcp_port, config.tcp_host)
	```

2. plugin/core/url.py

	```python
	    return urljoin('file:', pathname2url(path))
	```
	替换为：
	```python
	    return urljoin('file:', pathname2url(path)).replace("file:///Z:/", "file:///home/sunwind/")
	```

	```python
	def uri_to_filename(uri: str) -> str:
	```
	后面插入一行：
	```python
	    uri = uri.replace("file:///home/sunwind/", "file:///Z:/")
	```

这里的路径需要根据你自己的实际情况替换掉，老衲为了省事直接写死了，没有考虑写进LSP扩展的用户配置文件里。

## 启用 gopls

1. 打开命令面板（Ctrl+Shift+P）
2. 找到并执行 “LSP: Enable Language Server Globally”
   ![](sublime-text-remote-gopls-lsp2.png)
3. 选择 gopls，注意别选错了(有个叫 golsp，很容易混淆）
4. 打开配置文件：
   ![](sublime-text-remote-gopls-lsp3.png)      
   输入：
	```json
	{
		"clients":
		{
			"gopls":
			{
				"command":
				[
				],
				"enabled": true,
				"languageId": "go",
				"scopes":
				[
					"source.go"
				],
				"syntaxes":
				[
					"Packages/Go/Go.sublime-syntax"
				],
				"tcp_host": "192.168.137.2",
				"tcp_port": 10888
			}
		}
	}
	```

	![](sublime-text-remote-gopls-lsp4.png)

5. 保存后重启 Sublime Text


# 成果展示

自动完成和语法提示：

![](sublime-text-remote-gopls-lsp5.png)

类型提示：

![](sublime-text-remote-gopls-lsp6.png)

虽然目前 gopls 还有一些问题，但重要的是，一点都不卡了！