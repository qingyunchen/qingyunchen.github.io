---
layout:     post
title:      WSL2 (Ubuntu 22.04) 机器学习环境配置实录
subtitle:   深度学习环境从安装到 VS Code 远程开发
date:       2026-04-18
author:     BY chenqingyun
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - 机器学习
    - Linux
    - WSL
    - PyTorch
    - 环境配置
---

在 Windows 环境下进行机器学习开发，WSL2 是目前最高效的选择之一。本文记录了从零开始配置 Ubuntu 22.04 机器学习环境的全过程，包括 CUDA 检查、PyTorch/TensorFlow 安装以及 VS Code 远程开发配置。

---

## 1. WSL 安装与基础管理

### 安装 Ubuntu 22.04
在 PowerShell 中运行：
```bash
wsl --install -d Ubuntu-22.04
```
- **默认账户**：username
- **默认密码**：123456

### 常用管理命令
- **查看已安装版本**：`wsl -l -v`
- **启动**：`wsl -d Ubuntu-22.04`
- **停止所有 WSL 实例**：`wsl --shutdown`
- **退出当前终端**：输入 `exit` 或按 `Ctrl + D`

---

## 2. 环境检查

进入 Ubuntu 后，首先确认系统版本和驱动状态：

```bash
# 1. 检查 Ubuntu 版本
cat /etc/os-release

# 2. 检查 NVIDIA 驱动和 CUDA 是否就绪
nvidia-smi
```
> **注意**：如果 `nvidia-smi` 能够显示显卡信息，说明 Windows 宿主机的驱动已经正确映射到 WSL。

---

## 3. Python 基础环境配置

更新包列表并安装 pip：

```bash
sudo apt update
sudo apt upgrade -y

# 安装 pip
sudo apt install python3-pip -y

# 安装虚拟环境工具（我不安装）
pip3 install virtualenv
```

---

## 4. 深度学习框架安装

### 安装 PyTorch
根据 `nvidia-smi` 显示的 CUDA 版本进行选择。如果显示 CUDA 11.7，建议安装兼容的 CUDA 11.8 版本：

```bash
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

**验证 GPU 是否可用：**
```bash
python3 -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"
```

### 安装 TensorFlow(我不安装)
```bash
pip3 install tensorflow
```

**验证：**
```bash
python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```

---

## 5. VS Code 远程开发配置

### 进入 WSL 环境
1. 点击 VS Code 左下角的蓝色图标（两个尖尖）。
2. 选择 **Connect to WSL**。

### 插件安装注意事项
在 WSL 环境下，VS Code 需要重新安装相关插件（如 Python）。
- **离线安装**：如果网络较慢，可以在插件设置页面手动下载 `.vsix` 文件，然后通过插件面板右上角的 `...` 选择 **Install from VSIX** 导入。
- **选择解释器**：按 `Ctrl + Shift + P`，搜索 `Python: Select Interpreter`，配置正确的 Python 地址。

### 项目路径建议
建议将项目文件夹放在 WSL 的本地文件系统中，或者通过 `/mnt/` 访问 Windows 磁盘：
- **新建文件夹**：`F:\study\pytorch`
- **WSL 访问路径**：`/mnt/f/study/pytorch`

---

通过以上步骤，你就可以在 WSL 环境下畅快地进行机器学习开发了！
