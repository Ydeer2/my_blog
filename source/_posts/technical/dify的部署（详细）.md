---
cover: https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/image-20260323204633263.png
title: Dify的部署（详细）
date: 2026-3-23
categories:
  - Dify
tags:
  - 笔记
---

# Dify的部署（详细）

## 什么是Dify？

Dify是一个开源的大语言模型（LLM）应用开发平台，旨在简化和加速生成式AI应用的创建和部署。

简单点说就是一个AI Agent的低代码开发开发平台。

Dify可以直接在官网里面使用云服务，但是由于官网是在外网，并且免费版有额度。速度慢也不划算。

不过好在Dify是开源的，我们可以自己部署，自己玩。

## Dify部署平台的选择

Dify的部署依赖docker，不依赖操作系统。

Linux，maxOS和Windows的Dify部署区别如下

| 操作系统    | 主要部署方式                | 说明                                                                                                      |
| :---------- | :-------------------------- | :-------------------------------------------------------------------------------------------------------- |
| **Linux**   | **Docker / Docker Compose** | 最原生、最高效的方式。所有官方文档和教程都以Linux（如Ubuntu, CentOS）为例。                               |
| **macOS**   | **Docker Desktop**          | 安装Docker Desktop for Mac后，即可使用与Linux完全相同的`docker-compose`命令进行部署。                     |
| **Windows** | **Docker Desktop + WSL 2**  | 需要在Windows上先安装Docker Desktop，并启用WSL 2（Windows Subsystem for Linux）后端，然后运行Docker命令。 |

这里最推荐使用Linux部署dify，首先如果你要用Windows部署Dify的话，你要启动WSL 2，这相当于你要在Window里搞一个linux环境，这个linux不是使用虚拟化硬件的方式，而是直接使用你的硬件，但是这个linux的存储空间默认是在c盘，改存储空间挺麻烦的，macOS我没用过，但是用过都说好。这里我演示的使用linux虚拟机部署Dify，这个挺方便的，要用的时候打开，不用直接关闭，最后觉得占空间直接连同系统一起删除。干净又快捷。

## 部署步骤

### 第一步：下载所需软件（linux虚拟机）

- **虚拟机软件**：推荐使用 **VMware Workstation Pro**（付费，功能全面）或 **VirtualBox**（免费开源）。
  - VMware 官网：https://www.vmware.com/products/workstation-pro.html
  - VirtualBox 官网：https://www.virtualbox.org/
- **Linux 系统镜像**： **Rocky Linux**（长期支持版，稳定，兼容性好），这里以**Rocky Linux**为例，因为博主之前使用的一直是centOS的发行版。
  - 下载地址：https://rockylinux.org/download

这里就不给具体的教程了，可以直接在网上搜索linux虚拟机安装教程。

Dify官网上描述Dify运行的最低配置如下：

- **CPU**：至少 2 核
- **内存**：分配给虚拟机 **4GB** 以上（Dify 推荐 4GB+，8GB 更佳）
- **磁盘**：分配给虚拟机 **至少 20GB** 空闲空间

### 第二步：初始化 Rocky Linux 系统

#### 2.1 更新系统（可选但推荐）

```bash
sudo dnf update -y
```

#### 2.3 安装常用工具

```bash
sudo dnf install -y vim wget curl git net-tools
```

---

### 第三步： 安装 Docker 与 Docker Compose

Rocky Linux 9 默认不含 Docker，需添加官方仓库。

#### 3.1 卸载旧版本（如有）

```bash
sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

#### 3.2 安装必要依赖并添加 Docker 仓库（使用阿里云镜像）

```bash
sudo dnf install -y yum-utils
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### 3.3 安装 Docker Engine 及相关插件

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 3.4 启动 Docker 并设置开机自启

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

#### 3.5 将当前用户加入 docker 组（避免每次输入 sudo）

```bash
sudo usermod -aG docker $USER
```

**退出当前终端并重新登录**，或执行 `newgrp docker` 使权限生效。

#### 3.6 配置 Docker 镜像加速器（强烈推荐）

由于国内访问 Docker Hub 较慢，编辑 `/etc/docker/daemon.json`：

bash

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.xuanyuan.me",
    "https://docker.1ms.run",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF
```

重启 Docker：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 3.7 验证安装

```bash
docker --version
docker compose version
```

---

### 第四步： 部署 Dify

#### 4.1 克隆 Dify 仓库

```dify
git clone https://github.com/langgenius/dify.git
cd dify/docker
```

#### 4.2 复制环境变量模板

```bash
cp .env.example .env
```

如果需要修改端口（例如默认的 80 端口已被占用），可以编辑 `.env` 文件：

```bash
vim .env
```

找到 `EXPOSE_NGINX_PORT=80`，修改为你需要的端口（如 `EXPOSE_NGINX_PORT=8080`）。

#### 4.3 启动 Dify 服务

```bash
docker compose up -d
```

![image-20260323201802997](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/image-20260323201802997.png)

首次启动会拉取所有必需的镜像（约 2-3GB），过程10~30分钟，请耐心等待。可通过以下命令查看进度：

```bash
docker compose ps
```

所有容器状态应为 `Up` 才算成功。

---

### 第五步： 防火墙配置

Rocky Linux 9 默认使用 firewalld，需要放行 Dify 的访问端口（如 80 或自定义端口）：

```bash
sudo firewall-cmd --permanent --add-port=80/tcp   # 若你修改了端口，请替换
sudo firewall-cmd --reload
```

---

### 第六步：访问 Dify 并初始化

打开浏览器，输入 `http://<虚拟机IP>:<端口>`（例如 `http://192.168.88.130:8080`）。首次访问会自动跳转到初始化页面：

1. 设置管理员邮箱、用户名、密码。
2. 登录后，进入右上角头像 → **设置** → **模型供应商**，添加你的大模型 API Key（如 OpenAI、DeepSeek、Azure OpenAI 等）。

至此，Dify 已成功部署并可以使用。

![image-20260323202017855](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/image-20260323202017855.png)
