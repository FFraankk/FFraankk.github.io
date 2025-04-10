---
title: ROS2和Miniconda安装
date: 2025-3-17 2:00:00 +0800
categories: [Robotics,ROS]
tags: [ROS,Robotics,conda,miniconda]
author: FFraankk
---

本文由AI辅助完成



# ROS2 Humble 安装

配置条件：Ubuntu22.04；优质网络环境（建议使用clash的TUN模式，在使用TUN模式之前需要安装服务模式）或者国内镜像站
原文地址：[Ubuntu (deb packages)](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html)

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


 
# Miniconda
本来本人是不喜欢用虚拟环境的，但是由于本人研究的宇树的机器狗开发环境使用到了miniconda，所以也不得不安装虚拟环境，正好写一些来记录。
**Miniconda** 是 Anaconda 的轻量版，仅包含 Python、Conda 包管理器和基础依赖，支持灵活创建虚拟环境，适合对空间敏感或需自定义配置的场景。  

**对比**：  
1. **Anaconda**：预装 1500+ 数据科学库（如 NumPy、Pandas），开箱即用，但占用数GB空间，适合新手或全栈数据科学。  
2. **Miniconda**：仅核心组件，需手动安装所需包，轻量（约400MB），适合开发者精准控制环境。  
3. **venv**：Python 内置模块，仅隔离Python依赖，无跨平台包管理功能，依赖pip，轻量但功能单一，适合纯Python简单项目。  


## 安装并且初始化
### 下载并安装Miniconda
在终端中输入如下命令
```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```

安装完成后，您需要初始化Conda：
```
~/miniconda3/bin/conda init --all
source ~/.bashrc
```
### 创建新的环境

```
conda create -n <envname> python=3.8
```
### 进入虚拟环境
```
conda activate <envname>
```
### 禁用默认激活base环境
```
conda config --set auto_activate_base false
```


## Miniconda 常用操作指南
---

### 一、环境管理
1. **创建新环境**  
   - 指定 Python 版本：  
     ```bash
     conda create -n env_name python=3.8  # 创建名为 env_name 的环境，Python 版本为 3.8
     ```
   - 安装指定包的同时创建环境：  
     ```bash
     conda create -n env_name numpy pandas  # 创建环境并安装 numpy、pandas
     ```

2. **激活与退出环境**  
   ```bash
   conda activate env_name  # 激活环境
   conda deactivate         # 退出当前环境
   ```

3. **查看与删除环境**  
   ```bash
   conda env list           # 列出所有环境
   conda remove -n env_name --all  # 删除指定环境
   ```

4. **克隆与导出环境**  
   - 克隆环境：  
     ```bash
     conda create --name new_env --clone old_env  # 克隆 old_env 为 new_env
     ```
   - 导出环境配置：  
     ```bash
     conda env export > environment.yml  # 导出当前环境的包列表
     ```

---

### 二、包管理
1. **安装与卸载包**  
   ```bash
   conda install numpy       # 安装包（当前环境）
   conda install -n env_name numpy  # 在指定环境中安装包
   conda remove numpy        # 卸载包
   ```

2. **更新包与 conda 自身**  
   ```bash
   conda update numpy       # 更新指定包
   conda update --all       # 更新所有包
   conda update conda       # 更新 conda 工具
   ```

3. **查看包信息**  
   ```bash
   conda list              # 列出当前环境已安装的包
   conda search pytorch    # 搜索可用包
   ```

---

### 三、镜像源配置
1. **切换清华源（加速下载）**  
   ```bash
   conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
   conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
   conda config --set show_channel_urls yes
   ```

2. **恢复默认源**  
   ```bash
   conda config --remove-key channels  # 移除所有镜像源
   ```

---

### 四、其他实用操作
1. **修改默认环境路径**  
   - 编辑配置文件 `~/.condarc`，添加以下内容：  
     ```yaml
     envs_dirs:
       - /path/to/custom/envs  # 自定义环境存储路径
     pkgs_dirs:
       - /path/to/custom/pkgs  # 自定义包存储路径
     ```

2. **清理缓存**  
   ```bash
   conda clean --all  # 清理所有缓存和未使用的包
   ```

3. **验证安装与版本**  
   ```bash
   conda --version    # 查看 conda 版本
   python --version   # 查看当前 Python 版本
   ```

---

### 五、常见场景示例
- **PyTorch 环境创建**：  
  ```bash
  conda create -n pytorch_env python=3.8
  conda activate pytorch_env
  conda install pytorch torchvision -c pytorch
  ```

- **在 PyCharm 中使用 Conda 环境**：  
  在 PyCharm 的终端中直接运行 `conda activate env_name`，或在项目设置中手动选择 Conda 环境的 Python 解释器路径。

---

通过以上操作，可高效管理多个独立开发环境，避免依赖冲突。如需更详细命令说明，可参考 [Conda 官方文档](https://docs.conda.io/) 或相关博客教程。