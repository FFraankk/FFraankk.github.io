---
title: ROS2安装
date: 2025-3-17 2:00:00 +0800
categories: [Robotics,ROS]
tags: [ROS,Robotics]
author: FFraankk
---

本文由AI辅助完成

# ROS2 Humble 安装

配置条件：Ubuntu22.04；优质网络环境（建议使用clash的TUN模式，在使用TUN模式之前需要安装服务模式）或者国内镜像站

---

## 设置区域设置
确保系统使用支持 `UTF-8` 的区域设置。

```bash
locale  # 检查是否支持UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8  # 生成UTF-8区域设置
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8  # 永久设置
export LANG=en_US.UTF-8  # 临时设置当前会话

locale  # 验证设置
```

---

## 配置软件源

1. **启用Ubuntu Universe仓库**：
   ```bash
   sudo apt install software-properties-common
   sudo add-apt-repository universe
   ```

2. **添加ROS 2 GPG密钥**：
   ```bash
   sudo apt update && sudo apt install curl -y
   sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
   ```

3. **添加ROS 2软件源**：
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
   ```

---

## 安装ROS 2软件包

1. **更新软件包缓存**：
   ```bash
   sudo apt update
   ```

2. **升级系统（关键步骤）**：
   ```bash
   sudo apt upgrade
   ```


3. **选择安装模式**：
   - **桌面安装（推荐）**：包含ROS、RViz、示例和教程  
     ```bash
     sudo apt install ros-humble-desktop
     ```
   - **ROS基础安装（精简版）**：仅含通信库、消息包和命令行工具  
     ```bash
     sudo apt install ros-humble-ros-base
     ```

4. **安装开发工具**：
   ```bash
   sudo apt install ros-dev-tools  # 包含编译器等构建ROS包的工具
   ```

---

### 关键说明
- 所有`ros-humble-*`的包名对应ROS 2 Humble版本
- 安装完成后需配置环境变量：  
  ```bash
  source /opt/ros/humble/setup.bash
  ```
- 若需长期使用，可将环境变量写入`~/.bashrc`
   ```
   echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
   ```

---

## **尝试示例**  
如果已安装 `ros-humble-desktop`，可通过以下示例验证ROS 2功能。

---

### **Talker-Listener 测试**  
1. **启动C++ Talker节点**  
   打开第一个终端，加载ROS 2环境并运行C++版本的Talker：  
   ```bash
   source /opt/ros/humble/setup.bash
   ros2 run demo_nodes_cpp talker
   ```  
   **预期输出**：  
   ```
   [INFO] [talker]: Publishing: "Hello World: {整数序号}"
   ```

2. **启动Python Listener节点**  
   打开第二个终端，加载ROS 2环境并运行Python版本的Listener：  
   ```bash
   source /opt/ros/humble/setup.bash
   ros2 run demo_nodes_py listener
   ```  
   **预期输出**：  
   ```
   [INFO] [listener]: I heard: "Hello World: {整数序号}"
   ```

3. **验证结果**  
   - **Talker终端** 持续输出发布的消息。  
   - **Listener终端** 持续显示接收到的消息。  
   若两者正常交互，表明ROS 2的**C++和Python API**均工作正常，安装成功！

---


 