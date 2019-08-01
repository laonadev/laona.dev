---
title: "dig +trace 不等同于递归查询"
date: 2019-08-02T02:19:56+08:00
lastmod: 2019-08-02T02:19:56+08:00
draft: false
keywords: ["dig", "recursion"]
description: "dig +trace 会忽略胶水记录，和实际的递归服务器查询过程不一样。"
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

老衲昨天写了两篇文章，都与胶水记录（Glue Record）有关：

* [《胶水记录（Glue Record）是什么？有什么作用？》](https://laona.dev/post/glue-record/)
* [《搭建权威 DNS 服务器必须设置胶水记录（Glue Record）吗？》](https://laona.dev/post/is-glue-record-necessary-for-dns-server/)

其中大量使用了 `dig` 命令演示递归查询过程，对于它的准确性一直表示怀疑，因此今天特地做了一番实验进行验证，最终发现 `dig +trace` 命令和递归服务器的递归查询过程的确不一样。

<!--more-->

还是以递归查询 `jd.com` 为例：

# dig 9.9.4 实验

```
# dig +trace +additional jd.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> +trace +additional jd.com
;; global options: +cmd
.                       154569  IN      NS      m.root-servers.net.
.                       154569  IN      NS      b.root-servers.net.
.                       154569  IN      NS      c.root-servers.net.
.                       154569  IN      NS      d.root-servers.net.
.                       154569  IN      NS      e.root-servers.net.
.                       154569  IN      NS      f.root-servers.net.
.                       154569  IN      NS      g.root-servers.net.
.                       154569  IN      NS      h.root-servers.net.
.                       154569  IN      NS      a.root-servers.net.
.                       154569  IN      NS      i.root-servers.net.
.                       154569  IN      NS      j.root-servers.net.
.                       154569  IN      NS      k.root-servers.net.
.                       154569  IN      NS      l.root-servers.net.
.                       154569  IN      RRSIG   NS 8 0 518400 20190813050000 20190731040000 59944 . SEMFJ6/3EEZWMhYUHkFu3wp6+9KNADqacI27Eo6Zv/N2x8nKrlFb01F4 AYvNkuRMRXpJwtch8N7hc/eZm4Qjommc/2sP1NygHriLr2RMqyRPklYr URrOkN+TP3KCwOSPRLvAzJnCayrXU7jnnz4x0l4enIjy6/nFkpVEfemv VLMHcM3btzrcbvjOzkNFE/7UHSwBOIFe6D5Q5uL9dlBNpDJf7HI/5j4v dZnbtBBnlYorvLJ01wEc6ZbceJPz67cA2W8sLWhdjnbXJWlexAFoK+FF +msbDAaci908ks7rnwh9/icGcuHCXm+nHrsufEXh92hG7qyQKKk9frLD 56sK0A==
;; Received 525 bytes from 8.8.8.8#53(8.8.8.8) in 43 ms

com.                    172800  IN      NS      l.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
com.                    172800  IN      NS      c.gtld-servers.net.
com.                    172800  IN      NS      d.gtld-servers.net.
com.                    172800  IN      NS      e.gtld-servers.net.
com.                    172800  IN      NS      f.gtld-servers.net.
com.                    172800  IN      NS      g.gtld-servers.net.
com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      h.gtld-servers.net.
com.                    172800  IN      NS      i.gtld-servers.net.
com.                    172800  IN      NS      j.gtld-servers.net.
com.                    172800  IN      NS      k.gtld-servers.net.
com.                    172800  IN      NS      m.gtld-servers.net.
com.                    86400   IN      DS      30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.                    86400   IN      RRSIG   DS 8 1 86400 20190814170000 20190801160000 59944 . q7wAN0bPBGlgcN+edSDbKVc2KzcqBXKuI8RECDtpPK2c7zm7HXBS4VLi AVP3g1fDPw0Z3DgtgM6VmMaWmHaUtAtKuSTGyLipfL+PdIXTQQsyDYUb 0IrjmKdFdKJphM6qtQvjrYShsSfIHLFasMDXIe0/x88yDaY8Aw8JsOJW Tc/VAovF7P98GgnqB7uNZFeuKYi/38DzjOYhXUJX3JYRocsv4eb09cay 4n/d7osYvGh+2THk85uptHTzQXHcuQ0CzLWI7wDA/ukj7asdauauHJOp D1qAnYKwVVEGFZ1pPwO8SgBF2KZACfGkNQCivD7VruugmKvkD6KSmEvl GCZHnw==
l.gtld-servers.net.     172800  IN      A       192.41.162.30
l.gtld-servers.net.     172800  IN      AAAA    2001:500:d937::30
b.gtld-servers.net.     172800  IN      A       192.33.14.30
b.gtld-servers.net.     172800  IN      AAAA    2001:503:231d::2:30
c.gtld-servers.net.     172800  IN      A       192.26.92.30
c.gtld-servers.net.     172800  IN      AAAA    2001:503:83eb::30
d.gtld-servers.net.     172800  IN      A       192.31.80.30
d.gtld-servers.net.     172800  IN      AAAA    2001:500:856e::30
e.gtld-servers.net.     172800  IN      A       192.12.94.30
e.gtld-servers.net.     172800  IN      AAAA    2001:502:1ca1::30
f.gtld-servers.net.     172800  IN      A       192.35.51.30
f.gtld-servers.net.     172800  IN      AAAA    2001:503:d414::30
g.gtld-servers.net.     172800  IN      A       192.42.93.30
g.gtld-servers.net.     172800  IN      AAAA    2001:503:eea3::30
a.gtld-servers.net.     172800  IN      A       192.5.6.30
a.gtld-servers.net.     172800  IN      AAAA    2001:503:a83e::2:30
h.gtld-servers.net.     172800  IN      A       192.54.112.30
h.gtld-servers.net.     172800  IN      AAAA    2001:502:8cc::30
i.gtld-servers.net.     172800  IN      A       192.43.172.30
i.gtld-servers.net.     172800  IN      AAAA    2001:503:39c1::30
j.gtld-servers.net.     172800  IN      A       192.48.79.30
j.gtld-servers.net.     172800  IN      AAAA    2001:502:7094::30
k.gtld-servers.net.     172800  IN      A       192.52.178.30
k.gtld-servers.net.     172800  IN      AAAA    2001:503:d2d::30
m.gtld-servers.net.     172800  IN      A       192.55.83.30
m.gtld-servers.net.     172800  IN      AAAA    2001:501:b1f9::30
;; Received 1166 bytes from 192.203.230.10#53(e.root-servers.net) in 39 ms

jd.com.                 172800  IN      NS      ns1.jdcache.com.
jd.com.                 172800  IN      NS      ns2.jdcache.com.
jd.com.                 172800  IN      NS      ns3.jdcache.com.
jd.com.                 172800  IN      NS      ns4.jdcache.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN NSEC3 1 1 0 - CK0Q1GIN43N1ARRC9OSM6QPQR81H5M9A NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN RRSIG NSEC3 8 2 86400 20190807044529 20190731033529 17708 com. zH+3/u7U02O4Hu/leXwkHCaL+tMktPL9uFMxc0YzWj8h4Btgaw2cofp/ t/jpQvv9nGJXatSYucoCE0f14Rw4CyigHG2j3LmxBTDIoewTXQ7wMbYo TUFiEh0kXcZMp3+irzTxlPI9+SebEpQkUbYz93eKDbg5nC3+tyE6o/3E Nl4=
VCN61V6E8RJN4R24GKDTFMUBGC610E4P.com. 86400 IN NSEC3 1 1 0 - VCN77CSSQ0F81UJ2JQ85RQR2AF15I71Q NS DS RRSIG
VCN61V6E8RJN4R24GKDTFMUBGC610E4P.com. 86400 IN RRSIG NSEC3 8 2 86400 20190808045137 20190801034137 17708 com. ZeT3GsyO3s7eumN3G5a9mjxKQ2gmDBf7pF0CIzq5ohWfmfBLY198j2d1 Qj+kTkhFnNX4p2bIFDu1YWdKOzquNg/6ce/yHxCjiwAYa0eQo5TwXOaj 2VfDaUR7z8HcNjfWMpPgFXWKw/aE29CoPxf9s/mYpeI7adp7IgKlLWJr zgs=
ns1.jdcache.com.        172800  IN      A       111.13.28.10
ns2.jdcache.com.        172800  IN      A       111.206.226.10
ns3.jdcache.com.        172800  IN      A       120.52.149.254
ns4.jdcache.com.        172800  IN      A       106.39.177.32
;; Received 664 bytes from 192.33.14.30#53(b.gtld-servers.net) in 362 ms

jd.com.                 120     IN      A       120.52.148.118
jd.com.                 120     IN      NS      ns1.jd.com.
jd.com.                 120     IN      NS      ns2.jdcache.com.
jd.com.                 120     IN      NS      ns2.jd.com.
jd.com.                 120     IN      NS      ns4.jdcache.com.
jd.com.                 120     IN      NS      ns3.jdcache.com.
jd.com.                 120     IN      NS      ns3.jd.com.
jd.com.                 120     IN      NS      ns4.jd.com.
jd.com.                 120     IN      NS      ns1.jdcache.com.
ns1.jd.com.             120     IN      A       111.13.28.10
ns1.jdcache.com.        720     IN      A       111.13.28.10
ns2.jd.com.             120     IN      A       111.206.226.10
ns2.jdcache.com.        720     IN      A       111.206.226.10
ns3.jd.com.             120     IN      A       120.52.149.254
ns3.jdcache.com.        720     IN      A       120.52.149.254
ns4.jd.com.             120     IN      A       106.39.177.32
ns4.jdcache.com.        720     IN      A       106.39.177.32
;; Received 331 bytes from 106.39.177.32#53(ns4.jdcache.com) in 126 ms
```

在每次返回 NS 记录时，都会有胶水记录返回 DNS 服务器对应 IP 地址（A/AAAA 记录），**按照老衲的理解**，递归进入下一步查询时，可以直接使用胶水记录中的 IP 地址作为 DNS 服务器的地址。

然而实际情况使用 `tcpdump` 抓包后一看，大吃一惊！

即使返回了胶水记录，`dig` 还是针对：

* 每个根服务器：`*.root-servers.net` 
* 每个 `.com` 域的权威 DNS 服务器：`*.gtld-servers.net` 
* 每个域名的权威 DNS 服务器：`ns*.jdcache.com` 

做了一次 A/AAAA 记录查询，**完全忽略了胶水记录**，在 `tcpdump` 命令输出过程中，明显感觉到了延时。

tcpdump 输出：

```
# tcpdump -nnn "udp and port 53"                 
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
03:02:08.408728 IP 103.102.45.214.59374 > 8.8.8.8.53: 54799 [1au] NS? . (28)
03:02:08.411778 IP 8.8.8.8.53 > 103.102.45.214.59374: 54799$ 14/0/1 NS m.root-servers.net., NS b.root-servers.net., NS c.root-servers.net., NS d.root-servers.net., NS e.root-servers.net., NS f.root-servers.net., NS g.root-servers.net., NS h.root-servers.net., NS a.root-servers.net., NS i.root-servers.net., NS j.root-servers.net., NS k.root-servers.net., NS l.root-servers.net., RRSIG (525)
03:02:08.413083 IP 103.102.45.214.54272 > 8.8.8.8.53: 50422+ A? m.root-servers.net. (36)
03:02:08.413096 IP 103.102.45.214.54272 > 8.8.8.8.53: 39561+ AAAA? m.root-servers.net. (36)
03:02:08.415264 IP 8.8.8.8.53 > 103.102.45.214.54272: 50422 1/0/0 A 202.12.27.33 (52)
03:02:08.415286 IP 8.8.8.8.53 > 103.102.45.214.54272: 39561 1/0/0 AAAA 2001:dc3::35 (64)
03:02:08.415844 IP 103.102.45.214.45371 > 8.8.8.8.53: 35686+ A? b.root-servers.net. (36)
03:02:08.415859 IP 103.102.45.214.45371 > 8.8.8.8.53: 64598+ AAAA? b.root-servers.net. (36)
03:02:08.418025 IP 8.8.8.8.53 > 103.102.45.214.45371: 35686 1/0/0 A 199.9.14.201 (52)
03:02:08.418040 IP 8.8.8.8.53 > 103.102.45.214.45371: 64598 1/0/0 AAAA 2001:500:200::b (64)
03:02:08.418387 IP 103.102.45.214.49046 > 8.8.8.8.53: 7782+ A? c.root-servers.net. (36)
03:02:08.418403 IP 103.102.45.214.49046 > 8.8.8.8.53: 21629+ AAAA? c.root-servers.net. (36)
03:02:08.421263 IP 8.8.8.8.53 > 103.102.45.214.49046: 7782 1/0/0 A 192.33.4.12 (52)
03:02:08.421378 IP 8.8.8.8.53 > 103.102.45.214.49046: 21629 1/0/0 AAAA 2001:500:2::c (64)
03:02:08.421762 IP 103.102.45.214.43005 > 8.8.8.8.53: 55459+ A? d.root-servers.net. (36)
03:02:08.421774 IP 103.102.45.214.43005 > 8.8.8.8.53: 63187+ AAAA? d.root-servers.net. (36)
03:02:08.423882 IP 8.8.8.8.53 > 103.102.45.214.43005: 55459 1/0/0 A 199.7.91.13 (52)
03:02:08.423975 IP 8.8.8.8.53 > 103.102.45.214.43005: 63187 1/0/0 AAAA 2001:500:2d::d (64)
03:02:08.424239 IP 103.102.45.214.49361 > 8.8.8.8.53: 28373+ A? e.root-servers.net. (36)
03:02:08.424255 IP 103.102.45.214.49361 > 8.8.8.8.53: 2019+ AAAA? e.root-servers.net. (36)
03:02:08.427143 IP 8.8.8.8.53 > 103.102.45.214.49361: 2019 1/0/0 AAAA 2001:500:a8::e (64)
03:02:08.427157 IP 8.8.8.8.53 > 103.102.45.214.49361: 28373 1/0/0 A 192.203.230.10 (52)
03:02:08.427465 IP 103.102.45.214.45051 > 8.8.8.8.53: 28232+ A? f.root-servers.net. (36)
03:02:08.427478 IP 103.102.45.214.45051 > 8.8.8.8.53: 7095+ AAAA? f.root-servers.net. (36)
03:02:08.430352 IP 8.8.8.8.53 > 103.102.45.214.45051: 7095 1/0/0 AAAA 2001:500:2f::f (64)
03:02:08.430369 IP 8.8.8.8.53 > 103.102.45.214.45051: 28232 1/0/0 A 192.5.5.241 (52)
03:02:08.430687 IP 103.102.45.214.49636 > 8.8.8.8.53: 31201+ A? g.root-servers.net. (36)
03:02:08.430700 IP 103.102.45.214.49636 > 8.8.8.8.53: 37140+ AAAA? g.root-servers.net. (36)
03:02:08.433575 IP 8.8.8.8.53 > 103.102.45.214.49636: 31201 1/0/0 A 192.112.36.4 (52)
03:02:08.433587 IP 8.8.8.8.53 > 103.102.45.214.49636: 37140 1/0/0 AAAA 2001:500:12::d0d (64)
03:02:08.433868 IP 103.102.45.214.47958 > 8.8.8.8.53: 43912+ A? h.root-servers.net. (36)
03:02:08.433881 IP 103.102.45.214.47958 > 8.8.8.8.53: 17413+ AAAA? h.root-servers.net. (36)
03:02:08.436572 IP 8.8.8.8.53 > 103.102.45.214.47958: 43912 1/0/0 A 198.97.190.53 (52)
03:02:08.436667 IP 8.8.8.8.53 > 103.102.45.214.47958: 17413 1/0/0 AAAA 2001:500:1::53 (64)
03:02:08.436883 IP 103.102.45.214.43974 > 8.8.8.8.53: 25088+ A? a.root-servers.net. (36)
03:02:08.436901 IP 103.102.45.214.43974 > 8.8.8.8.53: 13317+ AAAA? a.root-servers.net. (36)
03:02:08.440623 IP 8.8.8.8.53 > 103.102.45.214.43974: 13317 1/0/0 AAAA 2001:503:ba3e::2:30 (64)
03:02:08.440640 IP 8.8.8.8.53 > 103.102.45.214.43974: 25088 1/0/0 A 198.41.0.4 (52)
03:02:08.441037 IP 103.102.45.214.34153 > 8.8.8.8.53: 63417+ A? i.root-servers.net. (36)
03:02:08.441051 IP 103.102.45.214.34153 > 8.8.8.8.53: 17274+ AAAA? i.root-servers.net. (36)
03:02:08.443162 IP 8.8.8.8.53 > 103.102.45.214.34153: 63417 1/0/0 A 192.36.148.17 (52)
03:02:08.443174 IP 8.8.8.8.53 > 103.102.45.214.34153: 17274 1/0/0 AAAA 2001:7fe::53 (64)
03:02:08.443494 IP 103.102.45.214.39078 > 8.8.8.8.53: 44683+ A? j.root-servers.net. (36)
03:02:08.443535 IP 103.102.45.214.39078 > 8.8.8.8.53: 61608+ AAAA? j.root-servers.net. (36)
03:02:08.445608 IP 8.8.8.8.53 > 103.102.45.214.39078: 44683 1/0/0 A 192.58.128.30 (52)
03:02:08.445624 IP 8.8.8.8.53 > 103.102.45.214.39078: 61608 1/0/0 AAAA 2001:503:c27::2:30 (64)
03:02:08.446009 IP 103.102.45.214.47445 > 8.8.8.8.53: 48522+ A? k.root-servers.net. (36)
03:02:08.446025 IP 103.102.45.214.47445 > 8.8.8.8.53: 52090+ AAAA? k.root-servers.net. (36)
03:02:08.449581 IP 8.8.8.8.53 > 103.102.45.214.47445: 48522 1/0/0 A 193.0.14.129 (52)
03:02:08.449602 IP 8.8.8.8.53 > 103.102.45.214.47445: 52090 1/0/0 AAAA 2001:7fd::1 (64)
03:02:08.449993 IP 103.102.45.214.39931 > 8.8.8.8.53: 8107+ A? l.root-servers.net. (36)
03:02:08.450011 IP 103.102.45.214.39931 > 8.8.8.8.53: 34280+ AAAA? l.root-servers.net. (36)
03:02:08.452113 IP 8.8.8.8.53 > 103.102.45.214.39931: 8107 1/0/0 A 199.7.83.42 (52)
03:02:08.452135 IP 8.8.8.8.53 > 103.102.45.214.39931: 34280 1/0/0 AAAA 2001:500:9f::42 (64)
03:02:08.455452 IP 103.102.45.214.37461 > 192.203.230.10.53: 55461 [1au] A? jd.com. (35)
03:02:08.457484 IP 192.203.230.10.53 > 103.102.45.214.37461: 55461- 0/15/27 (1166)
03:02:08.458311 IP 103.102.45.214.59699 > 8.8.8.8.53: 10471+ A? l.gtld-servers.net. (36)
03:02:08.458325 IP 103.102.45.214.59699 > 8.8.8.8.53: 16116+ AAAA? l.gtld-servers.net. (36)
03:02:08.460641 IP 8.8.8.8.53 > 103.102.45.214.59699: 10471 1/0/0 A 192.41.162.30 (52)
03:02:08.460703 IP 8.8.8.8.53 > 103.102.45.214.59699: 16116 1/0/0 AAAA 2001:500:d937::30 (64)
03:02:08.460943 IP 103.102.45.214.35234 > 8.8.8.8.53: 37280+ A? b.gtld-servers.net. (36)
03:02:08.460955 IP 103.102.45.214.35234 > 8.8.8.8.53: 40020+ AAAA? b.gtld-servers.net. (36)
03:02:08.463610 IP 8.8.8.8.53 > 103.102.45.214.35234: 37280 1/0/0 A 192.33.14.30 (52)
03:02:08.463666 IP 8.8.8.8.53 > 103.102.45.214.35234: 40020 1/0/0 AAAA 2001:503:231d::2:30 (64)
03:02:08.463949 IP 103.102.45.214.57378 > 8.8.8.8.53: 42477+ A? c.gtld-servers.net. (36)
03:02:08.463966 IP 103.102.45.214.57378 > 8.8.8.8.53: 35216+ AAAA? c.gtld-servers.net. (36)
03:02:08.466165 IP 8.8.8.8.53 > 103.102.45.214.57378: 35216 1/0/0 AAAA 2001:503:83eb::30 (64)
03:02:08.466270 IP 8.8.8.8.53 > 103.102.45.214.57378: 42477 1/0/0 A 192.26.92.30 (52)
03:02:08.466497 IP 103.102.45.214.41532 > 8.8.8.8.53: 5275+ A? d.gtld-servers.net. (36)
03:02:08.466526 IP 103.102.45.214.41532 > 8.8.8.8.53: 57003+ AAAA? d.gtld-servers.net. (36)
03:02:08.469193 IP 8.8.8.8.53 > 103.102.45.214.41532: 5275 1/0/0 A 192.31.80.30 (52)
03:02:08.469211 IP 8.8.8.8.53 > 103.102.45.214.41532: 57003 1/0/0 AAAA 2001:500:856e::30 (64)
03:02:08.469470 IP 103.102.45.214.54212 > 8.8.8.8.53: 16543+ A? e.gtld-servers.net. (36)
03:02:08.469483 IP 103.102.45.214.54212 > 8.8.8.8.53: 55193+ AAAA? e.gtld-servers.net. (36)
03:02:08.472327 IP 8.8.8.8.53 > 103.102.45.214.54212: 16543 1/0/0 A 192.12.94.30 (52)
03:02:08.472379 IP 8.8.8.8.53 > 103.102.45.214.54212: 55193 1/0/0 AAAA 2001:502:1ca1::30 (64)
03:02:08.472625 IP 103.102.45.214.46070 > 8.8.8.8.53: 43953+ A? f.gtld-servers.net. (36)
03:02:08.472637 IP 103.102.45.214.46070 > 8.8.8.8.53: 13776+ AAAA? f.gtld-servers.net. (36)
03:02:08.475489 IP 8.8.8.8.53 > 103.102.45.214.46070: 13776 1/0/0 AAAA 2001:503:d414::30 (64)
03:02:08.475507 IP 8.8.8.8.53 > 103.102.45.214.46070: 43953 1/0/0 A 192.35.51.30 (52)
03:02:08.475723 IP 103.102.45.214.51014 > 8.8.8.8.53: 62928+ A? g.gtld-servers.net. (36)
03:02:08.475735 IP 103.102.45.214.51014 > 8.8.8.8.53: 29143+ AAAA? g.gtld-servers.net. (36)
03:02:08.477792 IP 8.8.8.8.53 > 103.102.45.214.51014: 29143 1/0/0 AAAA 2001:503:eea3::30 (64)
03:02:08.477809 IP 8.8.8.8.53 > 103.102.45.214.51014: 62928 1/0/0 A 192.42.93.30 (52)
03:02:08.478114 IP 103.102.45.214.49275 > 8.8.8.8.53: 333+ A? a.gtld-servers.net. (36)
03:02:08.478131 IP 103.102.45.214.49275 > 8.8.8.8.53: 31178+ AAAA? a.gtld-servers.net. (36)
03:02:08.481169 IP 8.8.8.8.53 > 103.102.45.214.49275: 31178 1/0/0 AAAA 2001:503:a83e::2:30 (64)
03:02:08.481179 IP 8.8.8.8.53 > 103.102.45.214.49275: 333 1/0/0 A 192.5.6.30 (52)
03:02:08.481413 IP 103.102.45.214.49179 > 8.8.8.8.53: 53128+ A? h.gtld-servers.net. (36)
03:02:08.481425 IP 103.102.45.214.49179 > 8.8.8.8.53: 7245+ AAAA? h.gtld-servers.net. (36)
03:02:08.484195 IP 8.8.8.8.53 > 103.102.45.214.49179: 7245 1/0/0 AAAA 2001:502:8cc::30 (64)
03:02:08.484209 IP 8.8.8.8.53 > 103.102.45.214.49179: 53128 1/0/0 A 192.54.112.30 (52)
03:02:08.484506 IP 103.102.45.214.49975 > 8.8.8.8.53: 56255+ A? i.gtld-servers.net. (36)
03:02:08.484540 IP 103.102.45.214.49975 > 8.8.8.8.53: 27961+ AAAA? i.gtld-servers.net. (36)
03:02:08.486866 IP 8.8.8.8.53 > 103.102.45.214.49975: 27961 1/0/0 AAAA 2001:503:39c1::30 (64)
03:02:08.486877 IP 8.8.8.8.53 > 103.102.45.214.49975: 56255 1/0/0 A 192.43.172.30 (52)
03:02:08.487097 IP 103.102.45.214.36677 > 8.8.8.8.53: 49016+ A? j.gtld-servers.net. (36)
03:02:08.487109 IP 103.102.45.214.36677 > 8.8.8.8.53: 57022+ AAAA? j.gtld-servers.net. (36)
03:02:08.489380 IP 8.8.8.8.53 > 103.102.45.214.36677: 49016 1/0/0 A 192.48.79.30 (52)
03:02:08.489391 IP 8.8.8.8.53 > 103.102.45.214.36677: 57022 1/0/0 AAAA 2001:502:7094::30 (64)
03:02:08.489620 IP 103.102.45.214.49808 > 8.8.8.8.53: 4381+ A? k.gtld-servers.net. (36)
03:02:08.489631 IP 103.102.45.214.49808 > 8.8.8.8.53: 22145+ AAAA? k.gtld-servers.net. (36)
03:02:08.491899 IP 8.8.8.8.53 > 103.102.45.214.49808: 4381 1/0/0 A 192.52.178.30 (52)
03:02:08.491909 IP 8.8.8.8.53 > 103.102.45.214.49808: 22145 1/0/0 AAAA 2001:503:d2d::30 (64)
03:02:08.492110 IP 103.102.45.214.56100 > 8.8.8.8.53: 15798+ A? m.gtld-servers.net. (36)
03:02:08.492122 IP 103.102.45.214.56100 > 8.8.8.8.53: 56048+ AAAA? m.gtld-servers.net. (36)
03:02:08.494840 IP 8.8.8.8.53 > 103.102.45.214.56100: 56048 1/0/0 AAAA 2001:501:b1f9::30 (64)
03:02:08.494851 IP 8.8.8.8.53 > 103.102.45.214.56100: 15798 1/0/0 A 192.55.83.30 (52)
03:02:08.495413 IP 103.102.45.214.37253 > 192.33.14.30.53: 1521 [1au] A? jd.com. (35)
03:02:08.498291 IP 192.33.14.30.53 > 103.102.45.214.37253: 1521- 0/8/5 (664)
03:02:08.498741 IP 103.102.45.214.48540 > 8.8.8.8.53: 13340+ A? ns1.jdcache.com. (33)
03:02:08.498755 IP 103.102.45.214.48540 > 8.8.8.8.53: 59469+ AAAA? ns1.jdcache.com. (33)
03:02:08.616686 IP 8.8.8.8.53 > 103.102.45.214.48540: 59469 0/1/0 (79)
03:02:08.616699 IP 8.8.8.8.53 > 103.102.45.214.48540: 13340 1/0/0 A 111.13.28.10 (49)
03:02:08.617265 IP 103.102.45.214.47676 > 8.8.8.8.53: 5469+ A? ns2.jdcache.com. (33)
03:02:08.617284 IP 103.102.45.214.47676 > 8.8.8.8.53: 36730+ AAAA? ns2.jdcache.com. (33)
03:02:08.682073 IP 8.8.8.8.53 > 103.102.45.214.47676: 5469 1/0/0 A 111.206.226.10 (49)
03:02:08.721863 IP 8.8.8.8.53 > 103.102.45.214.47676: 36730 0/1/0 (83)
03:02:08.722344 IP 103.102.45.214.48381 > 8.8.8.8.53: 8096+ A? ns3.jdcache.com. (33)
03:02:08.722358 IP 103.102.45.214.48381 > 8.8.8.8.53: 58833+ AAAA? ns3.jdcache.com. (33)
03:02:08.785845 IP 8.8.8.8.53 > 103.102.45.214.48381: 8096 1/0/0 A 120.52.149.254 (49)
03:02:08.793548 IP 8.8.8.8.53 > 103.102.45.214.48381: 58833 0/1/0 (83)
03:02:08.793931 IP 103.102.45.214.41451 > 8.8.8.8.53: 61075+ A? ns4.jdcache.com. (33)
03:02:08.793944 IP 103.102.45.214.41451 > 8.8.8.8.53: 46690+ AAAA? ns4.jdcache.com. (33)
03:02:08.857881 IP 8.8.8.8.53 > 103.102.45.214.41451: 46690 0/1/0 (83)
03:02:08.858079 IP 8.8.8.8.53 > 103.102.45.214.41451: 61075 1/0/0 A 106.39.177.32 (49)
03:02:08.858567 IP 103.102.45.214.37760 > 106.39.177.32.53: 56070 [1au] A? jd.com. (35)
03:02:08.984224 IP 106.39.177.32.53 > 103.102.45.214.37760: 56070*- 1/8/9 A 120.52.148.118 (331)
^C
128 packets captured
128 packets received by filter
0 packets dropped by kernel
```


# dig 9.11.5 实验

换了个 `dig` 版本再次尝试：

```
$ dig +trace +additional jd.com

; <<>> DiG 9.11.5-P1-1ubuntu2.5-Ubuntu <<>> +trace +additional jd.com
;; global options: +cmd
.                       747     IN      NS      a.root-servers.net.
.                       747     IN      NS      b.root-servers.net.
.                       747     IN      NS      c.root-servers.net.
.                       747     IN      NS      d.root-servers.net.
.                       747     IN      NS      e.root-servers.net.
.                       747     IN      NS      f.root-servers.net.
.                       747     IN      NS      g.root-servers.net.
.                       747     IN      NS      h.root-servers.net.
.                       747     IN      NS      i.root-servers.net.
.                       747     IN      NS      j.root-servers.net.
.                       747     IN      NS      k.root-servers.net.
.                       747     IN      NS      l.root-servers.net.
.                       747     IN      NS      m.root-servers.net.
;; Received 239 bytes from 127.0.0.1#53(127.0.0.1) in 25 ms

com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
com.                    172800  IN      NS      c.gtld-servers.net.
com.                    172800  IN      NS      d.gtld-servers.net.
com.                    172800  IN      NS      e.gtld-servers.net.
com.                    172800  IN      NS      f.gtld-servers.net.
com.                    172800  IN      NS      g.gtld-servers.net.
com.                    172800  IN      NS      h.gtld-servers.net.
com.                    172800  IN      NS      i.gtld-servers.net.
com.                    172800  IN      NS      j.gtld-servers.net.
com.                    172800  IN      NS      k.gtld-servers.net.
com.                    172800  IN      NS      l.gtld-servers.net.
com.                    172800  IN      NS      m.gtld-servers.net.
com.                    86400   IN      DS      30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.                    86400   IN      RRSIG   DS 8 1 86400 20190814170000 20190801160000 59944 . q7wAN0bPBGlgcN+edSDbKVc2KzcqBXKuI8RECDtpPK2c7zm7HXBS4VLi AVP3g1fDPw0Z3DgtgM6VmMaWmHaUtAtKuSTGyLipfL+PdIXTQQsyDYUb 0IrjmKdFdKJphM6qtQvjrYShsSfIHLFasMDXIe0/x88yDaY8Aw8JsOJW Tc/VAovF7P98GgnqB7uNZFeuKYi/38DzjOYhXUJX3JYRocsv4eb09cay 4n/d7osYvGh+2THk85uptHTzQXHcuQ0CzLWI7wDA/ukj7asdauauHJOp D1qAnYKwVVEGFZ1pPwO8SgBF2KZACfGkNQCivD7VruugmKvkD6KSmEvl GCZHnw==
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
;; Received 1166 bytes from 199.7.91.13#53(d.root-servers.net) in 255 ms

jd.com.                 172800  IN      NS      ns1.jdcache.com.
jd.com.                 172800  IN      NS      ns2.jdcache.com.
jd.com.                 172800  IN      NS      ns3.jdcache.com.
jd.com.                 172800  IN      NS      ns4.jdcache.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN NSEC3 1 1 0 - CK0Q1GIN43N1ARRC9OSM6QPQR81H5M9A NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN RRSIG NSEC3 8 2 86400 20190807044529 20190731033529 17708 com. zH+3/u7U02O4Hu/leXwkHCaL+tMktPL9uFMxc0YzWj8h4Btgaw2cofp/ t/jpQvv9nGJXatSYucoCE0f14Rw4CyigHG2j3LmxBTDIoewTXQ7wMbYo TUFiEh0kXcZMp3+irzTxlPI9+SebEpQkUbYz93eKDbg5nC3+tyE6o/3E Nl4=
VCN61V6E8RJN4R24GKDTFMUBGC610E4P.com. 86400 IN NSEC3 1 1 0 - VCN77CSSQ0F81UJ2JQ85RQR2AF15I71Q NS DS RRSIG
VCN61V6E8RJN4R24GKDTFMUBGC610E4P.com. 86400 IN RRSIG NSEC3 8 2 86400 20190808045137 20190801034137 17708 com. ZeT3GsyO3s7eumN3G5a9mjxKQ2gmDBf7pF0CIzq5ohWfmfBLY198j2d1 Qj+kTkhFnNX4p2bIFDu1YWdKOzquNg/6ce/yHxCjiwAYa0eQo5TwXOaj 2VfDaUR7z8HcNjfWMpPgFXWKw/aE29CoPxf9s/mYpeI7adp7IgKlLWJr zgs=
ns1.jdcache.com.        172800  IN      A       111.13.28.10
ns2.jdcache.com.        172800  IN      A       111.206.226.10
ns3.jdcache.com.        172800  IN      A       120.52.149.254
ns4.jdcache.com.        172800  IN      A       106.39.177.32
;; Received 664 bytes from 192.26.92.30#53(c.gtld-servers.net) in 175 ms

jd.com.                 120     IN      A       120.52.148.118
jd.com.                 120     IN      NS      ns4.jdcache.com.
jd.com.                 120     IN      NS      ns3.jdcache.com.
jd.com.                 120     IN      NS      ns1.jdcache.com.
jd.com.                 120     IN      NS      ns2.jd.com.
jd.com.                 120     IN      NS      ns3.jd.com.
jd.com.                 120     IN      NS      ns2.jdcache.com.
jd.com.                 120     IN      NS      ns1.jd.com.
jd.com.                 120     IN      NS      ns4.jd.com.
ns1.jd.com.             120     IN      A       111.13.28.10
ns1.jdcache.com.        720     IN      A       111.13.28.10
ns2.jd.com.             120     IN      A       111.206.226.10
ns2.jdcache.com.        720     IN      A       111.206.226.10
ns3.jd.com.             120     IN      A       120.52.149.254
ns3.jdcache.com.        720     IN      A       120.52.149.254
ns4.jd.com.             120     IN      A       106.39.177.32
ns4.jdcache.com.        720     IN      A       106.39.177.32
;; Received 331 bytes from 120.52.149.254#53(ns3.jdcache.com) in 42 ms
```

tcpdump 输出：
```
$ sudo tcpdump -nnn "udp and port 53"                         
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
03:02:26.874851 IP 192.168.137.2.19648 > 114.114.114.114.53: 12919 [1au] NS? . (40)
03:02:26.899039 IP 114.114.114.114.53 > 192.168.137.2.19648: 12919 13/0/1 NS a.root-servers.net., NS b.root-servers.net., NS c.root-servers.net., NS d.root-servers.net., NS e.root-servers.net., NS f.root-servers.net., NS g.root-servers.net., NS h.root-servers.net., NS i.root-servers.net., NS j.root-servers.net., NS k.root-servers.net., NS l.root-servers.net., NS m.root-servers.net. (239)
03:02:27.906412 IP 192.168.137.2.59905 > 199.7.91.13.53: 6784 [1au] A? jd.com. (47)
03:02:28.162112 IP 199.7.91.13.53 > 192.168.137.2.59905: 6784- 0/15/27 (1166)
03:02:28.165031 IP 192.168.137.2.53860 > 192.26.92.30.53: 12764 [1au] A? jd.com. (47)
03:02:28.339964 IP 192.26.92.30.53 > 192.168.137.2.53860: 12764- 0/8/5 (664)
03:02:28.342979 IP 192.168.137.2.5045 > 114.114.114.114.53: 34624+ [1au] A? ns1.jdcache.com. (44)
03:02:28.343935 IP 192.168.137.2.20388 > 114.114.114.114.53: 52053+ [1au] AAAA? ns1.jdcache.com. (44)
03:02:28.367670 IP 114.114.114.114.53 > 192.168.137.2.5045: 34624 1/0/1 A 111.13.28.10 (60)
03:02:28.368039 IP 114.114.114.114.53 > 192.168.137.2.20388: 52053 0/1/1 (90)
03:02:28.369163 IP 192.168.137.2.49753 > 114.114.114.114.53: 4153+ [1au] A? ns2.jdcache.com. (44)
03:02:28.369327 IP 192.168.137.2.15385 > 114.114.114.114.53: 22014+ [1au] AAAA? ns2.jdcache.com. (44)
03:02:28.393206 IP 114.114.114.114.53 > 192.168.137.2.15385: 22014 0/1/1 (94)
03:02:28.393231 IP 114.114.114.114.53 > 192.168.137.2.49753: 4153 1/0/1 A 111.206.226.10 (60)
03:02:28.394485 IP 192.168.137.2.29482 > 114.114.114.114.53: 11026+ [1au] AAAA? ns3.jdcache.com. (44)
03:02:28.419199 IP 114.114.114.114.53 > 192.168.137.2.29482: 11026 0/1/1 (94)
03:02:28.420244 IP 192.168.137.2.58878 > 114.114.114.114.53: 35780+ [1au] A? ns4.jdcache.com. (44)
03:02:28.420471 IP 192.168.137.2.41094 > 114.114.114.114.53: 50469+ [1au] AAAA? ns4.jdcache.com. (44)
03:02:28.445206 IP 114.114.114.114.53 > 192.168.137.2.58878: 35780 1/0/1 A 106.39.177.32 (60)
03:02:28.445248 IP 114.114.114.114.53 > 192.168.137.2.41094: 50469 0/1/1 (94)
03:02:28.446601 IP 192.168.137.2.57466 > 120.52.149.254.53: 40214 [1au] A? jd.com. (47)
03:02:28.489201 IP 120.52.149.254.53 > 192.168.137.2.57466: 40214*- 1/8/9 A 120.52.148.118 (331)
^C
22 packets captured
22 packets received by filter
0 packets dropped by kernel
```

这个更新版本的 `dig` 没有再针对：

* 每个根服务器：`*.root-servers.net` 
* 每个 `.com` 域的权威 DNS 服务器：`*.gtld-servers.net` 

重复查询 A/AAAA 记录，说明使用了胶水记录，但是它仍然对：

* 每个域名的权威 DNS 服务器：`ns*.jdcache.com` 

查询 A/AAAA 记录，说明这一级返回的**胶水记录仍然被忽略了**。

# 递归查询是否使用胶水记录

两个版本的 `dig` 都出现了忽略胶水记录的情况，由此老衲开始怀疑自己前面提到的“**按照老衲的理解**，递归进入下一步查询时，可以直接使用胶水记录中的 IP 地址作为 DNS 服务器的地址。”，是否是正确的？递归查询真的会忽略胶水记录吗？

老衲接着在自建测试用的权威 DNS 服务（`ns*.ming.app`）上进行了另一个实验：

1. 将一个未使用过的域名（`jidan.app`）的 DNS 服务器指向到这组服务器（`ns*.ming.app`）
2. 然后分别使用 `dig @8.8.8.8 jidan.app` 和 `dig @114.114.114.114 jidan.app` 查询 A 记录
3. 在 `ns*.ming.app` 服务器上使用 `tcpdump` 跟踪是否有 NS 查询

结果发现 `tcpdump` 中均未出现 NS 查询：

**8.8.8.8：**
```
# tcpdump -nnn "udp and port 53"
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
03:22:08.428827 IP 173.194.171.5.37225 > 103.102.--.--.53: 719% [1au] A? jidan.app. (49)
03:22:08.429394 IP 103.102.--.--.53 > 173.194.171.5.37225: 719*- 1/0/0 A 127.0.0.1 (52)
^C
2 packets captured
2 packets received by filter
0 packets dropped by kernel
```

**114.114.114.114：**
```
# tcpdump -nnn "udp and port 53"
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
03:19:01.689191 IP 60.215.138.161.45243 > 103.102.--.--.53: 48181% [1au] A? jidan.app. (38)
03:19:01.689391 IP 103.102.--.--.53 > 60.215.138.161.45243: 48181*- 1/0/0 A 127.0.0.1 (52)
^C
2 packets captured
2 packets received by filter
0 packets dropped by kernel
```

权威服务器没有收到 NS 查询，说明 `8.8.8.8` 和 `114.114.114.114` 这两个递归服务确实使用了胶水记录，其它递归服务器应该也遵循同样的规范吧？（再次期待熟读 RFC 的施主贡献下相关规范资料所在章节）

# 结论

`dig +trace` 确实不等同于递归查询，根据不同 `dig` 的版本，它可能会忽略全部或部分胶水记录。