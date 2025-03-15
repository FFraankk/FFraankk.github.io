---
title: Windows11 安装 Ubuntu WSL
date: 2025-3-15 22:00:00 +0800
categories: [Windows,Ubuntu]
tags: [Ubuntu,Windows]
author: FFraankk
---

# 安装WSL

配置条件：windows11 

## 前言

本人主最近想要学习深度学习和强化学习的相关知识，深度学习的训练部分需要用到显卡的算算力。然而在本人的诸多设备中，只有主力机是有英伟达独立显卡的。这台机子储存着众多重要资料，本人并不敢轻易给破坏这台电脑的系统，所以排除了双系统的选择。同时，使用虚拟机会浪费性能，本人并不希望看到这样的事情发生。所以思来想去也就只有WSL能满足这样的条件了。从微软官方网站了解到，开发人员可以在 Windows 计算机上同时访问 Windows 和 Linux 的强大功能。 通过适用于 Linux 的 Windows 子系统 (WSL)，开发人员可以安装 Linux 发行版（例如 Ubuntu、OpenSUSE、Kali、Debian、Arch Linux 等），并直接在 Windows 上使用 Linux 应用程序、实用程序和 Bash 命令行工具，不用进行任何修改，也无需承担传统虚拟机或双启动设置的费用。【1】下面我们一起跟着windows的官方教程进行wsl的安装。

## 安装Powershell

Power shell是Windows环境所开发的壳程式(shell)及脚本语言技术。可以管理 Windows 服务器（特别是域domain），现在的开源 PowerShell 也可以管理 Linux 和 Mac.【2】我们用他来进行命令行的操作。
在Microsoft Store中搜索Powershell中搜索Poweshell可以直接安装，这里不做过多赘述。建议把Powershell固定在任务栏中，我们以后还会经常用到它。

## 开启虚拟化
根据微软官方描述：最新版本的 WSL 使用 Hyper-V 体系结构来实现其虚拟化。所以我们要去windows下开启虚拟化。
1. 选择“开始” ，输入“Windows 功能”，然后从结果列表中选择“打开或关闭 Windows 功能”
2. 在刚刚打开的 “Windows 功能 ”窗口中，找到“虚拟机平台”并选择它
3. 顺带把“适用于Linux的Windows子系统”勾选上
4. 选择“确定”。 可能需要重启电脑

## 安装Linux发行版

我们先来看看WLS都能安装什么发行版。
以管理员身份运行使用`wsl --list --online`命令得到下面的显示:
```
PS C:\Users\GCX> wsl --list --online
以下是可安装的有效分发的列表。
使用 'wsl.exe --install <Distro>' 安装。

NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Debian                          Debian GNU/Linux
kali-linux                      Kali Linux Rolling
Ubuntu-18.04                    Ubuntu 18.04 LTS
Ubuntu-20.04                    Ubuntu 20.04 LTS
Ubuntu-22.04                    Ubuntu 22.04 LTS
Ubuntu-24.04                    Ubuntu 24.04 LTS
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_7                 Oracle Linux 8.7
OracleLinux_9_1                 Oracle Linux 9.1
openSUSE-Leap-15.6              openSUSE Leap 15.6
SUSE-Linux-Enterprise-15-SP5    SUSE Linux Enterprise 15 SP5
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
openSUSE-Tumbleweed             openSUSE Tumbleweed
```
可以看到当前可以安装的版本都在这里了
我们要安装Ubuntu-22.04版本，所以输入`wsl --install Ubuntu-22.04`
稍等片刻，等待Ubuntu安装并启动
按照提示设置new UNIX username 和 password。请注意，输入密码时并不会出现
这时就算安装完成了



## 迁移wsl
注意到wsl默认安装在了C盘，C盘空间紧张，要尽快将系统移出，避免C盘爆红。
1. 在Powershell中输入`wsl --shutdown`,关闭所有正在运行的wsl
2. 输入`wsl -l -v`查看wsl虚拟机状态和名称
3. 在确保关闭以后，在目标目录新建文件夹，使用`wsl --export Ubuntu-22.04 <\path\ubuntu.tar>`
4. 在目标位置出现后，注销原来的`wsl --unregister Ubuntu-22.04`
5. 将刚刚导出的系统重新导入`wsl --import Ubuntu-22.04 \path \path\Ubuntu.tar`
6.重新启动ubuntu

假如默认进入的是root界面，我们需要设置默认用户。
1. 打开具有管理员权限的Powershell。
2. 关闭WSL子系统 `wsl --shutdown`
3. 设置默认用户 `Ubuntu2204 config --default-user <Username>`,其中<Username>换成自己之前的用户名。

## 显卡相关
在windows中将显卡驱动审计到最新以后，在ubuntu中输入`nvidia-smi`查看信息。如出现类似文字，则说明安装成功，记住这里显示的CUDA Version 后面会用
```
Sat Mar 15 22:09:06 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.40.07              Driver Version: 551.52         CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3070 ...    On  |   00000000:01:00.0  On |                  N/A |
| N/A   43C    P8             19W /  150W |     969MiB /   8192MiB |      3%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A        24      G   /Xwayland                                   N/A      |
```
下面配置CUDA，在英伟达的官网下载[CUDA](https://developer.nvidia.com/cuda-toolkit-archive)，选择你的CUDA版本，选择系统：linux，选择架构：x86_64,选择发行版：WSL-Ubuntu，选择版本：2.0，选择安装方式，不同的情况选用不同的安装方式，本人wsl可以直接访问所需要的网站，就选择deb（network）,这个最方便，复制命令，粘贴到wsl中等待下载。

下载完成后，添加环境变量，使用vim编译器修改.bashrc文件 `sudo vim ~/.bashrc`
出现文件后按住下键，移动到页面最后，按一下`i`进入编辑模式，在新的一行添加上
```
export PATH=/etc/alternatives/cuda-12/bin:$PATH
export LD_LIBRARY_PATH=/etc/alternatives/cuda-12/lib64:$LD_LIBRARY_PATH
```
键入上述字符后按下`esc`退出编辑模式，输入`:wq`退出vim编译器。

`source ~/.bashrc`重新加载一下环境变量
执行`nvcc -V` 能看到版本信息说明安装成功
```
frank@LAPTOP-OEVMF5QQ:~$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2024 NVIDIA Corporation
Built on Thu_Mar_28_02:18:24_PDT_2024
Cuda compilation tools, release 12.4, V12.4.131
Build cuda_12.4.r12.4/compiler.34097967_0
```

## WSL资源限制
为了避免wsl占用过多资源，我们做出一下限制
在`C:\Users\<username>`中新建`.wslconfig`,用记事本打开后输入
```
[wsl2]
autoProxy=true
swap=16GB
memory=12GB
processors=2
```
可以按照自己需求修改内存，核心数，swap交换等限制




至此，wsl设置结束。

【1】[如何使用 WSL 在 Windows 上安装 Linux](https://learn.microsoft.com/zh-cn/windows/wsl/install)

【2】[什么是Windows PowerShell？怎么操作它？（秒懂） - 知乎](https://zhuanlan.zhihu.com/p/366637644)