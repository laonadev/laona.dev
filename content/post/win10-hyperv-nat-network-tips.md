---
title: "Win10下Hyper-V开发虚拟机NAT网络环境配置要点"
date: 2019-07-30T15:11:14+08:00
lastmod: 2019-07-30T15:11:14+08:00
draft: false
keywords: ["win10", "hyper-v", "nat", "ics", "网卡共享"]
description: "老衲解决了Hyper-V环境中几个NAT网络方面的坑，终于使Hyper-V中的虚拟机成功联网。"
tags: ["hyper-v", "开发环境"]
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

如何安装Hyper-V和创建虚拟机，网络上教程很多，老衲这里不再重复念了。

老衲的任务是在笔记本（Win10工作站版本）上安装Hyper-V + Ubuntu Server，帮助队员们随时随地灵「遭」活「受」办「剥」公「削」。

在这过程中，网络设置方面主要有4个坑需要填：

1. NAT固定网关地址
2. 防火墙开放NAT网段的通行
3. 共享外网网卡给NAT虚拟路由器
4. 保证共享网络在重启后继续有效


且听老衲逐个分解。

<!--more-->

# 一、 NAT固定网关地址

因为笔记本主要使用无线网卡上网，IP地址/IP网段无法固定，因此必须将虚拟机放在NAT网络环境中，固定使用一个子网，宿主机通过NAT网关与虚拟机通信。

老衲创建的虚拟路由器如下：

![](win10-hyperv-nat-network-tips0.png)

修改对应的适配器选项如下：

![](win10-hyperv-nat-network-tips1.png)

这里将`192.168.137.1`作为NAT网关IP地址，所有虚拟机之后都要通过这个NAT网关连接外网。

并且，无论在什么网络环境中，宿主机（Win10这台）都能直接连接虚拟机。

# 二、防火墙开放NAT网段的通行

在 Windows 防火墙高级管理中，在入站规则和出站规则中，分别新建自定义规则：

![](win10-hyperv-nat-network-tips2.png)

规则类型选择“自定义”，程序选“所有程序”，协议和端口、作用域，均不设置，作用域中设置将规则作用于以下IP地址，添加NAT网络的网段，此例中是`192.168.137.0/24`：

![](win10-hyperv-nat-network-tips3.png)

其余规则选项均使用默认值通行。

同样地，再新建一条出站规则，允许`192.168.137.0/24`这个网段通行。

设置这些通行规则后，从虚拟机发送到外网的数据包在经过宿主机的Windows防火墙时，才会被放行，否则将被拦截。

# 三、共享外网网卡给NAT虚拟路由器

前面提到，所有虚拟机都要通过NAT网关连接到外网，那么必须将外网网卡的网络共享给这个NAT网关网卡才行。

还是在更改适配器设置窗口中操作：

![](win10-hyperv-nat-network-tips4.png)

设置完后成，虚拟机就可以正常连接外网了（前提是你的虚拟机的网络设置正确地配置，怎么配置这里老衲不做讲述，每个操作系统都不一样）。

# 四、保证共享网络在重启后继续有效

老衲原以为第三步配置完就结束了，结果第二天开机发现虚拟机又不能上外网了，检查了各项配置，均正常，外网网卡处的共享设置也存在。

经过几番折腾，最终找到原因：

https://answers.microsoft.com/en-us/windows/forum/windows_10-networking/ics-internet-connection-sharing-dosent-work-in/a203c90f-1214-4e5e-ae90-9832ae5ceb55

这是Win10已知的BUG，存在好几年了仍然没有修复。

## 制作网卡共享脚本

于是老衲就想在开机时使用脚本重新配置一次网卡共享，果然有网友和我遇见一样的问题：

https://ntzyz.io/post/fix-windows-10-ics-issue

像老衲这样在Windows使用方面的经验一直没怎么长进的，进入Win10时代更是落后一大截，更别提写PowerShell脚本了，看到上面文章中连PowerShell脚本都写好了，顿时大喜。

```powershell
$NetShare = New-Object -ComObject HNetCfg.HNetShare
$wlan = $null
$ethernet = $null

foreach ($int in $NetShare.EnumEveryConnection) {
  $props = $NetShare.NetConnectionProps.Invoke($int)
  if ($props.Name -eq "WLAN") {
    $wlan = $int;
  }
  if ($props.Name -eq "vEthernet (sunwind-dev)") {
    $ethernet = $int;
  }
}

$wlanConfig = $NetShare.INetSharingConfigurationForINetConnection.Invoke($wlan);
$ethernetConfig = $NetShare.INetSharingConfigurationForINetConnection.Invoke($ethernet);

$wlanConfig.DisableSharing();
$ethernetConfig.DisableSharing();

$wlanConfig.EnableSharing(0);
$ethernetConfig.EnableSharing(1);
```

施主们可以照抄，但记得把脚本中的两张网卡更换成自己的网卡名称。

老衲改好后，把它保存在了`D:\software\fix_win10_ics.ps1`，并写了一个批处理文件`D:\software\fix_win10_ics.bat`，用于配置开机时自动启动执行：

```bat
powershell D:\software\fix_win10_ics.ps1
```

## 测试脚本

尝试在PowerShell运行一次，出错错误：
```cmd
D:\software\fix_win10_ics.ps1 : 无法加载文件 D:\software\fix_win10_ics.ps1，因为在此系统上禁止运行脚本。有关详细信息，
请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
所在位置 行:1 字符: 1
+ D:\software\fix_win10_ics.ps1
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) []，PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```
必须允许运行本地未签名的脚本，执行以下命令设置只有远程下载的脚本才需要签名：
```cmd
Set-ExecutionPolicy RemoteSigned
```

设置后，就可以成功执行这个PowerShell脚本了：

![](win10-hyperv-nat-network-tips7.png)

## 设置开机自动启动

原以为把脚本创建一个快捷方式复制到开机启动目录中就可以了：

![](win10-hyperv-nat-network-tips5.png)

![](win10-hyperv-nat-network-tips6.png)

后来发现这个方案不可行，仍然有个问题需要解决：必须使用管理员权限运行这个脚本。

最后通过配置计划任务解决了这个问题：

![](win10-hyperv-nat-network-tips8.png)

注意，因为笔记本电脑很经常使用电池供电，此处注意有坑：

![](win10-hyperv-nat-network-tips9.png)

默认是勾选的，要去除。

这样才算大功告成！