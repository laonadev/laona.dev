---
title: "胶水记录（Glue Record）是什么？有什么作用？"
date: 2019-08-01T16:58:42+08:00
lastmod: 2019-08-01T16:58:42+08:00
draft: false
keywords: ["DNS", "胶水记录", "Glue Record"]
description: "通过DNS查询实例分析胶水记录（Glue Record）在查询过程中的作用。"
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

# 什么是胶水记录

“为什么叫胶水（Glue）记录，是不是很粘？粘在哪？”

“这个胶水和大家说的 Python 语言是一门胶水语言，是一回事吗？”

<!--more-->

胶水记录是一种特殊的记录类型，来对比下它和普通的 DNS 记录有什么差别：

* 普通的 DNS 记录是保存在权威服务器上的

	* 权威服务器一般由域名注册商提供，当然也有由第三方提供的独立解析服务。

	* 用户通过域名解析后台设置解析记录，值是保存在权威服务器上的。

* 胶水记录保存在注册局的DNS服务器上

	* 注册局（Domain Registry）是管理某个后缀所有域名的机构。

	* 用户通过域名注册商后台设置胶水记录时，值是保存到注册局的服务器上的。


**有施主问，为什么我在注册商后台找不到设置胶水记录的地方？**

原因：

* 不是所有的注册商都支持，也不一定都叫胶水记录设置，比如国内一般叫“注册DNS服务器”
* 有些注册商要求注册一定年限以上的域名才能使用这个功能，还有些直接对这个功能额外收费

回到正题，我还是没明白什么是胶水记录？要想弄清楚，还得先从 DNS 查询的过程说起。

# DNS 查询过程

以查询 `jd.com` 这个域名的 A 记录为例，使用 `dig` 命令来分析整个查询过程。

我们会以递归服务器的工作方式来逐级查询 DNS 记录。

## 一、获取根服务器列表

为什么要根服务器列表？因为所有域名的 DNS 记录查询入口都在这里，就好像你要进入一个多级的目录，你得一级一级地进入，而根服务器就是入口。

全球共有 13 组公认的 DNS 服务器，可在 IANA 网站上查询到：

https://www.iana.org/domains/root/servers

![](is-glue-record-necessary-for-dns-server0.png)

所有递归服务器上都会内置这张表。

## 二、向根服务器发送查询

老衲随机选取一个根服务器进行查询，这里老衲选了 `m.root-servers.net`，它的IP地址是 `202.12.27.33`。

下面是和这台根服务器的对话过程：

“嗨，请问你这有 jd.com 吗？”

“没有，但是你可以找管理 .com 域名的这些服务器问问。哦，对了，他们的 IP 地址我这刚好有，直接给你吧！”

```
# dig @202.12.27.33 jd.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> @202.12.27.33 jd.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29264
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 13, ADDITIONAL: 27
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;jd.com.                                IN      A

;; AUTHORITY SECTION:
com.                    172800  IN      NS      d.gtld-servers.net.
com.                    172800  IN      NS      f.gtld-servers.net.
com.                    172800  IN      NS      e.gtld-servers.net.
com.                    172800  IN      NS      g.gtld-servers.net.
com.                    172800  IN      NS      k.gtld-servers.net.
com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      c.gtld-servers.net.
com.                    172800  IN      NS      j.gtld-servers.net.
com.                    172800  IN      NS      l.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
com.                    172800  IN      NS      m.gtld-servers.net.
com.                    172800  IN      NS      i.gtld-servers.net.
com.                    172800  IN      NS      h.gtld-servers.net.

;; ADDITIONAL SECTION:
a.gtld-servers.net.     172800  IN      A       192.5.6.30
b.gtld-servers.net.     172800  IN      A       192.33.14.30
c.gtld-servers.net.     172800  IN      A       192.26.92.30
d.gtld-servers.net.     172800  IN      A       192.31.80.30
e.gtld-servers.net.     172800  IN      A       192.12.94.30
f.gtld-servers.net.     172800  IN      A       192.35.51.30
g.gtld-servers.net.     172800  IN      A       192.42.93.30
h.gtld-servers.net.     172800  IN      A       192.54.112.30
i.gtld-servers.net.     172800  IN      A       192.43.172.30
j.gtld-servers.net.     172800  IN      A       192.48.79.30
k.gtld-servers.net.     172800  IN      A       192.52.178.30
l.gtld-servers.net.     172800  IN      A       192.41.162.30
m.gtld-servers.net.     172800  IN      A       192.55.83.30
a.gtld-servers.net.     172800  IN      AAAA    2001:503:a83e::2:30
b.gtld-servers.net.     172800  IN      AAAA    2001:503:231d::2:30
c.gtld-servers.net.     172800  IN      AAAA    2001:503:83eb::30
d.gtld-servers.net.     172800  IN      AAAA    2001:500:856e::30
e.gtld-servers.net.     172800  IN      AAAA    2001:502:1ca1::30
f.gtld-servers.net.     172800  IN      AAAA    2001:503:d414::30
g.gtld-servers.net.     172800  IN      AAAA    2001:503:eea3::30
h.gtld-servers.net.     172800  IN      AAAA    2001:502:8cc::30
i.gtld-servers.net.     172800  IN      AAAA    2001:503:39c1::30
j.gtld-servers.net.     172800  IN      AAAA    2001:502:7094::30
k.gtld-servers.net.     172800  IN      AAAA    2001:503:d2d::30
l.gtld-servers.net.     172800  IN      AAAA    2001:500:d937::30
m.gtld-servers.net.     172800  IN      AAAA    2001:501:b1f9::30

;; Query time: 211 msec
;; SERVER: 202.12.27.33#53(202.12.27.33)
;; WHEN: Thu Aug 01 18:13:03 CST 2019
;; MSG SIZE  rcvd: 831
```

