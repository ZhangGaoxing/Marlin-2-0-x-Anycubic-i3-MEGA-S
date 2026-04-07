# CoLiDo D1315 固件适配指南

本文档介绍如何在 knutwurst 的 Marlin 2.1.x 多机型固件中为 **CoLiDo D1315**（Delta 三角洲打印机）编译、刷写固件，以及首次运行后的测试与配置流程。

---

## 目录

- [机型说明](#机型说明)
- [硬件参数](#硬件参数)
- [编译与刷写](#编译与刷写)
- [固件功能列表](#固件功能列表)
- [刷写后测试流程](#刷写后测试流程)
- [初始配置](#初始配置)
- [无屏幕操作指南](#无屏幕操作指南)
- [常用 G/M 指令速查](#常用-gm-指令速查)
- [常见问题](#常见问题)

---

## 机型说明

| 项目 | 内容 |
|------|------|
| 机型品牌 | CoLiDo D1315 |
| 运动学结构 | Delta（三角洲，基于 Mini Kossel / jcrocholl） |
| 原始固件 | Marlin 1.0.0 |
| 主板 | RAMPS 1.4（ATmega2560 @ 16MHz） |
| 编译标识 | `KNUTWURST_D1315` |
| 编译环境名 | `D1315` |

---

## 硬件参数

以下参数从原始固件串口日志（`output.log`）和 Repetier-Host 注册表配置中提取：

| 参数 | 值 | 来源 |
|------|-----|------|
| 步进 X/Y/Z（steps/mm） | 100 / 100 / 100 | M92（串口日志） |
| 步进 E（steps/mm） | 94.31 | M92（串口日志） |
| 最大进给速度 XYZ/E（mm/s） | 200 / 43 | M203 |
| 最大加速度（mm/s²） | 9000（全轴） | M201 |
| 打印/回抽/移动加速度（mm/s²） | 3000 | M204 |
| Jerk X/Y/Z（mm/s） | 20 / 20 / 20 | M205 |
| 热端 PID（P/I/D） | 24.00 / 0.40 / 20.00 | M301 |
| 热床 | 无 | — |
| SD 卡 | 无 | — |
| Delta 斜杆长度（mm） | 150.0 | 实测 |
| Delta 半径（mm） | 67.0（Repetier rostockRadius = Marlin DELTA_RADIUS） | Repetier 配置 |
| Delta 高度（mm） | 161.0（Repetier zhomepos）→ Marlin DELTA_HEIGHT **170.0** | 实机校准 |
| 可打印半径（mm） | 67.0 | 计算值 |
| 限位开关位置 | 顶部（MAX），接在 RAMPS MIN 接口 | 实测 |
| 限位开关类型 | NC（常闭），上拉，触发时为 HIGH | 实测 |

---

## 编译与刷写

### 环境准备

```powershell
# 创建 conda 环境（Python 3.11）
conda create -n marlin-pio python=3.11
conda activate marlin-pio

# 安装 PlatformIO
pip install platformio
```

### 编译

```powershell
cd "C:\path\to\Marlin-2-0-x-Anycubic-i3-MEGA-S"
& "C:\Users\zhang\miniconda3\envs\marlin-pio\Scripts\pio.exe" run -e D1315
```

生成固件：`.pio\build\D1315\firmware.hex`

### 刷写

打印机通过 USB 连接电脑（COM 端口因系统而异，下例为 COM3）：

```powershell
& "C:\Users\zhang\miniconda3\envs\marlin-pio\Scripts\pio.exe" run -e D1315 --target upload
```

或使用 avrdude 手动指定端口：

```powershell
.\avrdude.exe -c wiring -p atmega2560 -P COM3 -b 115200 `
  -U flash:w:".pio\build\D1315\firmware.hex":i
```

---

## 固件功能列表

相比原始 Marlin 1.0.0，新固件（Marlin 2.1.2.7）新增/改进以下功能：

| 功能 | 说明 |
|------|------|
| **EEPROM 存储** | 所有参数可通过 `M500` 持久化保存 |
| **线性提升（Linear Advance）** | `M900 K<值>` 减少过渡段拉丝，改善角落质量 |
| **S-Curve 加速** | 更平滑的速度曲线，减少机械振动 |
| **自动温度控制（AUTOTEMP）** | 根据打印速度自动调节温度 |
| **热保护（Thermal Runaway）** | 热端温度异常时自动停机，防止火灾 |
| **断电续打（Power Loss Recovery）** | 断电后可恢复打印（需 SD 卡） |
| **耗材检测（Filament Runout）** | 耗材耗尽时暂停打印（需传感器） |
| **Multi-Point 调平** | 支持手动 G29 网格调平 |
| **M665/M666 Delta 几何调整** | 运行时调整斜杆长度、半径、高度及塔角偏差 |
| **Host Action Commands** | 支持 OctoPrint/Repetier-Server 暂停/继续/取消 |
| **PID 自整定** | `M303` 自动整定热端 PID 参数 |
| **弧线插补（ARC Support）** | G2/G3 圆弧指令，减少 G-code 文件体积 |

> **不包含的功能**（因 D1315 硬件限制而禁用）：
> - LCD 屏幕控制（无屏幕）
> - 热床控制（无热床）
> - DELTA_AUTO_CALIBRATION G33（无探针）
> - BLTouch / 自动调平探针

---

## 刷写后测试流程

使用串口终端（波特率 **250000**，推荐 Repetier-Host 或 OctoPrint）连接打印机，按以下顺序测试：

### 第一步：检查串口响应

上电后应收到一次启动日志，最后一行为：
```
//action:prompt_end
```
如果循环输出，检查 Arduino ISP 的 D10 线是否仍连接 Mega RESET 引脚。

### 第二步：检查限位开关状态

```gcode
M119
```

**预期结果**（静止，未触碰开关时）：
```
x_max: open
y_max: open
z_max: open
filament: open
```

手动按下每个塔顶的限位开关，对应轴应变为 `TRIGGERED`。

若全部显示 `TRIGGERED`，修改 `Configuration.h` 中 D1315 的 `*_MAX_ENDSTOP_INVERTING`（`true` ↔ `false`）。

### 第三步：测试电机方向

手动控制单轴运动（使用 Repetier-Host 的手动控制面板或发 G0 指令），确认三根立柱方向正确。若某轴方向反转，在 `ini/d1315.ini` 中修改对应的 `-DINVERT_*_DIR=true/false`。

### 第四步：测试归零

```gcode
G28
```

**预期行为**：三根立柱**同时向上**移动，触碰顶部限位后回退约 5mm 停止，坐标归零。

若电机不动则限位开关读数有误（见第二步）；若碰到开关不停止则引脚映射错误。

### 第五步：检查打印高度

```gcode
G28
G1 Z5 F1000   ; 移到 Z=5mm
G1 Z0 F300    ; 缓慢下降到 Z=0
```

在 Z=0 时，喷嘴应刚好接触或轻压打印平台。用一张标准 A4 纸（约 0.1mm）测试间隙。

---

## 初始配置

首次运行或调整几何参数后需进行以下配置：

### 1. 重置 EEPROM（清除旧版参数）

```gcode
M502   ; 恢复固件默认值
M500   ; 保存到 EEPROM
```

### 2. 调整 Delta 高度（H 值）

```gcode
G28              ; 先归零
G1 Z0 F300       ; 移到理论 Z=0
; 用纸测试喷嘴间隙
M665 H162.0      ; 增大 H 值 → 喷嘴抬高；减小 H 值 → 喷嘴降低
M500
G28
G1 Z0 F300       ; 重复测试直到满意
```

每次修改 H 值后需重新 `G28`。

### 3. 调整 Delta 半径（R 值）

R 值影响边缘高度一致性：

```gcode
; 测试中心和边缘高度是否一致
; 若边缘偏高 → 减小 R；边缘偏低 → 增大 R
M665 R67.5
M500
G28
```

### 4. 调整塔角偏差（X/Y/Z 微调）

若三个方向 120° 处高度不一致，微调对应塔的角度：

```gcode
M665 X0.5 Y-0.3 Z0.0   ; 微调 X/Y/Z 塔角（单位：度）
M500
```

### 5. 调整各塔端点偏差（M666）

若归零后三塔触发位置有偏差：

```gcode
M666 X0.5 Y-0.2 Z0.0   ; 各塔端点偏移（正值 = 塔稍低）
M500
```

### 6. PID 自整定（如热端温度不稳定）

```gcode
M303 E0 S200 C8   ; 在 200°C 进行 8 次循环自整定
; 完成后记录输出的 Kp/Ki/Kd 值，然后：
M301 P<Kp> I<Ki> D<Kd>
M500
```

### 7. 关闭耗材检测（如未安装传感器）

```gcode
M412 S0
M500
```

---

## 无屏幕操作指南

D1315 没有 LCD 屏幕，所有操作通过串口终端完成。推荐工具：

| 工具 | 特点 |
|------|------|
| **Repetier-Host** | 功能完整，有手动控制面板，适合调试 |
| **OctoPrint** | 网页界面，适合联网控制 |
| **Pronterface** | 轻量，开源，适合快速测试 |
| **Arduino IDE 串口监视器** | 临时调试，不支持文件上传 |

**连接参数**：波特率 `250000`，无奇偶校验，1 停止位

---

## 常用 G/M 指令速查

### 运动控制

| 指令 | 说明 | 示例 |
|------|------|------|
| `G28` | 全轴归零（Delta 为向上归位） | `G28` |
| `G1` | 直线移动 | `G1 X0 Y0 Z50 F3000` |
| `G0` | 快速移动（不挤料） | `G0 Z100 F5000` |
| `G92` | 设置当前位置为坐标原点 | `G92 E0` |
| `G2/G3` | 顺/逆时针弧线移动 | `G2 I10 J0 F1000` |

### 温度控制

| 指令 | 说明 | 示例 |
|------|------|------|
| `M104` | 设置热端目标温度（不等待） | `M104 S200` |
| `M109` | 设置热端温度并等待到达 | `M109 S200` |
| `M105` | 查询当前温度 | `M105` |
| `M303` | PID 自整定 | `M303 E0 S200 C8` |
| `M301` | 设置热端 PID 参数 | `M301 P24 I0.4 D20` |

### 挤出机控制

| 指令 | 说明 | 示例 |
|------|------|------|
| `M82` | 挤出机绝对模式 | `M82` |
| `M83` | 挤出机相对模式 | `M83` |
| `G1 E10 F100` | 送料 10mm | — |
| `G1 E-5 F1000` | 回抽 5mm | — |
| `M900` | 设置 Linear Advance K 值 | `M900 K0.05` |

### Delta 几何参数

| 指令 | 说明 | 示例 |
|------|------|------|
| `M665` | 查询/设置 Delta 几何参数 | `M665 L150 R67 H161` |
| `M666` | 查询/设置各塔端点偏差 | `M666 X0 Y0 Z0` |

`M665` 参数说明：

| 参数 | 含义 |
|------|------|
| `L` | 斜杆长度（mm） |
| `R` | Delta 半径（mm） |
| `H` | Delta 高度/打印高度（mm） |
| `S` | 每秒插补段数 |
| `X/Y/Z` | 各塔角度偏差（度） |
| `A/B/C` | 各塔斜杆长度偏差（mm） |

### 限位开关与归零

| 指令 | 说明 |
|------|------|
| `M119` | 查询所有限位开关状态 |
| `M211 S0` | 关闭软件限位（调试时用） |
| `M211 S1` | 开启软件限位 |

### EEPROM 与系统

| 指令 | 说明 |
|------|------|
| `M500` | 保存当前参数到 EEPROM |
| `M501` | 从 EEPROM 加载参数 |
| `M502` | 恢复固件编译默认值（不自动保存） |
| `M503` | 打印当前所有参数（不读 EEPROM） |
| `M112` | 紧急停机 |
| `M999` | 清除紧急停机状态，恢复运行 |

### 风扇与电机

| 指令 | 说明 | 示例 |
|------|------|------|
| `M106` | 设置风扇转速（0-255） | `M106 S128` |
| `M107` | 关闭风扇 | `M107` |
| `M84` | 关闭所有电机 | `M84` |
| `M17` | 启用所有电机 | `M17` |

### 打印流程（无 SD 卡，通过主机发送）

```gcode
M109 S200          ; 加热到 200°C 并等待
G28               ; 归零
G92 E0            ; 挤出归零
M83               ; 挤出机相对模式
G1 Z5 F3000       ; 抬起喷嘴
; --- 开始打印（由切片软件生成的 G-code）---
G1 X0 Y0 Z0.2 F3000
G1 X30 E5 F600
; ...
M104 S0           ; 关闭热端
M84               ; 关闭电机
```

---

## 常见问题

**Q: G28 后电机不动**
- 检查 `M119`，若静止时有 `TRIGGERED`，修改 `Configuration.h` 中 D1315 的 `*_MAX_ENDSTOP_INVERTING`（true ↔ false）重新编译烧录。

**Q: G28 触碰开关后电机不停止**
- 说明限位引脚映射错误。检查 `ini/d1315.ini` 中 `X_MAX_PIN/Y_MAX_PIN/Z_MAX_PIN` 是否指向正确的物理引脚（应为 3/14/18，对应 RAMPS X-/Y-/Z- 接口）。

**Q: 某根立柱运动方向反转**
- 在 `ini/d1315.ini` 的 `build_flags` 中添加或修改：
  ```
  -DINVERT_X_DIR=true
  ```
  重新编译烧录。

**Q: 上电后串口日志循环输出**
- 断开 Arduino ISP 与打印机的全部连线，RESET 被持续触发导致循环重启。

**Q: 打印时热端温度不稳定**
- 运行 PID 自整定：`M303 E0 S200 C8`，将结果写入 `M301` 并 `M500` 保存。

**Q: 如何恢复原始 Repetier 固件**
- 使用 avrdude 通过 ArduinoISP 烧录备份文件：
  ```powershell
  .\avrdude.exe -c avrisp -p atmega2560 -P COM4 -b 9600 `
    -U flash:w:"D1315_backup.hex":i `
    -U eeprom:w:"D1315_eeprom.bin":r
  ```

**Q: 如何烧录固件**

- ICSP 标准针脚方向（面朝自己看排针）：
```
[ MISO | VCC  ]  ← 第1行（有白点/三角标记的角为 MISO）
[ SCK  | MOSI ]
[ RST  | GND  ]
```

```
Uno (ArduinoISP)  →   Mega ICSP 排针
─────────────────────────────────────
5V                →   VCC  (2号脚)
GND               →   GND  (6号脚)
D11 (MOSI)        →   MOSI (4号脚)
D12 (MISO)        →   MISO (1号脚)  ← 白点/缺角处
D13 (SCK)         →   SCK  (3号脚)
D10               →   RST  (5号脚)

Uno RESET ── 10µF(+) ── 电容 ── GND
```

- 烧写命令
```shell
.\avrdude.exe -c avrisp -p atmega2560 -P COM4 -b 19200 -U flash:w:"firmware.hex":i
```

**Q: 三点调平**

```
G28
G1 X0 Y0 Z0 F1500        ; 测中心
G1 X55 Y0 Z0 F1500  ; 测 X 方向边缘
G1 X0 Y55 Z0 F1500  ; 测 Y 方向边缘
G1 X-55 Y0 Z0 F1500 ; 测 X 反向边缘
```