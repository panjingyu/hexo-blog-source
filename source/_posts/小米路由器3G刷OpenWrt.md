---
title: 小米路由器3G刷OpenWrt
description: 记录ZJU校园网环境下对小米路由器3G的刷机过程和基本配置。
date: 2020-02-09 21:52:53
tags:
- OpenWrt
- 网络
---


# breed刷Mi开发版（已开启ssh）

> 删除normal_firmware_md5 6adeca490b4de682e8d944e380b8fa03，否则可能导致重启后无法进入小米开发版的问题。
> 要刷固件+编程器固件（full），我还刷了EEPROM（不过这个应该是包含在编程器固件里了？具体原理不明）

首先在小米路由器web页面上完成初始化设置（随意写）。然后ssh进入，root密码见[此处](https://d.miwifi.com/rom/ssh)。

# Mi开发版刷OpenWrt

Windows下用pscp上传OpenWrt固件到`/tmp`，注意端口号（默认端口号可能不为22，看是否改动了putty配置的ssh默认端口号）。注意需要加`-scp`选项，因为路由器不支持pscp默认的`-sftp`协议。

进入`/tmp`，刷入OpenWrt固件：
```bash
mtd write openwrt-ramips-mt7621-mir3g-squashfs-kernel1.bin kernel0
mtd write openwrt-ramips-mt7621-mir3g-squashfs-kernel1.bin kernel1 # 可选
mtd write openwrt-ramips-mt7621-mir3g-squashfs-rootfs0.bin rootfs0
reboot
```

可以在breed中设置环境变量`xiaomi.r3g.bootfw`为`2`。这一项用来选择启动的内核。如果在上一步把`kernel0`和`kernel1`都刷成OpenWrt了，就无所谓。

有线连入路由器，浏览器打开`openwrt.lan`站点后，在`system->backup/flash firmware`菜单中，将[下载](https://downloads.openwrt.org/releases/18.06.2/targets/ramips/mt7621/openwrt-18.06.2-ramips-mt7621-mir3g-squashfs-sysupgrade.tar)好并校验后的小米路由器3G专用的系统升级tar文件上传，升级固件。

升级完重启后有线连入路由器，ssh连`192.168.1.1`，马上修改`passwd`。

~~为了联网，可用手机热点：关闭所有天线，选择对应热点的天线（如热点是2.4G则选择支持2.4G的天线），接入热点后可成功`opkg update`。~~

如果只是为了opkg update，更推荐的方式是使用可直连的镜像网站；如`https://mirrors.zju.edu.cn/openwrt`。更换opkg源的方法是：
- 在`/etc/opkg`路径下，备份`distfeeds.conf`
- 将`distfeeds.conf`中`downloads.openwrt.org`部分替换为镜像站点，如`mirrors.zju.edu.cn/openwrt`

# 关于L2TP

```bash
opkg install xl2tpd
```

添加interface，设置L2TP协议（注意MAC地址是否与申请时一致，LAN与WAN具有不同MAC地址！）

防火墙`LAN->WAN`勾选`MSS`（ZJU校园网环境下，据说不勾会影响到一些应用，原理未知。）

# 关于DNS

openwrt默认开启了`Rebind protection`选项，导致路由器拒绝返回A记录为私有地址IP的DNS response。关闭该选项即可正常浏览校内网页。

# 关于ipv6

通过`odhcpd`实现IPv6中继：

登录网页管理界面，在`Network->Interfaces`页面底下有`Global network options->IPv6 ULA-Prefix`，这里有一个随机的`fd`开头的`/64`IPv6地址段（如`fddd:ddd1:9439::/48 fd93:8590:feaa::/48`），清空该地址并保存。

修改`/etc/config/dhcp`文件，添加如下部分，使用无状态地址自动配置（SLAAC）IPv6，不使用DHCPv6。
```
config dhcp 'lan'
    option dhcpv6 'disabled'
    option ra 'relay'
    option ndp 'relay'

config dhcp 'wan6'
    option interface 'wan'
    option dhcpv6 'disabled'
    option ra 'relay'
    option ndp 'relay'
    option master '1'
```

重启 odhcpd 服务：
```bash
/etc/init.d/odhcpd restart
```

在`/etc/rc.local`中添加本地启动脚本：
```bash
sleep 30
/etc/init.d/odhcpd restart
```

# 关于静态路由

首先可以把`10.0.0.0\8`对应的学校内网地址全都绕开L2TP，这样就已经解决90%以上情况下的问题了。

但是部分学校内网地址是其他网段的，需要另外配置。如何取得学校的网段信息是个问题，也许可以看一下EasyConnect等官方RVPN客户端，看看里面的路由表。

另外也遇到有人走VPN挂PT，这时候即使我配置了静态路由通过内网访问它，依然会被它的VPN限速。这里静态路由只能保证我避开自己的VPN带宽占用。

~~98上也有人贴了许多推荐配置的路由表，但是数据比较久远了（不过大部分是能用的）。本着宁缺毋滥的原则，我打算只添加经过自己验证的静态路由项；否则可能遇到一些网站上不去的问题（部分杭州的校外网站与学校网站的网段大致相同，路由需要更精细的划分）。~~

[下载学校官方VPN客户端](http://zuits.zju.edu.cn/_upload/article/files/a2/7d/5e8a5ec14fee86f74d2637207452/8ef1fc58-cf10-4140-b03b-51edaa7a20d5.zip)后，可以在里面找到静态路由表。按照其配置即可。