根服务器告诉老衲，`.com` 由 `*.gtld-servers.net` 这组服务器管理，这组服务器实际上就是 `.com` 域名的权威服务器，并且还顺便告诉了老衲他们的 IP 地址。

## 三：向 .com 权威服务器发送查询

老衲随机选取一个 `.com` 权威服务器进行查询，这里老衲选了 `a.gtld-servers.net`，它的IP地址是 `192.5.6.30`。

下面是和这台 `.com` 权威服务器的对话过程：

“嗨，请问你这有 jd.com 吗？”

“没有，但是你可以找管理 jd.com 域名的这些服务器问问。哦，对了，他们的 IP 地址我这刚好有，直接给你吧！”

```
# dig @192.5.6.30 jd.com  

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> @192.5.6.30 jd.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16955
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 4, ADDITIONAL: 5
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;jd.com.                                IN      A

;; AUTHORITY SECTION:
jd.com.                 172800  IN      NS      ns1.jdcache.com.
jd.com.                 172800  IN      NS      ns2.jdcache.com.
jd.com.                 172800  IN      NS      ns3.jdcache.com.
jd.com.                 172800  IN      NS      ns4.jdcache.com.

;; ADDITIONAL SECTION:
ns1.jdcache.com.        172800  IN      A       111.13.28.10
ns2.jdcache.com.        172800  IN      A       111.206.226.10
ns3.jdcache.com.        172800  IN      A       120.52.149.254
ns4.jdcache.com.        172800  IN      A       106.39.177.32

;; Query time: 3 msec
;; SERVER: 192.5.6.30#53(192.5.6.30)
;; WHEN: Thu Aug 01 18:20:21 CST 2019
;; MSG SIZE  rcvd: 179
```

`.com` 权威服务器告诉老衲，`jd.com` 由 `ns*.jdcache.com` 这组服务器管理，并且还顺便告诉了老衲他们的 IP 地址。

## 第四步：向 jd.com 域名权威服务器发送查询

老衲随机选取一个 `jd.com` 权威服务器进行查询，这里老衲选了 `ns1.jdcache.com`，它的IP地址是 `111.13.28.10`。

下面是和这台 `jd.com` 权威服务器的对话过程：

“嗨，请问你这有 jd.com 吗？”

“有的，就在我这里，给你吧！”

```
# dig @111.13.28.10 jd.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> @111.13.28.10 jd.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53607
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 8, ADDITIONAL: 9
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;jd.com.                                IN      A

;; ANSWER SECTION:
jd.com.                 120     IN      A       120.52.148.118

;; AUTHORITY SECTION:
jd.com.                 120     IN      NS      ns3.jd.com.
jd.com.                 120     IN      NS      ns3.jdcache.com.
jd.com.                 120     IN      NS      ns1.jdcache.com.
jd.com.                 120     IN      NS      ns1.jd.com.
jd.com.                 120     IN      NS      ns2.jdcache.com.
jd.com.                 120     IN      NS      ns2.jd.com.
jd.com.                 120     IN      NS      ns4.jdcache.com.
jd.com.                 120     IN      NS      ns4.jd.com.

;; ADDITIONAL SECTION:
ns1.jd.com.             120     IN      A       111.13.28.10
ns1.jdcache.com.        720     IN      A       111.13.28.10
ns2.jd.com.             120     IN      A       111.206.226.10
ns2.jdcache.com.        720     IN      A       111.206.226.10
ns3.jd.com.             120     IN      A       120.52.149.254
ns3.jdcache.com.        720     IN      A       120.52.149.254
ns4.jd.com.             120     IN      A       106.39.177.32
ns4.jdcache.com.        720     IN      A       106.39.177.32

;; Query time: 201 msec
;; SERVER: 111.13.28.10#53(111.13.28.10)
;; WHEN: Thu Aug 01 18:23:53 CST 2019
;; MSG SIZE  rcvd: 331
```

