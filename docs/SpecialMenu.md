# Special Menu（特殊菜单）功能说明

## 如何进入

在触摸屏**文件列表界面**，当 SD 卡未插入时，列表第一项会显示 `<Special Menu>`，点击即可进入。若 SD 卡已插入，`<Special Menu>` 出现在每个目录的第一条，同样点击进入。

---

## 主菜单结构

菜单共 4 页（每页 4 个条目），使用触摸屏左右翻页：

### 第 1 页
| 条目 | 功能 |
|---|---|
| `<Set Flowrate>` | 进入挤出流量子菜单 |
| `<Preheat Ultrabase>` | 预热热床至 60°C |
| `<Fil. Change Pause>` | 执行耗材更换暂停（M600）|
| `<Fil. Change Resume>` | 继续暂停中的耗材更换 |

### 第 2 页（因配置不同而异）

| 配置 | 条目 | 功能 |
|---|---|---|
| **无 BLTouch / 无 Chiron** | `<Easy 4 Point Level>` | 进入四点手动调平子菜单 |
| | `<Mesh Leveling>` | 进入手动网格调平子菜单 |
| | `<PID Tune Hotend>` | 喷头 PID 自动整定 |
| | `<PID Tune Ultrabase>` | 热床 PID 自动整定 |
| **BLTouch 版本** | `<Easy 4 Point Level>` | 进入四点手动调平子菜单 |
| | `<Auto Leveling>` | 进入 BLTouch Z 偏移调整子菜单 |
| | `<PID Tune Hotend>` | 喷头 PID 自动整定 |
| | `<PID Tune Ultrabase>` | 热床 PID 自动整定 |
| **Chiron 版本** | `<Easy 4 Point Level>` | 进入四点手动调平子菜单 |
| | `<Reset Level Grid>` | 重载调平数据（M501 + M420 S1）|
| | `<PID Tune Hotend>` | 喷头 PID 自动整定 |
| | `<PID Tune Ultrabase>` | 热床 PID 自动整定 |

### 第 3 页
| 条目 | 功能 |
|---|---|
| `<Load FW Defaults>` | 恢复固件默认参数（M502）|
| `<Save EEPROM>` | 保存当前参数到 EEPROM（M500）|
| `<Disable Fil. Sensor>` | 禁用断料传感器并保存 |
| `<Enable Fil. Sensor>` | 启用断料传感器并保存 |

### 第 4 页
| 条目 | 功能 |
|---|---|
| `<Exit>` | 退出 Special Menu |

---

## 子菜单详情

### 流量设置子菜单（`<Set Flowrate>`）

| 条目 | 功能 |
|---|---|
| `<Flow is XXX>` | 显示当前流量百分比（只读） |
| `<Up>` | 流量 +1%（上限 800%）|
| `<Down>` | 流量 -1%（下限 1%）|
| `<Exit Flow Settings>` | 退出 |

---

### 手动网格调平子菜单（`<Mesh Leveling>`，无 BLTouch / 无 Chiron 时可用）

进入时自动禁用软件限位（Soft Endstops），退出时恢复。

**第 1 页**
| 条目 | 执行命令 | 说明 |
|---|---|---|
| `<Start Mesh Leveling>` | G28 + G29 S1 | 归零并开始网格调平 |
| `<Z Up 0.1>` | G91 G1 Z+0.1 G90 | Z 轴上移 0.1mm |
| `<Z Down 0.1>` | G91 G1 Z-0.1 G90 | Z 轴下移 0.1mm |
| `<Z Up 0.02>` | G91 G1 Z+0.02 G90 | Z 轴上移 0.02mm |

**第 2 页**
| 条目 | 执行命令 | 说明 |
|---|---|---|
| `<Z Down 0.02>` | G91 G1 Z-0.02 G90 | Z 轴下移 0.02mm |
| `<Z Up 0.01>` | G91 G1 Z+0.03 G4 P250 G1 Z-0.02 G90 | 净移 +0.01mm（消除间隙） |
| `<Z Down 0.01>` | G91 G1 Z+0.02 G4 P250 G1 Z-0.03 G90 | 净移 -0.01mm（消除间隙） |
| `<Next Mesh Point>` | G29 S2 | 移至下一个网格点 |

**第 3 页**
| 条目 | 功能 |
|---|---|
| `<Save EEPROM>` | 保存调平数据（M500）|
| `<End Mesh Leveling>` | 退出子菜单并重新启用软件限位 |

---

### 四点简易调平子菜单（`<Easy 4 Point Level>`）

进入时先执行归零（G28）并禁用网格补偿（M420 S0），移到 A 点（Z=0）。退出时保存并启用网格补偿（M420 S1）。

**第 1 页**（点位坐标因机型而异）

| 条目 | 打印机型号 | 坐标 |
|---|---|---|
| `<Point A>`（前左）| i3 Mega / Mega-S / Mega-P | X15 Y15 |
| `<Point B>`（前右）| i3 Mega / Mega-S / Mega-P | X205 Y15 |
| | Mega-X | X295 Y15 |
| | Chiron | X385 Y15 |
| | 4Max Pro 2 | X255 Y15 |
| `<Point C>`（后右）| i3 Mega / Mega-S / Mega-P | X205 Y200 |
| `<Point D>`（后左）| i3 Mega / Mega-S / Mega-P | X15 Y200 |

**第 2 页**
| 条目 | 功能 |
|---|---|
| `<Exit Easy Level>` | 退出并保存网格补偿（M420 S1）|

---

### BLTouch Z 偏移调整子菜单（`<Auto Leveling>`，仅 BLTouch 版本）

进入时禁用软件限位，退出时恢复。

**第 1 页**
| 条目 | 功能 |
|---|---|
| `<Z Offset: XXXXX>` | 显示当前 Z 偏移值（只读） |
| `<Up>` | Z 偏移 +0.01mm |
| `<Down>` | Z 偏移 -0.01mm |
| `<Start Auto Leveling>` | 全自动调平（G28 + G29）后归位 |

**第 2 页**
| 条目 | 功能 |
|---|---|
| `<Enable HiSpeed Mode>` | 启用 BLTouch 高速模式（M401 S1 + M500）|
| `<Disable HiSpeed Mode>` | 禁用 BLTouch 高速模式（M401 S0 + M500）|
| `<SAVE and EXIT>` | 保存 EEPROM 并退出 |

---

## PID 自动整定说明

### 喷头 PID（`<PID Tune Hotend>`）

执行流程：归零 → 移至中心 → 风扇 67% → 目标 215°C × 15 次循环 → 自动保存（M500）→ 完成提示音。

### 热床 PID（`<PID Tune Ultrabase>`）

执行流程：目标 60°C × 6 次循环 → 自动保存（M500）→ 完成提示音。整定过程中热床会自然升降温，约需数分钟。

---

## 注意事项

- **网格调平和四点调平期间**：软件限位被临时禁用，Z 轴可能超出正常范围，请小心操作，避免喷头撞击热床
- **PID 整定期间**：喷头会升至 215°C，打印机会自动完成，请勿中途断电
- **耗材更换**（`<Fil. Change Pause>`）：需在打印中使用，空闲状态下执行无实际效果
- **恢复固件默认**（`<Load FW Defaults>`）会清除 EEPROM 中的 PID 值、调平数据等，操作后记得重新校准并执行 `<Save EEPROM>`