---
title: ProxmoxVE 如何创建虚拟机(VM)和容器(CT)？
description:
date: 2025-10-10
slug: proxmox-how-to-create-virtual-machines-and-containers
image: https://ryan1998.dpdns.org/cover/post/proxmox/cover-proxmox-how-to-create-virtual-machines-and-containers.png
categories:
    - Proxmox
    - KVM
    - LXC
tags:
    - Virtual Machine
    - Container
    - qm
    - pct
---


在 Proxmox Virtual Environment (Proxmox VE 或 PVE) 中，创建虚拟机 (VM) 和容器 (CT, 即 LXC 容器) 是核心功能。以下是详细的创建步骤：

> 准备工作：
>
> 1. 提前下载系统镜像，并上传至 `local`  `ISO 镜像`
>
>    ![upload-iso](https://ryan1998.dpdns.org/illus/create-vm-and-ct/0.1-upload-iso.png)
>
> 2. 提前下载容器模板，并上传至 `local` `CT 模板`
>
>    ![upload-ct](https://ryan1998.dpdns.org/illus/create-vm-and-ct/0.2-upload-ct.png)

<div style="page-break-after: always;"></div>

## **一、 创建虚拟机 (VM)**

虚拟机是完全虚拟化的实例，可以运行任何支持的操作系统（如 Windows、Linux、BSD 等）。

### **步骤 1：登录 Proxmox Web 界面**

- 打开浏览器，访问 `https://<你的PVE服务器IP>:8006`

- 使用 root 或具有权限的用户登录。

  ![login-proxmox](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.1.1-login-proxmox.png)

- 忽略 **无有效订阅**

  ![ignore-subsribe](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.1.2-ignore-subsribe.png)

<div style="page-break-after: always;"></div>

### **步骤 2：点击“创建虚拟机”**

- 在左侧服务器节点上右键，或点击顶部工具栏的 **“创建 VM”** 按钮。

  ![create-vm-button](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.2.1-create-vm-button.png)

<div style="page-break-after: always;"></div>

### **步骤 3：配置虚拟机参数**

#### **1. 常规 (General)**

- **VM ID**: 自动分配或手动指定（唯一数字 ID）。

- **名称**: 为虚拟机命名（如 `Gateway`）。

- **资源池**（可选）: 如果你使用资源池管理，可选择归属池。

- **高级**

  - 开机自启
  - 添加标签

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.3.1-create-vm-general.png)

<div style="page-break-after: always;"></div>

#### **2. 操作系统 (OS)**

- **ISO 镜像**: 从下拉菜单选择已上传的 ISO 文件（需提前上传到 `local` 或 `local-lvm` 存储）。

- **类型**: 选择 Guest OS 类型（如 Linux、Windows）和版本（如 Ubuntu 22.04）。

  > 💡 提示：选择正确的类型有助于 Proxmox 自动优化配置（如默认使用 VirtIO 驱动）。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.3.2-create-vm-iso.png)

<div style="page-break-after: always;"></div>

#### **3. 系统 (System)**

- **图形卡**: 通常选 `Default` 或 `VirtIO-GPU`。

- **BIOS**: 推荐 `OVMF (UEFI)`（尤其对 Windows 11 或 Secure Boot 支持更好）。

- **TPM**: 如需安装 Windows 11，勾选“添加 TPM 设备”。

- **机器类型**: 通常保持默认 `q35`。

- **SCSI 控制器**: 推荐 `VirtIO SCSI`（性能更好）。

- **Qemu Agent**: **强烈建议启用** —— 便于在 PVE 界面内查看 IP、关机、冻结文件系统等。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.3.3-create-vm-system.png)

<div style="page-break-after: always;"></div>

#### **4. 硬盘 (Hard Disk)**

- **总线/设备**: 推荐 `SCSI`（配合 VirtIO SCSI 控制器）。

- **存储**: 选择存储位置（如 `local-lvm` ）。

- **磁盘大小**: 如 `32G`。

- **缓存**: 通常选 `默认` 或 `Write back`（性能更好，但需 UPS 保障）。

- **丢弃 (Discard)**: 勾选以支持 TRIM（SSD 环境推荐）。

- **SSD 仿真**: 勾选（尤其对 Windows 客户机）。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.3.4-create-vm-hard-disk.png)

<div style="page-break-after: always;"></div>

#### **5. CPU**

- **核心数**: 如 `4`。

- **类型**: 推荐 `host`（获得最佳性能）或 `kvm64`（兼容性更好）。

- **Sockets / Cores / Threads**: 一般保持默认即可，或根据授权要求调整（如 Windows 授权限制）。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.3.5-create-vm-cpu.png)

<div style="page-break-after: always;"></div>

#### **6. 内存**

- **内存大小**: 如 `2048 MB`。

- **Ballooning**: 可选启用（允许动态调整内存，但需安装 qemu-guest-agent）。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.3.6-create-vm-memory.png)

<div style="page-break-after: always;"></div>

#### **7. 网络 (Network)**

- **网桥**: 默认 `vmbr0`（连接到物理网络）。

- **模型**: **强烈推荐 `VirtIO`**（半虚拟化，性能最佳）。

- **MAC 地址**: 自动生成或手动指定。