最终老衲得到了 `jd.com` 的 IP 地址为 `120.52.148.118`。

# 胶水记录在哪

在上面这部分查询过程中，其实有多台服务器返回了胶水记录：

1. 在第二步向根服务器查询返回的结果里，根服务器顺便告诉老衲的包含 `*.gtld-servers.net` 这组服务器的 IP 地址的记录，就是胶水记录。
2. 在第三步向 `.com` 权威服务器查询返回的结果里，`.com` 权威服务器顺便告诉老衲的包含 `ns*.jdcache.com` 这组服务器的 IP 地址的记录，就是胶水记录。

所以，胶水记录就是域管理者向上级域管理者提供的一组主机名和IP的映射表：

1. `.com` 域管理者（也就是 `.com` 域名注册局，VeriSign）向根域管理者（也就是 IANA）提供了：`.com` 的权威 DNS 服务器 `*.gtld-servers.net` 对应的 IP 地址
2. `jdcache.com` 域管理者向`.com` 域管理者（VeriSign）提供了：`jd.com` 的权威 DNS 服务器 `ns*.jdcache.com` 对应的 IP 地址

也就是由域的权威 DNS 服务器服务者提供的 DNS 服务器对应的 IP 地址数据。

# 胶水记录的主要作用

试想下：

* 如果：没有设置这些胶水记录

* 那么：这些服务器就无法**顺便**返回给老衲这些DNS服务器IP

* 结果：老衲要想得到这些DNS服务器的IP，就得额外查询

所以胶水记录的作用就是：**减少递归查询次数，加快 DNS 递归查询**。

老衲认为，正是由于它是和 DNS 查询结果一起**顺便**返回的，所以才叫做**胶水**记录。

# 胶水记录的其它作用

如果将域名将给自己搭建的权威 DNS 服务器，会如何呢？

以下以 `ming.app` 使用权威 DNS 服务器 `ns1.ming.app`、`ns2.ming.app`为示例做个实验：

![](glue-record0.png)


