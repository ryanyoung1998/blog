---
title: ProxmoxVE 如何使用 qm 命令管理 KVM 虚拟机？
description:
date: 2025-09-30
slug: proxmox-how-to-use-qm-manage-KVM-virtual-machines
image: https://ryan1998.dpdns.org/cover/post/proxmox/cover-proxmox-how-to-use-qm-manage-KVM-virtual-machines.png
categories:
    - Proxmox
    - KVM
tags:
    - Virtual Machines
    - qm
---



`qm` 是 Proxmox VE (PVE) 中用于**管理虚拟机 (QEMU/KVM)** 的核心命令行工具。它功能强大，几乎可以完成 Web 界面能做的所有操作，非常适合自动化脚本、批量管理和高级配置。

---

## **一、基本语法**

```bash
qm <command> <vmid> [options]
```

- `<command>`: 要执行的操作（如 `start`, `stop`, `create`, `set` 等）。
- `<vmid>`: 虚拟机的唯一数字 ID（如 `100`, `201`）。
- `[options]`: 命令的附加参数。

---

## **二、常用命令分类详解**

### **1. 生命周期管理（启动/停止/重启/重置）**

| 命令                   | 说明                 | 示例                |
| -------------------- | ------------------ | ----------------- |
| `qm start <vmid>`    | 启动虚拟机              | `qm start 100`    |
| `qm stop <vmid>`     | 优雅关机（发送 ACPI 信号）   | `qm stop 100`     |
| `qm shutdown <vmid>` | 同 `stop`，优雅关机      | `qm shutdown 100` |
| `qm reset <vmid>`    | 强制重启（相当于物理重置按钮）    | `qm reset 100`    |
| `qm suspend <vmid>`  | 挂起虚拟机（保存内存状态到磁盘）   | `qm suspend 100`  |
| `qm resume <vmid>`   | 恢复挂起的虚拟机           | `qm resume 100`   |
| `qm poweroff <vmid>` | 强制断电（不推荐，可能导致数据损坏） | `qm poweroff 100` |

> 💡 提示：`stop` 和 `shutdown` 会等待客户机操作系统正常关闭，如果客户机无响应，可使用 `poweroff` 强制关闭。

---

### **2. 创建与克隆**

#### **创建新虚拟机 (`qm create`)**

这是最复杂的命令之一，需要指定大量参数。

**基本结构：**

```bash
qm create <vmid> [OPTIONS]
```

**常用选项：**

```bash
# 创建一个基础 Ubuntu VM 示例
qm create 100 \
  --name ubuntu-test \
  --memory 2048 \
  --cores 2 \
  --net0 virtio,bridge=vmbr0 \
  --scsihw virtio-scsi-pci \
  --scsi0 local-lvm:32 \
  --ide2 local:iso/ubuntu-22.04.iso,media=cdrom \
  --boot order=scsi0;ide2 \
  --ostype l26 \
  --agent enabled=1
```

**关键参数解释：**

| 参数                                                  | 说明                   | 示例值                                    |
| --------------------------------------------------- | -------------------- | -------------------------------------- |
| `--name`                                            | 虚拟机名称                | `web-server`                           |
| `--memory`                                          | 内存大小 (MB)            | `2048`                                 |
| `--cores`                                           | CPU 核心数              | `2`                                    |
| `--sockets`                                         | CPU 插槽数              | `1`                                    |
| `--cpu`                                             | CPU 类型               | `host` (最佳性能), `kvm64` (兼容)            |
| `--net<n>`                                          | 网络配置                 | `virtio,bridge=vmbr0,firewall=1`       |
| `--scsihw`                                          | SCSI 控制器类型           | `virtio-scsi-pci`                      |
| `--scsi<n>`, `--virtio<n>`, `--ide<n>`, `--sata<n>` | 磁盘配置                 | `local-lvm:32,format=qcow2`            |
| `--ide2` (或 `--cdrom`)                              | 挂载 ISO               | `local:iso/ubuntu.iso,media=cdrom`     |
| `--boot`                                            | 启动顺序                 | `order=scsi0;ide2`                     |
| `--ostype`                                          | 操作系统类型               | `l26` (Linux 2.6+), `wxp`, `w7`, `w11` |
| `--agent`                                           | 启用 QEMU Guest Agent  | `enabled=1,fstrim_cloned_disks=1`      |
| `--bios`                                            | BIOS 类型              | `ovmf` (UEFI), `seabios` (传统)          |
| `--machine`                                         | 机器类型                 | `q35` (推荐), `i440fx`                   |
| `--tpmstate`                                        | 添加 TPM 设备 (Win11 需要) | `storage=local-lvm`                    |
| `--tablet`                                          | 启用 USB 平板设备 (改善鼠标体验) | `0` (禁用，VNC 下推荐) 或 `1`                 |

