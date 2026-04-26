# Anycubic 4Max Pro 固件适配指南

本文档介绍如何在 knutwurst 的 Marlin 2.1.x 多机型固件中为 **Anycubic 4Max Pro（原版）** 编译并刷入固件。

---

## 目录

- [机型说明](#机型说明)
- [固件特性](#固件特性)
- [硬件参数](#硬件参数)
- [最近配置变更](#最近配置变更)
- [构建环境列表](#构建环境列表)
- [编译前准备](#编译前准备)
- [编译方法](#编译方法)
- [刷机方法](#刷机方法)
- [刷写后必做操作](#刷写后必做操作)
- [校准步骤](#校准步骤)
- [预热温度调整](#预热温度调整)
- [Cura 配置](#cura-配置)
- [起始 G-code](#起始-g-code)
- [结束 G-code](#结束-g-code)
- [参数与 4Max Pro 2.0 的差异](#参数与-4max-pro-20-的差异)
- [常见问题](#常见问题)
- [相关链接](#相关链接)

---

## 机型说明

本固件支持以下 Anycubic 4Max Pro 系列：

| 编译标识 | 机型 |
|---|---|
| `KNUTWURST_4MAXP` | Anycubic 4Max Pro（**原版**，本文档描述的机型） |
| `KNUTWURST_4MAXP2` | Anycubic 4Max Pro 2.0 |

两者使用相同的主板（`BOARD_TRIGORILLA_14`，ATmega2560），但硬件参数存在差异，请务必选择正确的构建环境。

---

## 固件特性

本固件基于 Marlin 2.1.2.7，相比出厂原版固件新增/改进以下功能：

| 功能 | 说明 |
|---|---|
| **Marlin 2.1.x** | 从原厂版本大幅升级，稳定性与功能全面提升 |
| **修复热敏电阻配置** | 原厂热端温度传感器类型配置有误，已修正为 `TEMP_SENSOR_0 = 5` |
| **TMC2208 静音驱动支持** | 可选 `4MAXP_TMC` 环境，X/Y/Z 三轴静音，打印噪音大幅降低 |
| **网格调平 (MBL)** | 25 点手动网格调平，补偿不平整热床 |
| **Linear Advance** | 改善拐角溢料/欠料，提升过渡段打印质量 |
| **断料检测** | 耗材耗尽自动暂停，换料后继续打印 |
| **Baby Stepping** | 支持打印中实时调整 Z 偏移 |
| **PID 精确温控** | 热端与热床均采用 PID 精确温控 |
| **电源控制（PSU_CONTROL）** | 通过 PS_ON 引脚控制打印机电源，支持屏幕软关机 |
| **限位蜂鸣（ENDSTOP_BEEP）** | 触碰限位开关时发出短促蜂鸣提示 |
| **EEPROM 存储** | 所有参数可使用 `M500` 持久化保存 |
| **S-Curve 加速** | 更平滑的速度曲线，减少机械振动 |
| **热保护（Thermal Runaway）** | 热端温度异常时自动停机，防止安全事故 |

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

## 最近配置变更

以下是对 `Marlin/Configuration.h` 的最新修正，确保与 4Max Pro 硬件行为完全一致：

### 1. 启用电源控制（PSU_CONTROL）

**影响机型**：原版 4Max Pro（`KNUTWURST_4MAXP`）和 4Max Pro 2.0（`KNUTWURST_4MAXP2`）  
**变更前**：`//#define PSU_CONTROL`（注释禁用）  
**变更后**：对 4MAXP 系列条件启用

Anycubic 4Max Pro 使用非标准 ATX 电源，通过 PS_ON 引脚以 **Active HIGH** 信号控制电源，启用此选项后可通过打印机屏幕执行软关机操作。

> ⚠️ 同时将 `PSU_ACTIVE_STATE` 从标准 ATX 的 `LOW` 修正为适配本机型的 `HIGH`，极性相反会导致电源无法正常控制。

### 2. 启用限位蜂鸣（ENDSTOP_BEEP）

**变更前**：`// #define ENDSTOP_BEEP`（注释禁用）  
**变更后**：`#define ENDSTOP_BEEP`（启用）

归零过程中触碰限位开关时，蜂鸣器发出短促提示音，方便判断归零是否正常执行。

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

> ⚠️ **注意**：刷机完成后务必执行下方「[刷写后必做操作](#刷写后必做操作)」，避免旧参数冲突。

---

## 刷写后必做操作

### 首次刷写（或 EEPROM 结构发生变化时）

通过串口工具（波特率 **250000**，推荐 Pronterface 或 OctoPrint）或打印机屏幕终端连接后，依次发送：

```gcode
M502    ; 重置所有参数为固件默认值（清除旧 EEPROM 数据）
M500    ; 保存到 EEPROM
```

**完成后断电重启**。

> ⚠️ `M502` 会清除 EEPROM 中 **所有** 校准数据（PID、网格调平、步进值等）。只在首次刷机或遇到 EEPROM 版本不兼容时执行。

---

### 后续更新固件（已完成过校准，不想重新校准）

**刷机前**，先保存当前校准数据：

```gcode
M503    ; 列出所有当前参数（将输出内容复制保存备用）
```

刷机完成后，**不要执行** `M502`，直接验证 EEPROM 是否完整：

```gcode
M503    ; 查看参数是否保留
```

若 `M503` 输出的 PID、网格调平、步进值与刷机前一致，说明 EEPROM 完整保留，无需任何操作。

若 Marlin 检测到 EEPROM 版本变化（串口会输出 `EEPROM version mismatch` 警告），则需要重新校准，或从 `M503` 的备份数据手动恢复：

```gcode
; --- 1. 恢复出厂设置 ---
M502

; --- 2. 导入热端与热床 PID 参数 ---
M301 P23.60 I1.62 D86.02    ; 热端 PID
M304 P229.13 I44.40 D788.36 ; 热床 PID

; --- 3. 导入 5x5 网床调平数据 ---
G29 S3 I0 J0 Z-0.15000
G29 S3 I1 J0 Z0.05000
G29 S3 I2 J0 Z0.19000
G29 S3 I3 J0 Z0.28000
G29 S3 I4 J0 Z0.32000
G29 S3 I0 J1 Z-0.13000
G29 S3 I1 J1 Z0.03000
G29 S3 I2 J1 Z0.14000
G29 S3 I3 J1 Z0.20000
G29 S3 I4 J1 Z0.26000
G29 S3 I0 J2 Z-0.10000
G29 S3 I1 J2 Z0.00000
G29 S3 I2 J2 Z0.08000
G29 S3 I3 J2 Z0.10000
G29 S3 I4 J2 Z0.15000
G29 S3 I0 J3 Z-0.08000
G29 S3 I1 J3 Z-0.01000
G29 S3 I2 J3 Z0.05000
G29 S3 I3 J3 Z0.07000
G29 S3 I4 J3 Z0.05000
G29 S3 I0 J4 Z-0.05000
G29 S3 I1 J4 Z0.02000
G29 S3 I2 J4 Z0.07000
G29 S3 I3 J4 Z0.06000
G29 S3 I4 J4 Z0.04000

; --- 3. K 值校准 ---
M900 K0.15

; --- 4. 保存到 EEPROM (必须执行，否则重启失效) ---
M500

; --- 5. 启用调平并检查状态 ---
M420 S1
M420 V
```

**完成后断电重启**。

---

## 校准步骤

### 1. 热端 PID 自动校准（推荐）

```gcode
M106 S204           ; 开启冷却风扇，模拟实际打印状态
M303 E0 S205 C8 U1  ; 在 205°C 自动校准 8 轮，完成后自动应用
M500                ; 保存结果
```

热床 PID（如热床温度不稳定时执行）：

```gcode
M303 E-1 S60 C6 U1  ; 热床 60°C 校准
M500
```

### 2. 挤出机步进校准（推荐）

当前固件默认 E 步数：**418 步/mm**

校准方法：

1. 在挤出机进料管入口处标记 **120 mm** 处
2. 加热热端到打印温度（约 200°C），然后发送：
   ```gcode
   M83          ; 相对模式
   G1 E100 F100 ; 挤出 100 mm
   ```
3. 量取剩余长度，应为 20 mm。按以下公式计算新步数：

$$E_{新} = E_{旧} \times \frac{100}{100 - (实际剩余 - 20)}$$

4. 应用并保存：
   ```gcode
   M92 E<新数值>
   M500
   ```

### 3. 网格床面调平（推荐）

1. 打印机屏幕：**Extra Menu → Start Mesh Leveling**
2. 共 25 个点，每点用 A4 纸张法调整高度（纸张能感受轻微阻力即可）
3. 全部完成后选择 **Save EEPROM**

验证调平是否已保存：

```gcode
M420 V    ; 查看当前网格数据
```

### 4. Linear Advance K 值校准（推荐）

#### 固件配置确认

本固件已在 `Configuration_adv.h` 中启用 Linear Advance 1.5，初始 K 值为 `0.0`（等效关闭）：

```c
#define LIN_ADVANCE
#define ADVANCE_K 0.0
```

**注意**：起始 G-code 中的 `M900 K0.2` 是启动时临时写入的 K 值，每次打印前都会执行，无需修改固件。

---

#### 使用在线工具生成校准 G-code

访问 [https://marlinfw.org/tools/lin_advance/k-factor.html](https://marlinfw.org/tools/lin_advance/k-factor.html)，按以下参数填写：

**Printer Info（打印机信息）**

| 参数 | 4Max Pro（原版）| 说明 |
|---|---|---|
| Filament Diameter | `1.75` | 耗材直径 |
| Nozzle Diameter | `0.4` | 标准喷嘴 |
| Nozzle Temperature | `205` | PLA 推荐温度 |
| Bed Temperature | `60` | 热床温度 |
| Retraction Distance | `1.0` | 回抽距离（mm）|
| Layer Height | `0.2` | 层高（mm）|
| Fan Speed | `100` | 风扇百分比 |

**Print Bed（打印床）**

| 参数 | 值 |
|---|---|
| Bed Shape | `Rectangular` |
| Bed Size X | `270` |
| Bed Size Y | `205` |
| Origin Bed Center | 不勾选 |

**Speed（速度）**

| 参数 | 值 | 说明 |
|---|---|---|
| Use Mm/S | 勾选 | 使用 mm/s 单位 |
| Slow Printing Speed | `20` | 慢速段（蓝色线）|
| Fast Printing Speed | `70` | 快速段（红色线），需与慢速差异显著 |
| Movement Speed | `120` | 移动速度 |
| Retract Speed | `35` | 回抽速度 |
| Unretract Speed | `35` | 送丝速度 |
| Acceleration | `700` | 对应固件 `DEFAULT_ACCELERATION` |
| Jerk X/Y | `-1` | 使用固件默认值（8.2 mm/s）|

**Pattern（测试图案）**

| 参数 | 值 | 说明 |
|---|---|---|
| Lin Advance Version | `1.5` | Marlin 1.1.9 / 2.0 及以上使用 1.5 |
| Pattern Type | `Standard` | 标准图案 |
| Starting Value For K | `0` | K 值起始 |
| Ending Value For K | `1.0` | K 值终止（PLA 通常不超过 0.5）|
| K-Factor Stepping | `0.05` | 每行增量（0 到 1.0，共 20 行）|
| Slow Speed Length | `20` | 慢速段长度（mm）|
| Fast Speed Length | `40` | 快速段长度（mm）|
| Test Line Spacing | `5` | 行间距（mm）|
| Print Anchor Frame | 勾选 | 增强首尾附着力 |
| Line Numbering | 勾选 | 在每两行旁打印 K 值标注 |

**Advanced（高级）**

| 参数 | 值 |
|---|---|
| Use Bed Leveling | `No`（若已有网格数据可选 `Yes`）|
| Prime Nozzle | 勾选 |

点击 **Generate G-code** → **Save** 下载 `.gcode` 文件。

---

#### 打印与读取结果

1. 将生成的 G-code 文件复制到 SD 卡，打印
2. 观察测试图案：
   - **蓝色线**（慢速段）为参考基准
   - **红色线**（快速段）的端点是观察目标
   - 找到快速段**宽度与慢速段最接近**、拐角处**无溢料（bulge）也无缩料（gap）**的行
3. 读取该行旁边的 K 值标注（若启用了 Line Numbering），或通过行号推算：

$$K = K_{start} + \text{行号} \times K_{step}$$

---

#### 应用 K 值

通过串口或打印机屏幕终端发送：

```gcode
M900 K0.2   ; 替换 0.2 为实测最佳 K 值
M500        ; 保存到 EEPROM
```

永久保存后，起始 G-code 中的 `M900 K0.2` 也应更新为相同值，以保持一致：

```gcode
M900 K<实测值>   ; Linear Advance K 值（已替换为实际校准结果）
```

---

#### 各材料参考 K 值范围

| 耗材 | K 值范围 | 备注 |
|---|---|---|
| PLA | 0.1 ~ 0.3 | 直接驱动常见值约 0.1–0.2 |
| PETG | 0.2 ~ 0.5 | 流动性好，K 值偏高 |
| ABS | 0.1 ~ 0.3 | 与 PLA 相近 |
| TPU | 0 | 建议关闭（`M900 K0`），柔性耗材不适用 |

> ⚠️ **注意事项**：
> - 换用不同品牌/颜色耗材、更换喷嘴尺寸、更改打印温度后，需重新校准
> - 切片软件中的 **Coasting（滑行）** 与 Linear Advance 功能冲突，两者只能选一，Cura 中应禁用 Coasting
> - K 值对直接驱动（Direct Drive）通常在 0 ~ 0.5；如改为鲍登管（Bowden）挤出则需大幅提高

### 5. Z 轴偏移微调

```gcode
G28          ; 归零
M851 Z0      ; 清零偏移
G1 Z0 F300   ; 移到 Z=0
; 纸张测试，按实际情况调整偏移值
M851 Z-0.1   ; 示例：负值降低喷嘴，正值抬高喷嘴
M500
G28          ; 重新归零使偏移生效
```

---

## 预热温度调整

固件默认预热温度：PLA 热端 200°C / 热床 60°C，ABS 热端 240°C / 热床 90°C。可通过以下任一方式修改：

### 方法一：通过打印机屏幕设置（永久保存）

1. 屏幕主界面 → **控制（Control）** → **温度（Temperature）**
2. 选择 **预热 PLA 设置（Preheat PLA conf）**：
   - 设置热端温度（Hotend）
   - 设置热床温度（Bed）
   - 设置风扇速度（Fan speed，0–255）
3. 对 **预热 ABS 设置（Preheat ABS conf）** 重复相同操作
4. 返回 **控制（Control）** → **保存设置（Store settings）**，或发送：
   ```gcode
   M500
   ```

### 方法二：通过 G-code 指令设置（需串口工具）

```gcode
; M145：设置预热配置（S0 = 预热1/PLA，S1 = 预热2/ABS）
M145 S0 H200 B60  F0    ; 预热1（PLA）：热端 200°C，热床 60°C，风扇关
M145 S1 H240 B90  F255  ; 预热2（ABS）：热端 240°C，热床 90°C，风扇全速
M500                     ; 保存到 EEPROM
```

**常用材料参考温度：**

| 材料 | 热端 | 热床 | 风扇 |
|---|---|---|---|
| PLA | 190–210°C | 55–65°C | 全速（255）|
| PETG | 230–245°C | 70–85°C | 半速（128）|
| ABS | 230–250°C | 90–110°C | 关闭（0）|
| TPU | 210–230°C | 40–60°C | 低速（51）|

### 方法三：修改固件默认值（重新编译）

如需修改 `M502` 后仍有效的出厂默认值，编辑 `Marlin/Configuration.h`：

```c
// 约第 2100 行
#define PREHEAT_1_LABEL       "PLA"
#define PREHEAT_1_TEMP_HOTEND 200   // ← 修改热端温度
#define PREHEAT_1_TEMP_BED     60   // ← 修改热床温度
#define PREHEAT_1_FAN_SPEED     0   // 0 = 风扇关

#define PREHEAT_2_LABEL       "ABS"
#define PREHEAT_2_TEMP_HOTEND 240   // ← 修改热端温度
#define PREHEAT_2_TEMP_BED     90   // ← 修改热床温度
#define PREHEAT_2_FAN_SPEED   255   // 255 = 全速
```

修改后重新编译对应的构建环境（如 `pio run -e 4MAXP`）并刷写。

---

## Cura 配置

### 打印机参数（管理打印机 → 机器设置）

| 项目 | 值 |
|---|---|
| X 宽度 | 270 mm |
| Y 深度 | 205 mm |
| Z 高度 | 205 mm |
| 加热床 | ✓ 启用 |
| G-code 风格 | RepRap |

### 切片参数建议

| 参数 | 建议值 | 说明 |
|---|---|---|
| 外墙打印速度 | 40–50 mm/s | 过慢会导致积热溢料 |
| 打印加速度 | 500–700 mm/s² | 在固件参数范围内 |
| Z 缝对齐 | Sharpest Corner | 缝隙藏在模型不显眼处 |
| Outer Wall Wipe Distance | 0.2 mm | 消耗外墙末端余料 |
| Enable Coasting | **关闭** ✗ | 与 Linear Advance 冲突，二者不能共用 |
| Enable Combing | Not in Skin | 减少表面拉丝 |

### 预热温度参考

| 材料 | 热端 | 热床 |
|---|---|---|
| PLA | 190–210°C | 55–65°C |
| PETG | 230–245°C | 70–85°C |
| ABS | 230–250°C | 90–110°C |

---

## 起始 G-code

```gcode
G21                          ; 公制单位
G90                          ; 绝对坐标模式
M82                          ; 挤出机绝对模式
M107                         ; 关闭风扇

; ── 归零与调平 ──────────────────────────────
G28                          ; 全轴回零（X Y Z）
M420 S1                      ; 启用网格调平补偿
M900 K0.2                    ; Linear Advance K 值（按校准结果修改）

; ── 擦嘴 ────────────────────────────────────
G1 Z5 F3000                  ; 抬升 Z 轴防止刮床
G1 X-3 Y40 F6000             ; 移到擦嘴位置
G1 X-3 Y5  F3000             ; 擦嘴
G1 X-3 Y40 F3000
G1 X-3 Y5  F3000

; ── 打印前引线 ──────────────────────────────
G1 Z0.3 F1200                ; 降到打印高度
G1 X0   Y5  F3000            ; 移到床边起点
G92 E0                       ; 重置挤出量
G1 X0   Y60 E6   F800        ; 引线第一段
G1 X0.5 Y60      F3000       ; 轻微偏移
G1 X0.5 Y5  E12  F800        ; 引线第二段（双线更可靠）
G92 E0

; ── 开始打印 ────────────────────────────────
G1 Z1.0 F3000
M117 Printing...
```

---

## 结束 G-code

```gcode
; ══ 立刻离开模型（最高优先级）════════════════
M400                         ; 等待当前动作完成
G92 E0
G1 E-3 F1800                 ; 快速回抽 3 mm，立刻释放喷嘴压力
G91                          ; 相对坐标
G1 Z15 F3000                 ; 抬起 Z 轴 15 mm（离开模型顶面）
G90                          ; 切回绝对坐标
G1 X10 Y200 F9000            ; 以最快速度将喷嘴移到床尾远端
M107                         ; 关闭冷却风扇

; ══ 关闭加热，喷嘴已不在模型上方 ═════════════
M104 S0                      ; 关闭热端
M140 S0                      ; 关闭热床

; ══ 等待降温后深度回抽（防漏料） ═════════════
M109 R150                    ; 等待降至 150°C（此时才无流动风险）
G1 E-10 F300                 ; 深度回抽 10 mm，彻底防漏

; ══ 结束 ════════════════════════════════════
M84                          ; 关闭步进电机
M300 S2000 P200              ; 提示音
M300 S0    P100
M300 S2000 P200
M117 Print Complete!
```

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

### Q: 顶层出现溢料凸起

结束 G-code 中 `M400` + `G1 E-3` + `G1 Z15` + `G1 X10 Y200` 这四行必须**在关闭加热之前**执行，确保喷嘴在热端还热时立刻离开模型。

### Q: Z 缝位置不美观

在 Cura 中将 **Z Seam Alignment** 改为 **Sharpest Corner**，将 Z 缝藏在模型尖角处。

### Q: 更换耗材后需要重新校准哪些参数

每次更换不同品牌或材质的耗材，建议重新执行：
- **挤出机步进校准**（如果流量有明显变化）
- **Linear Advance K 值校准**（不同材质差异较大）
- **PID 校准**（如果打印温度差距大于 30°C）

### Q: 打印温度始终是 170°C，无论什么模型都一样

这是 **Cura 材料配置**问题，与固件无关。固件中 `EXTRUDE_MINTEMP 170` 是挤出最低安全温度（低于此值不允许挤出），不是目标打印温度。

解决方法（Cura）：

1. 点击顶部材料选择器（如 **Generic PLA**）→ **管理材料**
2. 找到当前使用的材料，点击编辑
3. 将 **打印温度（Printing Temperature）** 从 170°C 修改为正确值（PLA 通常 190–210°C）
4. 保存后重新切片

若问题仍存在，检查 Cura 的 **机器设置 → 起始 G-code**，确认其中没有硬编码 `M109 S170` 之类的温度命令。

---

### Q: 刷入新固件后如何保留已完成的 PID 和网格调平数据

Marlin 的校准数据存储在 EEPROM 中。只要不执行 `M502`，数据通常可以保留。  
**刷机前务必执行 `M503` 备份输出内容**，详细操作请参阅[刷写后必做操作](#刷写后必做操作)。

---

### Q: 如何恢复原厂固件

原厂固件（v1.1.7）备份地址：  
`https://drive.google.com/file/d/1FwKHQcOxPabLgirkihu3LnBMuHuZLqZR/view`  
使用与刷写本固件相同的方法，替换 hex 文件刷写即可。

---

## 相关链接

- [原项目 GitHub（knutwurst）](https://github.com/knutwurst/Marlin-2-0-x-Anycubic-i3-MEGA-S)
- [参考项目：ZhangGaoxing 4Max Pro 固件](https://github.com/ZhangGaoxing/Marlin-A4MaxPro-2.0.x)
- [Marlin 官方文档](https://marlinfw.org/docs/configuration/configuration.html)
- [PlatformIO 安装指南](https://platformio.org/install/cli)
