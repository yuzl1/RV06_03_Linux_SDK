# LubanCat-RV06 + One-KVM 集成镜像

> 野火 LubanCat-RV06 (RV1106) Buildroot 镜像，内置 One-KVM IP-KVM 服务，烧录即用。

## 快速开始

### 1. 下载镜像

从 GitHub Actions 下载最新构建产物：

> https://github.com/yuzl1/RV06_03_Linux_SDK/actions → 最新成功运行 → Artifacts → `lubancat-rv06-sd-image`

解压后获取：
```
image/
├── boot.img       kernel + device tree
├── env.img        uboot 环境变量
├── idblock.img    ID block
├── rootfs.img     Buildroot 根文件系统 (含 One-KVM)
├── uboot.img      U-Boot
└── update.img     完整固件 (一键烧录)
```

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

板子上电后，One-KVM 自动启动。浏览器访问：

```
http://<板子IP>:8080
```

默认无用户，首次访问在 Web UI 中创建管理员账号。

---

## 板子信息

| 项目 | 详情 |
|------|------|
| **型号** | LubanCat-RV06 |
| **SoC** | Rockchip RV1106 (ARMv7-A) |
| **系统** | Buildroot 2024.02.10 |
| **默认密码** | root (无密码，SSH 可登录) |
| **One-KVM 端口** | 8080 |
| **MAC 地址** | 固定为 `42:83:de:86:af:fd` |

---

## 自启动服务

```
/etc/init.d/
├── S38fixmac    ← 固定 eth0 MAC (网络启动前)
├── S40network   ← 网络配置
├── S50sshd      ← SSH 服务
└── S85onekvm    ← One-KVM (网络就绪后)
```

### 管理 One-KVM

```bash
/etc/init.d/S85onekvm start     # 启动
/etc/init.d/S85onekvm stop      # 停止
/etc/init.d/S85onekvm restart   # 重启
```

### 查看日志

```bash
cat /var/log/one-kvm.log
```

---

## 文件布局

```
/usr/bin/one-kvm                     ← One-KVM 主程序
/usr/lib/one-kvm-libs/               ← glibc 运行时库 (板子用 uClibc)
/usr/lib/one-kvm-libs/ld-linux-armhf.so.3  ← glibc 动态链接器
/etc/ssl/certs/ca-certificates.crt   ← TLS CA 证书
/etc/init.d/S85onekvm                ← 启动脚本
/etc/init.d/S38fixmac                ← MAC 固定脚本
/var/lib/one-kvm/                    ← 数据目录 (数据库+配置)
```

---

## CI/CD 自动构建

推送代码到 main 分支后，GitHub Actions 自动：

```
Job 1: 编译 One-KVM (armv7) + 提取 glibc 运行时库 (~30min)
Job 2: 集成到 Buildroot overlay → 编译完整 SD 卡镜像 (~2h)
产物: lubancat-rv06-sd-image.zip
```

---

## 自定义

### 修改 One-KVM 版本

更新 `.github/workflows/build-sd-image.yml` 中 clone 的 repo 地址或分支。

### 修改固定 MAC

编辑 `sysdrv/tools/board/buildroot/overlay/etc/init.d/S38fixmac` 中的 `FIXED_MAC`。

### 修改 One-KVM 端口

编辑 `sysdrv/tools/board/buildroot/overlay/etc/init.d/S85onekvm` 中的 `-p 8080`。

### 修改 Wi-Fi / USB 设备配置

编辑板卡配置：
```
project/cfg/BoardConfig_IPC/BoardConfig-SD_CARD-NONE-RV1106_LubanCat-RV06.mk
```

---

## SG90 舵机 + ATX Script 驱动 (sg90-atx 分支)

### 接线

| SG90 | RV1106 40pin |
|------|-------------|
| 橙线(PWM) | Pin 12 (PWM7 = pwmchip7/pwm0) |
| 红线(5V) | Pin 2 |
| 棕线(GND) | Pin 6 |

### Web UI 配置

Settings → ATX → 驱动类型选"脚本命令"：

| 输入框 | 命令 |
|--------|------|
| 电源按钮 | `/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 short` |
| 复位按钮 | `/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 --long-s 13 long` |

### 调试

```bash
# 手动测试
/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 short
/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 --long-s 13 long

# 直接调 PWM
echo 3000000 > /sys/class/pwm/pwmchip7/pwm0/duty_cycle   # 归位
echo 2650000 > /sys/class/pwm/pwmchip7/pwm0/duty_cycle   # 按下
```

---

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [yuzl1/One-KVM](https://github.com/yuzl1/One-KVM) | One-KVM ARMv7 编译 |
| [LubanCat/RV06_03_Linux_SDK](https://github.com/LubanCat/RV06_03_Linux_SDK) | 上游 SDK |
| [mofeng-git/One-KVM](https://github.com/mofeng-git/One-KVM) | One-KVM 上游 |