> 📌 **磁盘格式说明**：
>
> - `local-lvm:32` → 在 `local-lvm` 存储上创建 32GB 磁盘，使用默认格式（raw）。
> - `local-lvm:32,format=qcow2` → 指定格式为 qcow2。
> - `local:vm-100-disk-0.raw,size=32G` → 使用指定文件名和大小。

#### **克隆虚拟机 (`qm clone`)**

```bash
qm clone <源vmid> <新vmid> [OPTIONS]
```

**示例：**

```bash
# 克隆 VM 100 为 VM 101，命名为 "clone-vm"
qm clone 100 101 --name clone-vm --full 1

# 克隆为链接克隆（节省空间，依赖源磁盘）
qm clone 100 102 --name linked-clone --full 0
```

**关键选项：**

- `--full 1`: 完全克隆（独立磁盘）。
- `--full 0`: 链接克隆（共享磁盘快照）。
- `--name`: 新虚拟机名称。
- `--description`: 描述信息。

---

### **3. 配置管理 (`qm set`, `qm config`)**

#### **修改虚拟机配置 (`qm set`)**

用于修改已存在 VM 的配置。

**示例：**

```bash
# 增加内存到 4GB
qm set 100 --memory 4096

# 增加一个新硬盘 (scsi1)
qm set 100 --scsi1 local-lvm:16

# 添加第二个网卡
qm set 100 --net1 virtio,bridge=vmbr1

# 启用/禁用 QEMU Agent
qm set 100 --agent enabled=1

# 修改启动顺序
qm set 100 --boot order=ide2;scsi0

# 修改 CPU 核心数
qm set 100 --cores 4

# 删除某个设备 (如 cdrom)
qm set 100 --delete ide2

# 锁定虚拟机防止误操作
qm set 100 --lock backup  # 或 migrate, snapshot, rollback, suspend
```

#### **查看虚拟机配置 (`qm config`)**

```bash
qm config <vmid>
```

**示例：**

```bash
qm config 100
```

输出类似：

```
agent: enabled=1
boot: order=scsi0;ide2
cores: 2
ide2: local:iso/ubuntu-22.04.iso,media=cdrom
memory: 2048
name: ubuntu-test
net0: virtio=BC:24:11:AA:BB:CC,bridge=vmbr0
ostype: l26
scsi0: local-lvm:vm-100-disk-0,size=32G
scsihw: virtio-scsi-pci
```

---

### **4. 状态与信息查看**

| 命令                         | 说明                              | 示例                      |
| -------------------------- | ------------------------------- | ----------------------- |
| `qm list`                  | 列出所有虚拟机及其状态                     | `qm list`               |
| `qm status <vmid>`         | 查看指定 VM 的运行状态                   | `qm status 100`         |
| `qm current <vmid>`        | 显示当前运行时的配置（与 config 不同，包含运行时状态） | `qm current 100`        |
| `qm pending <vmid>`        | 显示待处理的配置更改（重启后生效）               | `qm pending 100`        |
| `qm running-config <vmid>` | 显示当前生效的配置（包含运行时和持久化配置）          | `qm running-config 100` |

**`qm list` 输出示例：**

```
VMID       NAME                 STATUS     MEM(MB)    BOOTDISK(GB) PID
100        ubuntu-test          running    2048       32.00        12345
101        clone-vm             stopped    2048       32.00        0
```

---

### **5. 快照管理 (`qm snapshot`, `qm rollback`, `qm delsnapshot`)**

#### **创建快照**

```bash
qm snapshot <vmid> <snapshot-name> [OPTIONS]
```

**示例：**

```bash
# 创建名为 "pre-update" 的快照
qm snapshot 100 pre-update --description "Before system update"

# 创建快照并同时停止 VM（确保数据一致性）
qm snapshot 100 pre-update --vmstate stop
```

#### **查看快照**

```bash
qm listsnapshot <vmid>
```

#### **回滚快照**

```bash
qm rollback <vmid> <snapshot-name>
```

**示例：**

```bash
qm rollback 100 pre-update
```

#### **删除快照**

```bash
qm delsnapshot <vmid> <snapshot-name>
```

**示例：**

```bash
qm delsnapshot 100 pre-update
```

---

