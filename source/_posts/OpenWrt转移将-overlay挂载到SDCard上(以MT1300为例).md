---
title: OpenWrt转移将/overlay挂载到SDCard上(以MT1300为例)
date: 2023-09-28 23:13:53
tags:
- Openwrt
categories:
- Openwrt
keyword:
- Openwrt
- SDCard
cover: https://blog-jeafly.oss-cn-nanjing.aliyuncs.com/pictures/OpenWrt/mt1300_hardware.png
---

### 1. 安装相关工具(汉化web界面)
```shell
opkg update
opkg install block-mount kmod-fs-ext4 e2fsprogs fdisk
opkg install luci-i18n-base-zh-cn
```

### 2.修改fstab配置文件，更改现有文件系统的挂载点
```shell
DEVICE="$(sed -n -e "/\s\/overlay\s.*$/s///p" /etc/mtab)"
uci -q delete fstab.rwm
uci set fstab.rwm="mount"
uci set fstab.rwm.device="${DEVICE}"
uci set fstab.rwm.target="/rwm"
uci commit fstab
```

### 3.查看U盘的相关信息
```shell
block info
```
如果磁盘格式不是ext4,请执行下面的命令，否则跳过（注意，以下操作会格式化磁盘，请提前备份磁盘文件），注意命令中的/dev/sda1是使用block命令查看到的实际节点，请根据自己的实际情况修改
```shell
DEVICE="/dev/mmcblk0p1"
umount /dev/mmcblk0p1
mkfs.ext4 ${DEVICE}
```
### 4.在配置文件中设置挂载点，注意命令中的/dev/sda1是使用block命令查看到的实际节点，请根据自己的实际情况修改
```shell
DEVICE="/dev/mmcblk0p1"
eval $(block info ${DEVICE} | grep -o -e "UUID=\S*")
uci -q delete fstab.overlay
uci set fstab.overlay="mount"
uci set fstab.overlay.uuid="${UUID}"
uci set fstab.overlay.target="/overlay"
uci commit fstab
```
### 5.将文件系统中现有的内容拷贝到U盘中
```shell
mkdir -p /tmp/cproot
mount --bind /overlay /tmp/cproot
mount ${DEVICE} /mnt
tar -C /tmp/cproot -cvf - . | tar -C /mnt -xf -
umount /tmp/cproot /mnt
```
### 6.重启路由器
```shell
reboot
```
{% mermaid %}
pie
    title Key elements in Product X
    "Calcium" : 42.96
    "Potassium" : 50.05
    "Magnesium" : 10.01
    "Iron" :  5
{% endmermaid %}

{% note info no-icon %}
完成后占用高问题(未测试)

安装openclash后，占用很高的问题，捎带理解一下linux占用如何看

http://www.ruanyifeng.com/blog/2011/07/linux_load_average_explained.html

开启（关闭） flow offload
```shell
iptables -I FORWARD 1 -m conntrack --ctstate RELATED,ESTABLISHED -j FLOWOFFLOAD
iptables -D FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j FLOWOFFLOAD
```
实测mtk7621a 日常使用， 5分钟占用从0.6起步，降低到0.2
{% endnote %}