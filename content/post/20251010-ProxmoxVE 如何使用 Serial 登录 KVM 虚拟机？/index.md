---
title: ProxmoxVE 如何使用 Serial 登录 KVM 虚拟机？
description:
date: 2025-10-10
slug: proxmox-how-to-login-KVM-virtual-machine-using-Serial
image: https://ryan1998.dpdns.org/cover/post/proxmox/cover-proxmox-how-to-login-KVM-virtual-machine-using-Serial.png
categories:
    - Proxmox
    - KVM
tags:
    - Virtual Machine
    - Serial
---


#### noVNC Vs Serial

| 对比维度      | noVNC（默认图形控制台）                 | 串口登录（Serial Console）                              |
|:---------:|:------------------------------ | ------------------------------------------------- |
| **访问方式**  | 通过 Proxmox Web 界面嵌入的 VNC 客户端访问 | 通过 `qm terminal <VMID>` 命令或 Web 控制台中的“串口控制台”标签页访问 |
| **资源开销**  | 较高（需渲染图形界面，占用更多 CPU/内存/带宽）     | **极低**（仅传输文本，几乎无图形开销）                             |
| **适用场景**  | 有桌面环境的 VM、图形化安装、GUI 应用调试       | 最小化安装、救援模式、内核调试                                   |
| **配置复杂度** | **默认启用**，无需额外配置                | 需在 VM 中启用串口设备（如 ttyS0），并配置 getty 或 systemd 服务     |
| **复制粘贴**  | 无法访问系统粘贴板，不支持复制粘贴              | 依赖 ProxmoxVE 访问系统粘贴板，**支持复制粘贴**                   |
| **日志记录**  | 不便于自动记录控制台输出                   | 易于重定向或记录串口输出，**适合审计或调试**                          |

## 一、关机状态下添加硬件

虚拟机 → 硬件 → 添加 → 串行端口 → [输入串行端口号] → 添加

![](../../../../assets/2025-09-30-10-13-20-image.png)

---

## 二、开机修改 `GRUB` 配置

#### 1：启动虚拟机

虚拟机 → 控制台 → Start Now

![](../../../../assets/2025-09-30-11-01-30-image.png)

#### 2. 登录系统

![](../../../../assets/2025-09-30-15-15-29-image.png)

#### 3. 修改 `GRUB` 配置文件

`vi /etc/default/grub`

在 `grub` 配置文件末尾添加

```shell
# 将控制台输出同时发送到本地显示（tty0）和串口（ttyS0）
# 本地端口 console=tty0
# 串行端口 console=ttyS0,115200n8
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"
# 让 GRUB 菜单（启动选择界面）本身也输出到串行端口
GRUB_TERMINAL=serial
# 定义串行端口的具体通信参数
# 波特率 --speed=115200
# 串口号 --unit=0
# 数据位 --word=8
# 关闭奇偶校验 --parity=no
# 停止位 --stop=1
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
```

![](../../../../assets/2025-10-02-11-11-32-image.png)

#### 4. 重新生成系统启动 `GRUB` 配置文件

按系统执行以下命令，重新生成 `grub.cfg`

- Debian/Ubuntu

  ```shell
  # Debian/Ubuntu
  update-grub
  ```

  ![](../../../../assets/2025-10-02-11-15-26-image.png)

- RHEL/CentOS/Fedora/Rocky等

- ```shell
  grub2-mkconfig -o /boot/grub2/gurb.cfg
  ```

查看 `grub.cfg` 文件验证配置是否生效

```shell
grep "linux.*console" /boot/grub/grub.cfg
```

#### 5. 启用串行端口登录服务

```shell
sudo systemctl enable serial-getty@ttyS0.service
```

#### 6. 重启虚拟机

```shell
reboot
```

---

### 三、使用串口登录虚拟机

在 ProxmoxVE Shell 使用 `qm terminal <VMID>` 登录虚拟机

```bash
qm terminal 100
```

![](../../../../assets/2025-10-02-11-18-27-image.png)

1. 按下 `Enter` 出现登录提示 [Hostname] Login:

2. 输入用户名 `root`

3. 输入密码

4. 确保用户名、密码正确，按下 `Enter` 即可成功登录

5. 如需退出虚拟机，按 `Ctrl+O` 退出登录

![](../../../../assets/2025-10-02-11-24-54-image.png)

> 此文档由 [*杨文*](mailto:yangwen_job@163.com) 于 *2025年10月01日* 编写。