### **6. 备份与恢复 (`qm backup`, `qm restore`)**

> ⚠️ 注意：`qm backup` 命令在较新版本的 PVE 中已被弃用，推荐使用 `vzdump` 命令进行备份。

#### **使用 `vzdump` 备份 VM**

```bash
# 备份 VM 100 到指定存储
vzdump 100 --storage backup-storage --compress zstd --mode snapshot

# 带描述和邮件通知
vzdump 100 --storage backup --notes "Daily backup" --mailto admin@example.com
```

#### **恢复备份 (`qm restore`)**

```bash
# 从备份文件恢复到新的 VM ID (102)
qm restore /var/lib/vz/dump/vzdump-qemu-100-2025_09_15-10_00_00.vma.zst 102 --storage local-lvm

# 覆盖现有 VM (谨慎使用!)
qm restore /path/to/backup.vma.zst 100 --force
```

---

### **7. 迁移与移动 (`qm migrate`, `qm move-disk`)**

#### **在线迁移 (Live Migration)**

```bash
qm migrate <vmid> <目标节点> [OPTIONS]
```

**示例：**

```bash
# 将 VM 100 迁移到节点 pve2
qm migrate 100 pve2 --online

# 迁移并指定迁移网络
qm migrate 100 pve2 --online --migration_network 192.168.10.0/24
```

#### **移动磁盘到其他存储**

```bash
qm move-disk <vmid> <磁盘标识符> <目标存储> [OPTIONS]
```

**示例：**

```bash
# 将 VM 100 的 scsi0 磁盘移动到 storage2
qm move-disk 100 scsi0 storage2 --delete 1  # 移动后删除源磁盘
```

---

### **8. 控制台访问 (`qm terminal`, `qm vncproxy`)**

#### **串行控制台 (需客户机配置串口)**

```bash
qm terminal <vmid>
```

#### **获取 VNC 代理信息 (用于外部 VNC 客户端连接)**

```bash
qm vncproxy <vmid>
```

输出类似：

```
{"status":"OK","data":{"port":5900,"ticket":"TICKET-STRING","cert_sha256":"..."}}
```

---

## **三、高级与实用技巧**

### **1. 批量操作脚本示例**

```bash
#!/bin/bash
# 批量启动 VM ID 100-105
for i in {100..105}; do
    echo "Starting VM $i..."
    qm start $i
done
```

```bash
# 批量设置所有运行中的 VM 启用 QEMU Agent
for vmid in $(qm list | awk '/running/ {print $1}'); do
    qm set $vmid --agent enabled=1
    echo "Enabled agent on VM $vmid"
done
```

### **2. 获取特定信息 (结合 grep/awk)**

```bash
# 获取所有运行中 VM 的 ID 和名称
qm list | grep running | awk '{print $1, $2}'

# 获取 VM 100 的 IP 地址 (需 qemu-guest-agent 启用)
qm agent 100 network-get-interfaces | grep -A 1 -B 2 "ip-address"
```

### **3. 使用 `qm guest` 命令 (需 qemu-guest-agent)**

```bash
# 在客户机内执行命令 (Linux 客户机)
qm guest exec 100 -- ls -l /tmp

# 获取客户机 IP
qm guest network-get-interfaces 100

# 请求客户机关机
qm guest shutdown 100

# 冻结/解冻文件系统 (用于备份一致性)
qm guest fsfreeze-status 100
qm guest fsfreeze-freeze 100
qm guest fsfreeze-thaw 100
```

---

## **四、重要注意事项**

1. **权限**: 大部分 `qm` 命令需要 `root` 权限或 `sudo`。
2. **配置文件位置**: VM 配置文件存储在 `/etc/pve/qemu-server/<vmid>.conf`。
3. **磁盘文件位置**: 取决于存储配置，通常在 `/var/lib/vz/images/<vmid>/` 或 LVM/ZFS 卷中。
4. **在线修改限制**: 部分配置（如 BIOS 类型、机器类型）需要关机后才能修改。
5. **备份首选 `vzdump`**: `qm backup` 已过时，生产环境请使用 `vzdump`。

---

## **五、总结**

`qm` 命令是 Proxmox VE 管理员的利器，掌握它能极大提升管理效率，尤其适合：

- 自动化部署和配置
- 批量管理大量虚拟机
- 编写监控和维护脚本
- 执行 Web 界面难以完成的精细操作

建议结合 `man qm` 查看最新官方文档，并在测试环境中熟练操作后再用于生产环境。

祝你玩转 Proxmox！ 🚀
