# LubanCat-RV06 + One-KVM 集成镜像

> 野火 LubanCat-RV06 (RV1106) 定制 Buildroot 镜像，内置 One-KVM IP-KVM 服务，烧录即用。

## 功能

| 功能 | 说明 |
|------|------|
| One-KVM | IP-KVM Web UI，USB 采集卡画面 + 远程键盘鼠标 |
| ttyd | Web 终端 (端口 7681)，浏览器直接操作 shell |
| SG90 舵机 | ATX Script 驱动，远程控制笔记本电源按钮 |
| 固定 MAC | `42:83:de:86:af:fd`，刷机 IP 不变 |

## 快速开始

### 1. 获取镜像

从 GitHub Actions 或 Releases 下载：

> https://github.com/yuzl1/RV06_03_Linux_SDK/actions → 最新构建 → `lubancat-rv06-sd-image`

或 [Releases](https://github.com/yuzl1/RV06_03_Linux_SDK/releases) 下载永久版本。

### 2. 烧录

**SPI NAND 烧录**

用瑞芯微工具（RKDevTool 或 upgrade_tool）直接烧 `update.img` 一键烧录。

**SD 卡烧录**

将以下文件分别烧录到对应分区（用瑞芯微工具选择对应位置）：

| 文件 | 分区 |
|------|------|
| `idblock.img` | ID Block |
| `uboot.img` | U-Boot |
| `env.img` | Env |
| `boot.img` | Boot |
| `rootfs.img` | Rootfs |

### 3. 使用

| 服务 | 地址 |
|------|------|
| One-KVM | `http://<板子IP>:8080` |
| ttyd 终端 | `http://<板子IP>:7681` (root/root) |
| SSH | `ssh root@<板子IP>` (无密码) |

## SG90 舵机配置

详见 [docs/SG90_SERVO.md](docs/SG90_SERVO.md)（sg90-atx 分支）。

One-KVM Web UI → Settings → ATX → 驱动类型选"脚本命令"：

| 输入框 | 命令 |
|--------|------|
| 电源按钮 | `/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 short` |
| 复位按钮 | `/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 --long-s 13 long` |

## 管理

```bash
/etc/init.d/S85onekvm start|stop|restart   # One-KVM 服务
/etc/init.d/S86ttyd start|stop|restart     # ttyd 终端
cat /var/log/one-kvm.log                    # 日志
```

## 分支

| 分支 | 说明 |
|------|------|
| `main` | 稳定版（ttyd + One-KVM 基础集成） |
| `sg90-atx` | SG90 舵机 + ATX Script 驱动 |

## 编译

```bash
# GitHub Actions 自动编译，本地编译需要 Linux + Buildroot SDK
./build.sh
# 产物在 output/image/
```

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [yuzl1/One-KVM](https://github.com/yuzl1/One-KVM) | One-KVM ARMv7 + ATX Script 后端 |
| [LubanCat/RV06_03_Linux_SDK](https://github.com/LubanCat/RV06_03_Linux_SDK) | 上游 SDK |
