---
title: PVE中OpenWrt的安装
date: 2025-6-20 11:00:00 +0800
categories: [PVE,OpenWrt]
tags: [OpenWrt,PVE]
author: FFraankk
---

# OPENWRT安装

## 虚拟机创建
作为网关的openwrt此时还没有存在。点击创建虚拟机，输入虚拟机名称OpenWRT，点击下一步，由于Openwrt镜像无法直接用在PVE中，所以暂时选择不使用任何介质。在磁盘选择中给OpenWRT分配50G空间，用于后续折腾。给OpenWRT分配一个核心就够用了，使用host类型可以增强性能，内存1G。在网络上，OpenWRT要先lan,再wan，所以桥接vmbr1，完成创建。

## 固件获取
ImmortalWrt是openwrt的分支，我们使用这个作为软路由，可以自己编译固件。在https://firmware-selector.immortalwrt.org/找到合适的固件，这里选择Generic x86/64 23.5.4，（备注：如果你是编译luci 24的版本，上述代码中 要将luci-i18n-opkg-zh-cn替换为luci-i18n-package-manager-zh-cn 因为新版本里它的名称发生变化。）点击自定义预安装软件包和/或首次启动脚本，修改初始密码，修改ip为192.168.58.1，即网关地址。记得删除前面的#。点击请求构建，在线编译。

## 镜像导入
在pve管理页面中，选择虚拟机100（openwrt），在硬件中单击硬盘，点击上方功能中的分离，随后删除CD/DVD驱动器和未使用磁盘0。将构建好的.iso.gz文件解压，变成后缀为.iso的文件通过local（pve）-iso image上传至pve中。在上传完成后的Task review中复制镜像的存放地址。在pve的shell中执行，qm importdisk 100 <镜像地址> local-lvm，转化iso文件。在虚拟机中单机未使用的磁盘0，点击编辑，设置总线-设备为SATA，点击添加。在选项中修改引导顺序，勾选刚刚添加的SATA0并且拖动到第一位。添加其他所需要的设备，开机。

至此，OpenWRT初始配置完成。

## 访问问题
但在这里，我们正在使用wan口管理并且访问PVE的管理界面，但是openwrt无法通过wan口访问，因此，我们需要使用之前虚拟出来的vmbr1来进行访问，如何实现呢？那就需要另外开一个虚拟机了，这里我们选择，ubuntu 22.04

参考：https://optimus-xs.github.io/posts/install-openwrt-in-pve/
