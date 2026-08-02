# SG90 舵机控制笔记本电源按钮

## 接线

| SG90 舵机 | LubanCat-RV06 40pin | 说明 |
|-----------|---------------------|------|
| 橙线 (PWM信号) | Pin 12 (PWM7) | = pwmchip7/pwm0 |
| 红线 (5V电源) | Pin 2 | 5V 供电 |
| 棕线 (GND) | Pin 6 | 共地 |

## 我的参数

> ⚠️ 每个舵机有效范围不同，下表是我的实测值：

| 参数 | 值 | 说明 |
|------|-----|------|
| `--chip` | 7 | Pin12 = PWM7 → pwmchip7 |
| `--channel` | 0 | pwm0 |
| `--rest` | **3000000** (3.0ms) | 归位：舵机臂平着，远离按钮 |
| `--press` | **2650000** (2.65ms) | 按下：舵机臂顶下去，刚好按下按钮 |
| 有效范围 | 1200000 ~ 3000000 | 超出没反应 |

### One-KVM Web UI 配置

Settings → ATX → 驱动类型选 **"脚本命令"**：

| 输入框 | 命令 |
|--------|------|
| 电源按钮 | `/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 short` |
| 复位按钮 | `/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 --long-s 13 long` |

> ⚠️ 必须用完整路径 `/usr/local/bin/servo-ctrl`，不能用相对路径 `servo-ctrl`

## 调试流程（我的实际过程）

### 1. 找 PWM 芯片

```bash
ls /sys/class/pwm/
# 输出: pwmchip0  pwmchip7
```

Pin12 是 PWM7，对应 pwmchip7。

### 2. 导出 PWM 通道

```bash
echo 0 > /sys/class/pwm/pwmchip7/export
ls /sys/class/pwm/pwmchip7/    # 确认 pwm0 出现
#可以看到出现了两个PWM设备，分别是pwmchip0和pwmchip7，pwmchip0是屏幕背光，pwmchip7是30pin排针上的PWM接口。 我用的板子官方说明
```

### 3. 扫描有效脉宽范围

```bash
# 扫一圈看舵机在哪个范围有反应
while true; do
    for d in 500000 1000000 1500000 2000000 2500000 2000000 1500000 1000000; do
        echo $d > /sys/class/pwm/pwmchip7/pwm0/duty_cycle
        sleep 0.3
    done
done
# Ctrl+C 停止，观察有效范围
```

我的有效范围：**1200000 ~ 3000000**

- 小于 1200000：舵机不动
- 1200000 ~ 3000000：舵机转动
- 大于 3000000：舵机不动（已到极限）

### 4. 确定归位和按下值

```bash
# 逐步微调，找到刚好能按下按钮的值
echo 3000000 > /sys/class/pwm/pwmchip7/pwm0/duty_cycle   # 平着（归位）
echo 2650000 > /sys/class/pwm/pwmchip7/pwm0/duty_cycle   # 按下按钮
```

### 5. 测试短按

```bash
servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 init
servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 short
```

---

## 参数说明（通用）

| 参数 | 含义 | 何时修改 |
|------|------|---------|
| `--chip N` | PWM 芯片号 (`ls /sys/class/pwm/`) | 换引脚时 |
| `--channel N` | PWM 通道号 | 换引脚时 |
| `--rest N` | 归位脉宽 (ns)，舵机离开按钮 | 换了舵机/固定位置时 |
| `--press N` | 按下脉宽 (ns)，舵机按下按钮 | 换了舵机/固定位置时 |
| `--short-s 0.5` | 短按持续时间 (秒) | 按需 |
| `--long-s 5` | 长按持续时间 (秒) | 按需 |

---

## 命令行参考

```bash
# 初始化
/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 init

# 短按 / 长按
/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 short
/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 long
# 我的笔记本 5 秒不关机，手动改成 13 秒
/usr/local/bin/servo-ctrl --chip 7 --channel 0 --rest 3000000 --press 2650000 --long-s 13 long

# 直接写 sysfs 手动调角度（精度 1ns，任意值）
echo 2700000 > /sys/class/pwm/pwmchip7/pwm0/duty_cycle
echo 3000000 > /sys/class/pwm/pwmchip7/pwm0/duty_cycle
```

## 40pin PWM 引脚参考 (LubanCat-RV06)

| 物理引脚 | 功能 | 实测 PWM 路径 |
|----------|------|--------------|
| Pin 12 | PWM7 | pwmchip7/pwm0 |
