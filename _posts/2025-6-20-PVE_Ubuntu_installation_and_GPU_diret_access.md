---
title: PVE中Ubuntu的安装与显卡直通
date: 2025-6-20 11:00:00 +0800
categories: [PVE,Ubuntu]
tags: [Ubuntu,PVE]
author: FFraankk
---

# Ubuntu的安装与显卡直通
在pve管理界面，通过local（pve）-iso image上传ubuntu镜像文件至pve中。点击创建虚拟机创建，选择刚刚上传的镜像文件，由于我们需要用到显卡直通，因此机型选择q35，而非默认。选择合适的内存大小和硬盘大小，核心类别选择host，完成创建并开启虚拟机。在VNC中完成Ubuntu系统的安装，重启，开机。

下面我们去PVE中开启显卡直通功能。

# 显卡直通配置

## 第一步：确认支持
确认自己的主板CPU是否支持Vt-d功能
不支持就搞不了直通。intel要b75以上芯片组才支持。也就是说intel4代酷睿处理器以上，都支持。amd不明。

VT-D是io虚拟化。不是VT-X，具体请参考下面文章

https://zhuanlan.zhihu.com/p/50640466
有很多新手，以为主板开启虚拟化功能，就能直通了，其实不是！要开启vt-d才能io虚拟化。AMD平台是iommu，某些OEM主板上叫SRIOV。请注意。

## 第二步：开启iommu
```
#编辑grub，请不要盲目改。根据自己的环境，选择设置
vi /etc/default/grub
#在里面找到：GRUB_CMDLINE_LINUX_DEFAULT="quiet"
#然后修改为：GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
#如果是amd cpu请改为：GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"#如果是需要显卡直通，建议在cmdline再加一句video=vesafb:off video=efifb:off video=simplefb:off，加了之后，pve重启进内核后停留在一个画面，这是正常情况GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt video=vesafb:off video=efifb:off video=simplefb:off"
```

修改完成之后，直接更新grub 
```
update-grub
```
注意，如果此方法还不能开启iommu，请修改 
```
 /etc/kernel/cmdline文件
```
并且使用
```
proxmox-boot-tool refresh 
```
更新启动项

## 第三步 加载相应的内核模块
```
echo vfio >> /etc/modules
echo vfio_iommu_type1 >> /etc/modules
echo vfio_pci >> /etc/modules
echo vfio_virqfd >> /etc/modules
```
使用

```
update-initramfs -k all -u
```

命令更新内核参数

重启主机

## 第四步 验证是否开启iommu
重启之后，在终端输入
```
dmesg | grep iommu
```
出现如下例子。则代表成功
```
[ 1.341100] pci 0000:00:00.0: Adding to iommu group 0
[ 1.341116] pci 0000:00:01.0: Adding to iommu group 1
[ 1.341126] pci 0000:00:02.0: Adding to iommu group 2
[ 1.341137] pci 0000:00:14.0: Adding to iommu group 3
[ 1.341146] pci 0000:00:17.0: Adding to iommu group 4
```
此时输入命令
```
find /sys/kernel/iommu_groups/ -type l 
```
#出现很多直通组，就代表成功了。如果没有任何东西，就是没有开启

# 显卡直通
理论上AMD RADEON 5xxx, 6xxx, 7xxx, Navi 5XXX(XT), NVIDIA GEFORCE 7, 8, GTX 4xx, 5xx, 6xx, 7xx, 9xx, 10xx and RTX 16xx/20xx/30xx都可以成功直通。

但是对于NVIDIA显卡，建议使用9代以上中端卡直通，且使用最新的驱动。

## 1、直接屏蔽显卡驱动
```
#直通AMD显卡，请使用下面命令
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist amdgpu" >> /etc/modprobe.d/blacklist.conf 
#直通NVIDIA显卡，请使用下面命令
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nvidiafb" >> /etc/modprobe.d/blacklist.conf 
#直通INTEL核显，请使用下面命令，注意！如果使用Gvt-G，请不要使用下面的命令
echo "blacklist snd_hda_intel" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist snd_hda_codec_hdmi" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist i915" >> /etc/modprobe.d/blacklist.conf 
```

## 2、把显卡绑定到vfio-pci
使用
```
lspci 
```
查看自己的显卡PCI地址，如02:00

使用
```
lspci -n 
```
查看显卡的did和vid。（xxxx:xxxx,eg:10de:1381）

 02:00.0 02:00.1一个是GPU，一个是声卡，两者都要一起直通，所以通过命令，把2者都绑定到vfio-pci上。
```
echo "options vfio-pci ids=10de:1381,10de:0fbc" > /etc/modprobe.d/vfio.conf
```
#注意，上面这条命令，ids=后面跟直通组的所有设备。中间以英文逗号隔开。自己的设备自己替换。

上述操作完成之后，再检查一下，是否将例子内容替换成自己的。使用以下命令查看。
```
cat /etc/modprobe.d/blacklist.conf
cat /etc/modprobe.d/vfio.conf
```

## 3、更新内核
对于nvidia显卡，需要
```
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
update-initramfs -k all -u 
```

随后重启

# Ubuntu显卡直通
在ubuntu系统的硬件中，选择添加-pci设备-显卡，勾选所有功能ROM-Bar，PCI-Express，不要勾选主GPU。

重启ubuntu。

此时显卡仍然无法完成直通，这是因为ubuntu没有显卡的驱动。下面安装英伟达驱动。

由于我们在ubuntu中进行下载、更新等操作，但是openwrt仍然无法上网，所以我们先外接一个USB网卡，将其直通进ubuntu，给ubuntu提供网络。

首先在浏览器英伟达官网下载linux驱动，后缀为.run。

之后安装所需要的依赖：
```
sudo apt update
sudo apt install build-essential curl wget gcc-12 g++ make 
```

使用
```
gcc-v
```
检查gcc的版本，发现是11，并不是我们安装的12版本，因此：
```
sudo update-alternatives --config gcc
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 2
```

再次使用
```
gcc-v
```
检查gcc的版本，发现是12，更新成功。

赋予我们下载的驱动执行权限
```
chmod +x <NVIDIA>
```

禁用nouveau(nouveau是通用的驱动程序)
```
sudo vim /etc/modprobe.d/blacklist.conf
```

添加如下内容
```
blacklist vga16fb
blacklist nouveau
blacklist rivafb
blacklist rivatv
blacklist nvidiafb
#options nouveau modeset=0
```

执行
```
sudo update-initramfs -u
```

关闭GUI界面避免影响x11
```
sudo systemctl set-default multi-user.target
```

重启
重启后应该出现命令行画面，输入用户名和密码进入系统，使用
```
sudo lsmod | grep nouveau
```
检查nouveau是否成功被加入黑名单，如果没有输出则证明禁用成功。

完成前置条件，开始安装驱动，
```
sudo ./<NVIDIA>
```

Would you like to run the nvidia-xconfig utility to automatically update your X configuration fileso that the NVlDlA X driver dill be used dhen you restart X? Any pre-existing X configuration filewill be backed up.选择"Yes”

其余根据需求选择，只记得第一个对话框要选择左边的。

驱动安装完成，输入
```
sudo systemctl set-default graphical.target
```
恢复GUI
重启电脑，Ubuntu画面由显卡输出，直通完成。可只用
```
nvida-smi
```
查看驱动版本。

参考链接：
https://zhuanlan.zhihu.com/p/124292857
https://www.xiaocaicai.com/2024/04/pve开启硬件直通功能/