- **防火墙**: 按需启用。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.3.7-create-vm-network.png)

<div style="page-break-after: always;"></div>

#### **8. 确认并完成**

- 点击 **“完成”** 创建虚拟机。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.3.8-create-vm-complate.png)

<div style="page-break-after: always;"></div>

### **步骤 4：启动并安装系统**

- 在左侧列表中选中新建的 VM，点击 **“启动”**。

- 点击 **“控制台”**（通常为 noVNC）进入安装界面。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.4.1-create-vm-start-now.png)

<div style="page-break-after: always;"></div>

- 按照操作系统安装向导完成安装。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.4.2-create-vm-install-system.png)

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/1.4.3-create-vm-startup-system.png)

<div style="page-break-after: always;"></div>

### **步骤 5：安装 QEMU Guest Agent（可选）**

- **Linux**: 安装 `qemu-guest-agent` 包（Ubuntu/Debian: `apt install qemu-guest-agent`；CentOS/RHEL: `yum install qemu-guest-agent`）。
- **Windows**: 下载并安装 [virtio 驱动 ISO](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/) 中的 `qemu-ga-x64.msi`。

<div style="page-break-after: always;"></div>

## **二、 创建容器 (CT - LXC Container)**

容器是轻量级、基于操作系统的虚拟化，仅支持 Linux，启动快、资源占用少。

### **步骤 1：点击“创建 CT”**

- 在节点上点击顶部工具栏 **“创建 CT”**。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.1.1-create-ct-button.png)

<div style="page-break-after: always;"></div>

### **步骤 2：配置容器参数**

#### **1. 常规 (General)**

- **CT ID**: 唯一数字 ID。

- **主机名**: 容器主机名（如 `Left`）。

- **资源池**（可选）。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.1-create-ct-general.png)

<div style="page-break-after: always;"></div>

#### **2. 模板 (Template)**

- 从下拉菜单选择已下载的模板（如 `ubuntu-22.04-standard...`）。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.2-create-ct-template.png)

<div style="page-break-after: always;"></div>

#### **3. 根磁盘 (Root Disk)**

- **存储**: 选择存储位置。

- **磁盘大小**: 如 `8G`（容器通常不需要很大空间）。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.3-create-ct-root-disk.png)

<div style="page-break-after: always;"></div>

#### **4. CPU**

- **核心数**: 如 `1`。
- ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.4-create-ct-cpu.png)

<div style="page-break-after: always;"></div>

#### **5. 内存**

- **内存大小**: 如 `512 MiB`。

- **交换空间 (Swap)**: 如 `512 MiB`。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.5-create-ct-memory.png)

<div style="page-break-after: always;"></div>

#### **6. 网络 (Network)**

- **网桥**: `vmbr0`。

- **IPv4/IPv6**: 可选 DHCP 或手动设置静态 IP。

- **网关**: 自动或手动填写。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.6-create-ct-network.png)

<div style="page-break-after: always;"></div>

#### **7. DNS**

- 设置 DNS 服务器（如 `1.1.1.1`）和搜索域。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.7-create-ct-dns.png)

<div style="page-break-after: always;"></div>

#### **8. 确认并完成**

- 点击 **“完成”** 创建容器。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.8-create-ct-complate.png)

- 输出 `TASK OK` 说明容器创建完成。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.2.9-create-ct-taskok.png)

<div style="page-break-after: always;"></div>

### **步骤 3：启动容器**

- 选中新建的 CT，点击 **“启动”**。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.3.1-create-ct-start-now.png)

- 可点击 **“控制台”** 进入 shell。

- 默认 root 密码是你在创建时设置的密码。

  ![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.3.2-create-ct-shell.png)

<div style="page-break-after: always;"></div>

### **步骤 4：进入容器管理**

```bash
# 在 Proxmox 主机命令行中：
pct enter <CTID>  # 如 pct enter 100
```

或通过 Web 界面控制台操作。

![](https://ryan1998.dpdns.org/illus/create-vm-and-ct/2.4.1-create-ct-pct-enter.png)

<div style="page-break-after: always;"></div>

## **三、常用操作对比**

| 功能     | 虚拟机 (VM)               | 容器 (CT)             |
| ------ | ---------------------- | ------------------- |
| 启动速度   | 较慢（需引导 OS）             | 极快（秒级）              |
| 资源开销   | 较高                     | 极低                  |
| 操作系统支持 | 任意（Windows/Linux/BSD等） | 仅 Linux             |
| 隔离性    | 完全隔离（Hypervisor 层）     | 进程级隔离（共享内核）         |
| 性能     | 接近原生（尤其 VirtIO 优化后）    | 接近原生                |
| 快照/备份  | 支持                     | 支持                  |
| 图形界面   | 支持（VNC/SPICE）          | 通常无（纯命令行）           |
| 适用场景   | 运行 Windows、不同内核、测试环境等  | 微服务、Web 服务、数据库、轻量应用 |

---

通过以上步骤，你就可以在 Proxmox VE 中成功创建和管理虚拟机与容器了。根据你的使用场景选择合适的虚拟化技术，通常：

- **需要运行 Windows 或完全隔离环境 → 选 VM**
- **追求轻量、快速部署 Linux 服务 → 选 CT**