```
 dig +trace +all www.ming.app

; <<>> DiG 9.11.5-P1-1ubuntu2.5-Ubuntu <<>> +trace +all www.ming.app
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44062
;; flags: qr ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;.                              IN      NS

;; ANSWER SECTION:
.                       491071  IN      NS      d.root-servers.net.
.                       491071  IN      NS      c.root-servers.net.
.                       491071  IN      NS      e.root-servers.net.
.                       491071  IN      NS      h.root-servers.net.
.                       491071  IN      NS      b.root-servers.net.
.                       491071  IN      NS      m.root-servers.net.
.                       491071  IN      NS      j.root-servers.net.
.                       491071  IN      NS      a.root-servers.net.
.                       491071  IN      NS      i.root-servers.net.
.                       491071  IN      NS      f.root-servers.net.
.                       491071  IN      NS      l.root-servers.net.
.                       491071  IN      NS      g.root-servers.net.
.                       491071  IN      NS      k.root-servers.net.

;; Query time: 30 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Aug 01 20:30:55 CST 2019
;; MSG SIZE  rcvd: 239

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42488
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 7, ADDITIONAL: 11

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 1472
;; QUESTION SECTION:
;www.ming.app.                  IN      A

;; AUTHORITY SECTION:
app.                    172800  IN      NS      ns-tld1.charlestonroadregistry.com.
app.                    172800  IN      NS      ns-tld2.charlestonroadregistry.com.
app.                    172800  IN      NS      ns-tld3.charlestonroadregistry.com.
app.                    172800  IN      NS      ns-tld4.charlestonroadregistry.com.
app.                    172800  IN      NS      ns-tld5.charlestonroadregistry.com.
app.                    86400   IN      DS      23684 8 2 3A5CC8A31E02C94ABA6461912FABB7E9F5E34957BB6114A55A864D96 AEC31836
app.                    86400   IN      RRSIG   DS 8 1 86400 20190814050000 20190801040000 59944 . AiczCFnzwYcTmvZwd7xs3w81JtrT0pNSiHQw78RAABTYD12r07jicZBT AubL/XJ5fbWQseCf6KNkvzqN2JI5a5JCmFy+2pr470HBiMlBisKE6/3B 37sP3A7m1bQzwqJzEc5gv6CvBxkJEVcbLLhwgyyJxz268+yCykF/2viN vUyy1Y0sjM+Q98uuJ/UbL0Hsi+Ie4HYRwA4/P21tmYfTav399Xuv8eD4 YpI4r13/PQO/EoVXti+Sj9Sv+ze+nlhxDDUaw8lcyiLH8ztmxICS+MJr k398s2KHHQ8lHmhYPEqQQEOIen2zpRJWkK8s4iKzHTiwUykxzpLuCIXy 6gQBOw==

;; ADDITIONAL SECTION:
ns-tld1.charlestonroadregistry.com. 172800 IN A 216.239.32.105
ns-tld1.charlestonroadregistry.com. 172800 IN AAAA 2001:4860:4802:32::69
ns-tld2.charlestonroadregistry.com. 172800 IN A 216.239.34.105
ns-tld2.charlestonroadregistry.com. 172800 IN AAAA 2001:4860:4802:34::69
ns-tld3.charlestonroadregistry.com. 172800 IN A 216.239.36.105
ns-tld3.charlestonroadregistry.com. 172800 IN AAAA 2001:4860:4802:36::69
ns-tld4.charlestonroadregistry.com. 172800 IN A 216.239.38.105
ns-tld4.charlestonroadregistry.com. 172800 IN AAAA 2001:4860:4802:38::69
ns-tld5.charlestonroadregistry.com. 172800 IN A 216.239.60.105
ns-tld5.charlestonroadregistry.com. 172800 IN AAAA 2001:4860:4805::69

;; Query time: 174 msec
;; SERVER: 192.203.230.10#53(192.203.230.10)
;; WHEN: Thu Aug 01 20:30:56 CST 2019
;; MSG SIZE  rcvd: 732

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29297
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 4, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 512
;; QUESTION SECTION:
;www.ming.app.                  IN      A

;; AUTHORITY SECTION:
ming.app.               10800   IN      NS      ns1.ming.app.
ming.app.               10800   IN      NS      ns2.ming.app.
rs5isqbli69sbr4fcjatu0sn1tfvka81.app. 900 IN NSEC3 1 0 1 139FCA6C02909936 RS5NLJ9CB29IIS9NLO4KRT2RUNI3NJ4M NS
rs5isqbli69sbr4fcjatu0sn1tfvka81.app. 900 IN RRSIG NSEC3 8 2 900 20190819163840 20190728163840 7866 app. oW/X7K5bBZflQD1wSsSsaG3CQS75PJN2Gg5k/fyKKRlfhh9PbXFAFKCE PU82m9bRENrYRMTw9CYaM3pwdcWFRBtTruQsz7pRsGeCBukr5bOJU8aJ YoMUH6n0v5oAjhMv4Y2UUA81NbCWPJjMvFOVvI+MKFdQrcdpZt0uf1aj Q1U=

;; ADDITIONAL SECTION:
ns1.ming.app.           3600    IN      A       103.102.--.--
ns2.ming.app.           3600    IN      A       103.102.--.--

;; Query time: 62 msec
;; SERVER: 216.239.32.105#53(216.239.32.105)
;; WHEN: Thu Aug 01 20:30:57 CST 2019
;; MSG SIZE  rcvd: 354

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26285
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.ming.app.                  IN      A

;; ANSWER SECTION:
www.ming.app.           300     IN      A       127.0.0.1

;; Query time: 153 msec
;; SERVER: 103.102.--.--#53(103.102.--.--)
;; WHEN: Thu Aug 01 20:30:57 CST 2019
;; MSG SIZE  rcvd: 58
```

如果没有返回下面这些胶水记录，要去哪里查 `ns1.ming.app` 和 `ns2.ming.app` 对应的 IP 地址呢？

```
;; ADDITIONAL SECTION:
ns1.ming.app.           3600    IN      A       103.102.--.--
ns2.ming.app.           3600    IN      A       103.102.--.--
```

所以胶水记录的另一个作用是：**域名作为权威 DNS 服务器时，使解析自身域名成为可能。**