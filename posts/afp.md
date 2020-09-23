---
title: "自建TimeCapsule给Mac备份"
date: 2020-07-07T00:00:00+08:00
author: "infsr"
Description: ""
tags: ["OpenWrt", "TimeCapsule", "Mac"]
Categories: []

---


  最近重装了Mac系统,之前的数据啥也被抢救回来,更深刻的体会到了备份的好处,正好现在家里用着openwrt,在openwrt上实现Time Machine功能并不难,有开源的软件包AFP Netatalk 可以使用,开始整...

<!--more-->

  oopenwrt官方固件没有自带需要自行安装, 我这里自己编译了openwrt添加了Netatalk 官方安装需要通过opkg安装netatalk 和avahi-daemon 这两个程序



需要修改的配置

/etc/afp.conf

```shell
;
; Netatalk 3.x configuration file
;

[Backups]
     path = /mnt/sdb1/Backups  # 挂载点
     time machine = yes
     vol size limit = 250000   # 设置磁盘大小
     valid users = @users
```

/etc/avahi/avahi-daemon.conf

```conf
[server]
host-name=Time Capsule   # 修改显示名称
#domain-name=local
use-ipv4=yes
use-ipv6=yes
check-response-ttl=no
use-iff-running=no

[publish]
publish-addresses=yes
publish-hinfo=yes
publish-workstation=no
publish-domain=yes
#publish-dns-servers=192.168.1.1
#publish-resolv-conf-dns-servers=yes

...
```

因为netatalk没有包含zeroconf支持,所以需要手动声明需要的afpovertcp device-info 和 adisk text-record属性 创建一个服务文件:

/etc/avahi/services/afp.service

```xml
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
 <name replace-wildcards="yes">%h</name>
  <service>
   <type>_afpovertcp._tcp</type>
   <port>548</port>
  </service>
  <service>
   <type>_device-info._tcp</type>
   <port>0</port>
   <txt-record>model=TimeCapsule</txt-record>
  </service>
  <service>
   <type>_adisk._tcp</type>
   <port>9</port>
   <txt-record>sys=waMa=0,adVF=0x100,adVU=00000000-AAAA-BBBB-CCCC-111111111111</txt-record>
   <txt-record>dk0=adVN=Backups,adVF=0x81</txt-record>
  </service>
</service-group>
```

model=TimeCapsule 决定了mac finder中显示的硬件图标

adVU=00000000-AAAA-BBBB-CCCC-111111111111修改为新生成的UUID

用 cat /proc/sys/kernel/random/uuid 来创建UUID

adVN=Backups必须和 afp.conf timemachine = YES 的虚拟宗卷名称相匹配



创建用户管理 加入到users用户组

mkdir /home

```shell
mkdir /home
useradd --create-home --groups users --user-group username
passwd username
cd /mnt/sdb1
mkdir Backups
chmod 755 Backups/
chgrp users Backups/
```

大功告成啦

最后就是在mac上设置Time Machine的过程了 非常简单 

自动备份省心了.. 上成果图

![](https://raw.githubusercontent.com/1nfsr/content/image-hosting/20200707190042.png)