---
title: ProxmoxVE 如何更换软件源？
description:
date: 2025-09-09
slug: proxmox-how-to-update-software-sources
image: https://ryan1998.dpdns.org/cover/post/proxmox/cover-proxmox-how-to-update-software-sources.png
categories:
    - Proxmox
tags:
    - Mirror
---


## 📌 重要说明

Proxmox VE 基于 Debian，因此其软件源与 Debian 兼容。PVE 有两个主要的源：

1. **系统软件源**：用于 `apt update` 和 `apt install`，管理 PVE 自身的更新。
2. **企业源（Enterprise）**：默认启用，但需要订阅才能使用。
3. **无订阅源（No-Subscription）**：免费使用，但默认可能未启用。
4. **LXC 模板源**：由 `pveam` 使用，默认指向国外的 `images.linuxcontainers.org`。

我们将分别配置这两类源。

---

## ✅ 一、更换 PVE 系统软件源为国内镜像（推荐）

### 步骤 1：备份原始源文件

```bash
cd /etc/apt/sources.list.d/
cp pve-enterprise.list pve-enterprise.list.bak
cp pve-install-repo.list pve-install-repo.list.bak  # 如果存在
```

### 步骤 2：禁用企业源（必须）

企业源需要有效订阅，否则会报错。

```bash
sed -i 's/^/#/' /etc/apt/sources.list.d/pve-enterprise.list
```

这会在 `pve-enterprise.list` 文件的每一行前加上 `#`，注释掉企业源。

### 步骤 3：添加国内无订阅源

推荐使用 **中科大（USTC）** 或 **阿里云** 镜像。

#### 方法 A：使用中科大镜像源

```bash
cat > /etc/apt/sources.list.d/pve-no-subscription.list << EOF
deb https://mirrors.ustc.edu.cn/proxmox/debian/pve $(grep "CODENAME" /etc/os-release | awk -F'=' '{print$2}') pve-no-subscription
EOF

# https://mirrors.ustc.edu.cn/proxmox/debian/pve/dists/bookworm/pve-no-subscription/binary-amd64/
```

> 🔍 脚本说明：自动识别系统版本（如 `bullseye` 或 `bookworm`）并插入。

#### 方法 B：手动编辑（推荐检查版本）

先查看你的 Debian 版本：

```bash
grep "VERSION=" /etc/os-release
# 示例输出：VERSION="7.2-1 (latest)"
# 实际版本 codename 可通过以下命令获取：
. /etc/os-release && echo $VERSION_CODENAME
```

常见对应关系：

- PVE 7.x → Debian `bullseye`
- PVE 8.x → Debian `bookworm`
- PVE 9.x → Debian `trixie`

然后创建文件：

```bash
nano /etc/apt/sources.list.d/pve-no-subscription.list
```

输入以下内容（以 **中科大** 为例）：

```conf
# Proxmox VE No-Subscription 镜像（中科大）
deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bookworm pve-no-subscription
```

如果你使用 **阿里云**：

```conf
# Proxmox VE No-Subscription 镜像（阿里云）
deb http://mirrors.aliyun.com/proxmox/debian/pve bookworm pve-no-subscription
```

> 📝 请根据你的实际版本（`bullseye` 或 `bookworm`）替换 `bookworm`。

### 步骤 4：更新软件包列表

```bash
apt update
```

如果无报错，说明源更换成功。

---

## ✅ 二、更换 LXC 模板下载源（可选，较复杂）

默认 LXC 模板从 `https://images.linuxcontainers.org` 下载，速度较慢。目前 **PVE 官方工具 `pveam` 不支持直接更换模板源**，但你可以：

### 方案 1：手动下载模板并放入缓存目录

1. 在浏览器或使用 `wget` 从国内镜像站下载模板：

   ```bash
   # 示例：从中科大镜像站下载 Ubuntu 22.04 模板
   wget https://mirrors.ustc.edu.cn/proxmox/images/system/debian-12-standard_12.12-1_amd64.tar.zst
   ```

   > 🔗 中科大 LXC 镜像站：[https://mirrors.ustc.edu.cn/proxmox/images/system/](https://mirrors.ustc.edu.cn/proxmox/images/system/)

2. 移动到 PVE 模板缓存目录：

   ```bash
   mv debian-12-standard_12.12-1_amd64.tar.zst /var/lib/vz/template/cache/
   ```

3. 在 Web 界面中创建容器时，即可选择该模板。

### 方案 2：修改 `pveam` 配置（高级，不推荐）

可以修改 `/usr/share/perl5/PVE/APLInfo.pm` 文件中的源地址，但容易在更新后被覆盖，不推荐。

---

## ✅ 三、更换 Debian 基础源（可选）

为了更快地安装普通 Debian 软件包，也可以更换 Debian 官方源。

```bash
nano /etc/apt/sources.list
```

替换为：

```conf
# Debian 国内镜像源（中科大）
deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free
deb https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free
deb https://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free
```

然后：

```bash
apt update
```

---

## ✅ 四、验证源是否生效

```bash
apt update
```

观察输出中的 URL 是否为国内镜像地址（如 `mirrors.ustc.edu.cn` 或 `mirrors.aliyun.com`）。

---

## 🛑 注意事项

1. **无订阅源不提供安全更新**：`pve-no-subscription` 源是社区维护的，更新可能延迟。生产环境建议购买订阅。
2. **不要同时启用企业源和无订阅源**：避免冲突。
3. **PVE 升级需谨慎**：更换源后升级 PVE 版本前，建议查阅官方文档。
4. **模板源无法完全自动化更换**：LXC 模板仍需手动下载或忍受较慢速度。

---

## ✅ 总结

| 源类型            | 推荐国内镜像     | 配置文件                                               |
| -------------- | ---------- | -------------------------------------------------- |
| **PVE 软件源**    | 中科大、阿里云    | `/etc/apt/sources.list.d/pve-no-subscription.list` |
| **Debian 基础源** | 中科大、清华、阿里云 | `/etc/apt/sources.list`                            |
| **LXC 模板源**    | 中科大（手动下载）  | 无直接配置，需手动放入 `/var/lib/vz/template/cache/`          |

**推荐操作**：

1. 注释 `pve-enterprise.list`
2. 创建 `pve-no-subscription.list` 指向中科大或阿里云
3. 运行 `apt update`
