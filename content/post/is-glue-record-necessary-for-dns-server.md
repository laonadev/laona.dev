---
title: "搭建权威 DNS 服务器必须设置胶水记录（Glue Record）吗？"
date: 2019-08-01T18:50:54+08:00
lastmod: 2019-08-01T18:50:54+08:00
draft: false
keywords: ["胶水记录", "DNS", "Glue Record"]
description: "实际上，在一些情况下，胶水记录并不是必须的。"
tags: ["DNS"]
categories: ["运维"]
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

庙里来了个新香目，和 DNS 服务器相关的，老衲今天和小和尚们针对这个项目讨论了一个问题：

因为许多域名注册商都不支持设置胶水记录（Glue Record），在搭建权威 DNS 服务器时，是否必须设置胶水记录？

这得先从 Glue Record 的概念讲起了……

<!--more-->

# 胶水记录的概念

请参考老衲上一篇文章里写的：

[《胶水记录（Glue Record）是什么？有什么作用？》](https://laona.dev/post/glue-record/)

总结起来就是：**减少递归查询次数，加快 DNS 递归查询**

# 胶水记录生效的前提

胶水记录设置后，能够减少递归查询次数，但老衲总结了一个生效的前提：

**“域名的父级域”和“域名的 DNS 服务器域名的父级域”使用同一组权威服务器。**

比如 `jd.com` 的权威服务器为 `ns*.jdcache.com`，而且 `jd.com` 和 `jdcache.com` 的父级域都是 `.com`，权威服务器都是 `*.gtld-servers.net`。

这样在向 `.com` 权威服务器 `*.gtld-servers.net` 查询 `jd.com` 时，服务器才能在返回 `ns*.jdcache.com` 的同时，顺便返回登记在案的 `ns*.jdcache.com` 的 IP 地址。

**如果没有满足这个前提条件，胶水记录还会生效吗？**

来看看 `laona.dev`，老衲将它托管在 DNSPod 解析：

* DNS服务器：`f1g1ns1.dnspod.net`、`f1g1ns2.dnspod.net`

* 域名的父级域：`.dev`，权威服务器为 `ns-tld*.charlestonroadregistry.com`

* DNS服务器域名的父级域：`.net`，权威服务器为 `*.gtld-servers.net`

两组权威服务器不同，老衲用 `dig @ns-tld1.charlestonroadregistry.com laona.dev` 查查看是否返回了 DNSPod 的胶水记录：

```
# dig @ns-tld1.charlestonroadregistry.com laona.dev

; <<>> DiG 9.11.5-P1-1ubuntu2.5-Ubuntu <<>> @ns-tld1.charlestonroadregistry.com laona.dev
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4730
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 2, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;laona.dev.                     IN      A

;; AUTHORITY SECTION:
laona.dev.              10800   IN      NS      f1g1ns1.dnspod.net.
laona.dev.              10800   IN      NS      f1g1ns2.dnspod.net.

;; Query time: 61 msec
;; SERVER: 216.239.32.105#53(216.239.32.105)
;; WHEN: Thu Aug 01 19:21:42 CST 2019
;; MSG SIZE  rcvd: 92
```

如同所料，`.dev` 的权威服务器 `ns-tld*.charlestonroadregistry.com` 中并没有保存 `dnspod.net` 的胶水记录，因此只返回了负责解析 `laona.dev` 的权威服务器名称，没有返回这组权威服务器的 IP 地址。

那么怎么得到这组权威服务器的 IP 地址呢？通过查询对应的 A 记录获得。

# 域名注册局的限制

许多域名在修改 DNS 服务器时，会检查 DNS 服务器对应的胶水记录是否存在，如果不存在将保存失败，比如：

阿里云的域名：

![](is-glue-record-necessary-for-dns-server2.png)

泡米网的域名：

![](is-glue-record-necessary-for-dns-server3.png)

这类检测只检测同父级权威服务器中是否有指定 DNS 服务器的胶水记录，如果 DNS 服务器的父级域和域名的父级域不同，则不会进行检测。

比如将 `****.com` 域名的 DNS 服务器设置为 `ns*.****.app`，即使这个 `ns*.****.app` 未设置胶水记录，仍然会保存成功。


# 结论

即使 DNS 服务器的域名设置了胶水记录，在查询过程中胶水记录也不是总会使用到的。

如果没有设置胶水记录，虽然不会影响 DNS 服务器工作，只是在某些情况下起不到加速递归查询的作用，但是这个 DNS 服务器将无法作为同父级域下域名的权威 DNS 服务器，只能作为其它父级域下域名的权威 DNS 服务器。

如果施主像老衲一样准备搭建权威 DNS 服务器，为了使胶水记录能在最大程度上生效，在选择域名时，老衲会给你这样的建议：

**使用和主要用户群体的域名后缀相同的后缀，或同一机构管理的后缀，作为权威 DNS 服务器的域名。**

比如用户主要使用 `.com` 和 `.net`，就选择使用 `ns*.yourdomain.com` 或 `ns*.yourdomain.net` 作为权威 DNS 服务器域名，因为 `.com` 和 `.net` 都由 VeriSign 管理。

如果施主的域名注册商不支持设置胶水记录，或施主不想为设置胶水记录付费，可以使用一些用户不怎么使用的小众后缀域名作为 DNS 服务器域名，这样就不需要设置胶水记录了，以避开注册商和注册局的限制。


# 老衲的疑问

老衲发现将域名作为权威 DNS 服务器域名时，比如下面的 `ns*.ming.app`，即使设置了胶水记录，在查询 `jidan.app` 这个使用 `ns*.ming.app` 作为权威 DNS 服务器的域名时也不会返回胶水记录：

![](is-glue-record-necessary-for-dns-server1.png)

```
# dig @ns-tld1.charlestonroadregistry.com jidan.app 

; <<>> DiG 9.11.5-P1-1ubuntu2.5-Ubuntu <<>> @ns-tld1.charlestonroadregistry.com jidan.app
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44163
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 2, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;jidan.app.                     IN      A

;; AUTHORITY SECTION:
jidan.app.              10800   IN      NS      ns1.ming.app.
jidan.app.              10800   IN      NS      ns2.ming.app.

;; Query time: 63 msec
;; SERVER: 216.239.32.105#53(216.239.32.105)
;; WHEN: Thu Aug 01 19:59:59 CST 2019
;; MSG SIZE  rcvd: 79
```

但是，如果把 `ming.app` 交给自己解析后，胶水记录会生效：

```
# dig @ns-tld1.charlestonroadregistry.com jidan.app

; <<>> DiG 9.11.5-P1-1ubuntu2.5-Ubuntu <<>> @ns-tld1.charlestonroadregistry.com jidan.app
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63383
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 2, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;jidan.app.                     IN      A

;; AUTHORITY SECTION:
jidan.app.              10800   IN      NS      ns1.ming.app.
jidan.app.              10800   IN      NS      ns2.ming.app.

;; ADDITIONAL SECTION:
ns1.ming.app.           3600    IN      A       103.102.--.--
ns2.ming.app.           3600    IN      A       103.102.--.--

;; Query time: 65 msec
;; SERVER: 216.239.32.105#53(216.239.32.105)
;; WHEN: Thu Aug 01 20:06:45 CST 2019
;; MSG SIZE  rcvd: 111
```

老衲猜测原因是：

如果 `ming.app` 是交给其它权威 DNS 服务器解析的，那么要查询 `ns*.ming.app` 对应的 IP 地址，到底是以登记在其它 DNS 服务器中的 A 记录为准，还是以胶水记录为准呢？这样容易产生不一致的结果。

期待哪位施主如果看到 RFC 文档中有相关章节有关于它的描述时，知会下老衲。