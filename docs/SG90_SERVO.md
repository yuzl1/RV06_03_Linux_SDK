# SG90 舵机控制笔记本电源按钮

## 接线

| SG90 舵机 | LubanCat-RV06 40pin | 说明 |
|-----------|---------------------|------|
| 橙线 (PWM信号) | Pin 12 (PWM7) | 任意 PWM 引脚均可，调试时改参数 |
| 红线 (5V电源) | Pin 2 或 4 | 固定 5V 输出 |
| 棕线 (GND) | Pin 6/9/14/20/25/30 | 任选一个 GND |

> 换引脚？改 `--chip` / `--channel` 参数就行，见下文。

## 使用

### 1. 命令行调试

```bash
# 初始化 PWM
servo-ctrl --chip 0 --channel 0 --rest 500000 --press 2000000 init

# 测试单个角度 (脉宽单位: 纳秒)
servo-ctrl --chip 0 --channel 0 --rest 500000 --press 2000000 test 1500000

# 短按 (0.5秒)
servo-ctrl --chip 0 --channel 0 --rest 500000 --press 2000000 short

# 长按 (5秒, 强制关机)
servo-ctrl --chip 0 --channel 0 --rest 500000 --press 2000000 long

# 查看当前状态
servo-ctrl --chip 0 --channel 0 status
```

### 2. One-KVM Web UI 配置

Settings → ATX → 驱动类型选 **"脚本命令"** → 设备填：

```
/usr/local/bin/servo-ctrl --chip 0 --channel 0 --rest 500000 --press 2000000
```

保存后在 Web UI 电源按钮区域点击短按/长按。

## 参数说明

| 参数 | 默认值 | 含义 | 何时修改 |
|------|--------|------|---------|
| `--chip 0` | 0 | PWM 芯片号 (`ls /sys/class/pwm/`) | 换引脚时 |
| `--channel 0` | 0 | PWM 通道号 | 换引脚时 |
| `--period 20000000` | 20ms (50Hz) | PWM 周期, SG90 固定 | 不修改 |
| `--rest 500000` | 0.5ms (0°) | 归位脉宽, 舵机离开按钮 | 角度不对时微调 |
| `--press 2000000` | 2.0ms (~90°) | 按下脉宽, 舵机按下按钮 | 角度不对时微调 |
| `--short-s 0.5` | 0.5 秒 | 短按持续时间 | 按需调整 |
| `--long-s 5` | 5 秒 | 长按持续时间 | 按需调整 |

### 角度微调参考

SG90 脉宽与角度对应关系 (50Hz):

| 脉宽 (ns) | 角度 |
|-----------|------|
| 500000 | 0° (远离按钮) |
| 1000000 | ~22° |
| 1500000 | 45° (中间) |
| 2000000 | ~90° (按下按钮) |
| 2500000 | ~135° (超出范围) |

> `--rest` 建议 500000~800000，`--press` 建议 1500000~2200000，根据实际机械位置微调。

## 调试流程

```
1. servo-ctrl ... test 500000    → 确认归位角度, 舵机臂不碰按钮
2. servo-ctrl ... test 1500000   → 确认中间位置
3. servo-ctrl ... test 2000000   → 确认按下角度, 刚好按下但不大力
4. 微调 --rest / --press 直到合适
5. servo-ctrl ... short          → 测试完整按一次
6. 把调试好的参数填入 One-KVM Web UI
```

## 40pin PWM 引脚参考 (LubanCat-RV06)

| 物理引脚 | 功能 | 内核 PWM 路径 (参考) |
|----------|------|---------------------|
| Pin 12 | PWM7 | pwmchip0/pwm0 |
| Pin 32 | PWM0 | 需实测 |

> 板上执行 `ls /sys/class/pwm/` 查看实际可用的 PWM 控制器。导出新通道: `echo 0 > /sys/class/pwm/pwmchip0/export`
