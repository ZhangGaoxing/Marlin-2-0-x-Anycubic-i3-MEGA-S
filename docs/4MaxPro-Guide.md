# Anycubic 4Max Pro 固件适配指南

本文档介绍如何在 knutwurst 的 Marlin 2.1.x 多机型固件中为 **Anycubic 4Max Pro（原版）** 编译并刷入固件。

---

## 目录

- [机型说明](#机型说明)
- [硬件参数](#硬件参数)
- [构建环境列表](#构建环境列表)
- [编译前准备](#编译前准备)
- [编译方法](#编译方法)
- [刷机方法](#刷机方法)
- [参数与 4Max Pro 2.0 的差异](#参数与-4max-pro-20-的差异)
- [常见问题](#常见问题)

---

## 机型说明

本固件支持以下 Anycubic 4Max Pro 系列：

| 编译标识 | 机型 |
|---|---|
| `KNUTWURST_4MAXP` | Anycubic 4Max Pro（**原版**，本文档描述的机型） |
| `KNUTWURST_4MAXP2` | Anycubic 4Max Pro 2.0 |

两者使用相同的主板（`BOARD_TRIGORILLA_14`，ATmega2560），但硬件参数存在差异，请务必选择正确的构建环境。

---

## 硬件参数

以下为适配原版 4Max Pro 所使用的硬件配置：

| 参数 | 值 |
|---|---|
| 主板 | BOARD_TRIGORILLA_14（ATmega2560） |
| 热端热敏电阻 | `5`（ATC Semitec 104GT-2） |
| 热床热敏电阻 | `5`（ATC Semitec 104GT-2） |
| 热端 PID（KP/KI/KD） | 18.53 / 1.27 / 67.55 |
| 热床 PID（KP/KI/KD） | 100.68 / 17.07 / 395.86 |
| X/Y/Z/E 步进（steps/mm） | 100 / 80 / 800 / 418 |
| 最大进给速度（X/Y/Z/E mm/s） | 150 / 150 / 20 / 80 |
| 最大加速度（X/Y/Z/E mm/s²） | 700 / 700 / 70 / 15000 |
| 打印/回抽/移动加速度（mm/s²） | 700 / 700 / 700 |
| Jerk（X/Y/Z） | 8.2 / 8.2 / 0.2 |
| 打印床尺寸（X×Y） | 270 × 205 mm |
| Z 轴最大高度 | 205 mm |
| X 轴最小位置 | -8 mm |
| Z 轴方向 | 正转（`INVERT_Z_DIR true`） |
| E0 方向 | `INVERT_E0_DIR false` |
| 触摸屏协议 | Anycubic TFT（LCD_SERIAL_PORT 3） |
| 热床调平方式 | 手动网格调平（5×5 点阵） |

---

## 构建环境列表

根据您的硬件配置选择对应的编译目标：

| 环境名称 | 描述 |
|---|---|
| `4MAXP` | 标准版，无 TMC 驱动，无 BLTouch |
| `4MAXP_TMC` | 安装了 TMC2208 静音驱动 |
| `4MAXP_BLT_10` | 安装了 BLTouch，Trigorilla v1.0 主板 |
| `4MAXP_BLT_11` | 安装了 BLTouch，Trigorilla v1.1 主板 |
| `4MAXP_TMC_BLT_10` | TMC 驱动 + BLTouch，Trigorilla v1.0 |
| `4MAXP_TMC_BLT_11` | TMC 驱动 + BLTouch，Trigorilla v1.1 |
| `4MAXP_DGUS` | 使用 DGUS2 新款触摸屏，无 TMC，无 BLTouch |
| `4MAXP_DGUS_TMC` | DGUS2 屏 + TMC 驱动 |
| `4MAXP_DGUS_BLT_10` | DGUS2 屏 + BLTouch，Trigorilla v1.0 |
| `4MAXP_DGUS_BLT_11` | DGUS2 屏 + BLTouch，Trigorilla v1.1 |
| `4MAXP_DGUS_TMC_BLT_10` | DGUS2 屏 + TMC + BLTouch，Trigorilla v1.0 |
| `4MAXP_DGUS_TMC_BLT_11` | DGUS2 屏 + TMC + BLTouch，Trigorilla v1.1 |

> **如何判断主板版本？** 查看 Trigorilla 主板上的丝印标识，v1.0 标注 `V1.0`，v1.1 标注 `V1.1`。

---

## 编译前准备

### 1. 安装 Python 和 PlatformIO

```bash
# 安装 PlatformIO CLI
pip install platformio
```

或安装 **Visual Studio Code** 并在扩展商店中搜索安装 **PlatformIO IDE** 插件。

### 2. 克隆/下载本仓库

```bash
git clone https://github.com/knutwurst/Marlin-2-0-x-Anycubic-i3-MEGA-S.git
cd Marlin-2-0-x-Anycubic-i3-MEGA-S
```

### 3. 确认目录结构

编译所需的配置文件均已就位：

```
Marlin/
  Configuration.h          ← 主配置（含 4MAXP 参数）
  Configuration_adv.h      ← 高级配置
  src/pins/pins.h          ← 引脚/环境映射
ini/
  i3_mega.ini              ← 构建环境定义（含 4MAXP 系列）
platformio.ini             ← PlatformIO 主配置
```

---

## 编译方法

### 命令行编译

在项目根目录下运行：

```bash
# 编译标准版（推荐首次使用）
pio run -e 4MAXP

# 编译 TMC 驱动版
pio run -e 4MAXP_TMC

# 编译 BLTouch 版（Trigorilla v1.0 主板）
pio run -e 4MAXP_BLT_10

# 编译所有 4MAXP 系列
pio run -e 4MAXP -e 4MAXP_TMC -e 4MAXP_BLT_10 -e 4MAXP_BLT_11
```

### VS Code + PlatformIO IDE 编译

1. 用 VS Code 打开项目文件夹
2. 在左侧打开 **PlatformIO** 面板
3. 展开 **Project Tasks**，找到目标环境（如 `4MAXP`）
4. 点击 **Build**

### 编译输出

成功编译后，固件文件位于：

```
.pio/build/4MAXP/firmware.hex
```

---

## 刷机方法

### 方法一：通过 USB 直接上传（需要连接打印机）

```bash
pio run -e 4MAXP -t upload
```

### 方法二：使用 SD 卡刷机

1. 将编译好的 `firmware.hex` 复制到 SD 卡根目录，重命名为 `firmware.hex`
2. 将 SD 卡插入打印机
3. 重启打印机，等待自动刷写（屏幕会显示进度）
4. 刷写完成后打印机自动重启

### 方法三：使用 Arduino IDE 上传

1. 安装 Arduino IDE 并添加 ATmega2560 支持
2. 打开 `Marlin/Marlin.ino`
3. 选择开发板：**Arduino Mega 2560**
4. 选择正确的串口，点击上传

> ⚠️ **注意**：刷机前建议通过打印机菜单执行 **「初始化 EEPROM」**（或发送 `M502` 然后 `M500`），避免旧设置冲突。

---

## 参数与 4Max Pro 2.0 的差异

原版 4Max Pro 与 4Max Pro 2.0 的主要配置差异如下：

| 参数 | 4Max Pro（原版）| 4Max Pro 2.0 |
|---|---|---|
| 热端热敏电阻类型 | `5`（ATC Semitec 104GT-2）| `11`（Wanhao/Keenovo Beta 3950）|
| 热端 PID（KP） | 18.53 | 17.13 |
| 热端 PID（KI） | 1.27 | 0.93 |
| 热端 PID（KD） | 67.55 | 78.58 |
| 热床 PID（KP） | 100.68 | 251.78 |
| 热床 PID（KI） | 17.07 | 49.57 |
| 热床 PID（KD） | 395.86 | 319.73 |
| 挤出机步进（steps/mm）| 418 | 415 |
| Z 最大进给速度（mm/s）| 20 | 18 |
| 最大加速度（X/Y mm/s²）| 700 | 1500 |
| 打印加速度（mm/s²）| 700 | 800 |
| Z 轴方向 | `true`（正转）| `false`（反转）|
| E0 方向 | `false` | `true` |
| 打印床 Y 轴尺寸 | 205 mm | 210 mm |
| Z 轴最大高度 | 205 mm | 190 mm |

> ⚠️ **使用错误环境时的风险**：若将 4Max Pro 2.0 的固件刷入原版 4Max Pro，Z 轴方向相反会导致归零时撞床损坏硬件，请务必核对机型。

---

## 常见问题

### Q: 刷入固件后 Z 轴归零方向反了

确认您使用的是 `4MAXP`（原版）而非 `4MAXP2`（2.0 版）。原版 4Max Pro 的 `INVERT_Z_DIR` 为 `true`，两者相反。

### Q: 挤出机温度读数异常

确认热端热敏电阻类型。原版 4Max Pro 使用 **ATC Semitec 104GT-2**（类型 `5`），若实际热敏电阻不同，需在 `Configuration.h` 中修改 `TEMP_SENSOR_0` 的值后重新编译。

### Q: 如何重新校准 PID

通过屏幕菜单或发送 G-code 执行自动 PID 调谐：

```gcode
; 热端 PID 调谐（目标温度 200°C，8 次循环）
M303 E0 S200 C8 U1

; 热床 PID 调谐（目标温度 60°C，8 次循环）
M303 E-1 S60 C8 U1

; 保存结果到 EEPROM
M500
```

### Q: 手动网格调平如何使用

1. 打印机菜单 → **调平** → **网格调平**
2. 按屏幕提示依次调整 5×5 共 25 个点的 Z 高度
3. 调平完成后保存（`M500`）

### Q: 编译报错 `Build environment is incompatible`

这是预检脚本检查到环境名未在 `pins.h` 的允许列表中。确认 `Marlin/src/pins/pins.h` 第 173 行的环境列表包含 `env:4MAXP` 等所有 4MAXP 系列环境。

### Q: 如何判断主板是 Trigorilla v1.0 还是 v1.1

查看主板上靠近电源接口处的丝印文字。v1.1 主板使用不同的 BLTouch 接线引脚，若接错版本固件 BLTouch 将无法工作。

---

## 相关链接

- [原项目 GitHub（knutwurst）](https://github.com/knutwurst/Marlin-2-0-x-Anycubic-i3-MEGA-S)
- [参考项目：ZhangGaoxing 4Max Pro 固件](https://github.com/ZhangGaoxing/Marlin-A4MaxPro-2.0.x)
- [Marlin 官方文档](https://marlinfw.org/docs/configuration/configuration.html)
- [PlatformIO 安装指南](https://platformio.org/install/cli)
