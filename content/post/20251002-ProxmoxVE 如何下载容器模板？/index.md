---
title: ProxmoxVE 如何下载容器模板？
description:
date: 2025-10-02
slug: proxmox-how-to-download-ct
image: cover-proxmox-how-to-download-ct.png
categories:
    - Proxmox
    - Virtualization
tags:
    - Container
    - pct
---

✅ 方法一：通过 Web 界面下载 CT 模板

### 步骤 1：登录 Proxmox VE Web 管理界面

打开浏览器，访问 `https://<你的-PVE-IP>:8006`，使用管理员账户登录。

---

### 步骤 2：选择存储位置

1. 在左侧树形菜单中，点击你的 **PVE 节点名称**（例如 `pve`）。
2. 在右侧的“摘要”页面中，找到 **“存储”** 部分。
3. 点击存放CT模板的存储节点（通常是 `local` 或 `local-lvm`）。

> 💡 推荐使用 `local` 存储（目录存储），因为它支持存放模板文件。`local-lvm` 是块存储，不能直接存放模板。

---

### 步骤 3：进入“内容”选项卡并下载模板

1. 点击你选择的存储（如 `local`）后，切换到顶部的 **“内容”**（Content）选项卡。
2. 点击右上角的 **“模板”** 按钮（或下拉菜单中的“模板”）。
3. 系统会从官方镜像源（`https://images.linuxcontainers.org`）加载可用的 LXC 模板列表。

> ⚠️ 如果列表为空，请检查：
> 
> - PVE 节点是否能访问互联网。
> - 是否启用了正确的软件源（见下方“常见问题”）。

4. 在模板列表中，选择你需要的操作系统模板，例如：
  
  - `alpine-3.18-default_20230125_amd64.tar.zst`
  - `ubuntu-22.04-standard_22.04-1_amd64.tar.zst`
  - `debian-12-standard_12.0-1_amd64.tar.zst`
5. 点击右侧的 **“下载”** 按钮。
  
6. 系统会弹出确认窗口，确认存储位置和文件名，点击 **“下载”** 开始下载。
  

---

### 步骤 4：等待下载完成

- 下载进度会在任务栏（底部）显示。
- 下载完成后，模板会出现在该存储的“内容”列表中，类型为 `vztmpl`。

---

## ✅ 方法二：通过命令行下载 CT 模板

你也可以直接在 PVE 节点的 shell 中使用 `pveam` 命令下载模板。

### 1. 查看可用模板

```bash
pveam available
```

这会列出所有可下载的模板（来自 `turnkey` 和 `system` 源）。

### 2. 搜索特定模板

```bash
pveam available --section system | grep ubuntu
```

### 3. 下载模板

例如，下载 Ubuntu 22.04 模板：

```bash
pveam download system ubuntu-22.04-standard_22.04-1_amd64.tar.zst
```

> 📌 模板默认下载到 `/var/lib/vz/template/cache/` 目录。

### 4. 查看已下载的模板

```bash
pveam list local
```

### 🔄 模板源说明

PVE 使用两个主要模板源：

- **`system`**：来自 `https://images.linuxcontainers.org`，包含 Alpine、Ubuntu、Debian、CentOS 等。
- **`turnkey`**：来自 `https://images.turnkeylinux.org`，包含预配置的应用模板（如 WordPress、LAMP 等）。

你可以通过以下命令管理源：

```bash
pveam help
pveam update  # 更新模板列表
```

---

## ✅ 方法三：通过国内镜像源下载（推荐）

可以尝试使用国内镜像源（如中科大、阿里云），但 `pveam` 默认不支持更换源。

需手动下载模板文件并放入 `/var/lib/vz/template/cache/`。

### 1. 登录国内镜像源查看模板列表

```shell
curl https://mirrors.ustc.edu.cn/proxmox/images/system/
```

### 2. 切换至模板缓存目录

```shell
cd /var/lib/vz/template/cache/
```

### 3. 下载模板至本地

```shell
wget https://mirrors.ustc.edu.cn/proxmox/images/system/debian-12-standard_12.12-1_amd64.tar.zst
wget https://mirrors.ustc.edu.cn/proxmox/images/system/centos-8-default_20201210_amd64.tar.xz
wget https://mirrors.ustc.edu.cn/proxmox/images/system/rockylinux-9-default_20240912_amd64.tar.xz
wget https://mirrors.ustc.edu.cn/proxmox/images/system/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst  
```

### 4. 查看已下载模板

```shell
# 下载完成后CT模板文件如下
root@pve:/var/lib/vz/template/cache# ls -l
total 465092
-rw-r--r-- 1 root root  99098368 Dec 10  2020 centos-8-default_20201210_amd64.tar.xz
-rw-r--r-- 1 root root 123731847 Sep  7 23:49 debian-12-standard_12.12-1_amd64.tar.zst
-rw-r--r-- 1 root root 104371140 Sep 12  2024 rockylinux-9-default_20240912_amd64.tar.xz
-rw-r--r-- 1 root root 149046471 May  9 03:39 ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst
```

---

## ❓ 常见问题

### 1. 模板列表为空？

- **原因**：PVE 无法访问模板源。
  
- **解决**：
  
  - 检查网络连接：`ping images.linuxcontainers.org`
    
  - 检查 DNS：`nslookup images.linuxcontainers.org`
    
  - 确保 `/etc/apt/sources.list.d/pve-enterprise.list` 被注释（企业源需要订阅）。
    
  - 启用免费源（如 `pve-no-subscription`）：
    
    ```bash
    echo "deb http://download.proxmox.com/debian/pve $(grep "VERSION=" /etc/os-release | sed -r 's/.*="\w+ (\w+).*/\1/;s/^bookworm$/bullseye/') no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
    apt update
    ```
    

3. 下载的模板在哪？

- 路径：`/var/lib/vz/template/cache/`
- 文件格式：`.tar.zst`（Zstandard 压缩）

---

## ✅ 总结

| 方法  | 适用场景 |
| --- | --- |
| **Web 界面下载** | 简单直观，适合大多数用户 |
| **命令行 `pveam`** | 适合脚本化、批量操作或无法使用 Web 界面时 |
| **命令行 `wegt`** | **推荐**，合适国内网络环境 |

**推荐流程**：

1. 使用 `local` 存储。
2. 在 Web 界面中点击“模板”按钮。
3. 选择并下载你需要的 CT 模板。
4. 下载完成后即可用于创建容器。