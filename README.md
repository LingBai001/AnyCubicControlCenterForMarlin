# AnyCubicControlCenterForMarlin-纵维立方有线控制终端

> 一个通过 USB 串口控制 Anycubic（纵维立方）全系 FDM 打印机的桌面控制终端
> 无需 OctoPrint、无需树莓派，插上 USB 线材,就能在电脑上完成打印前的全部操作

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Windows-lightgrey.svg)]()
[![Firmware](https://img.shields.io/badge/Firmware-Marlin%202.0.x-green.svg)]()

---

## 目录

- [为什么做这个](#为什么做这个)
- [功能特性](#功能特性)
- [支持的机型](#支持的机型)
- [快速开始](#快速开始)
- [界面说明](#界面说明)
- [技术实现](#技术实现)
- [已知限制](#已知限制)
- [常见问题](#常见问题)
- [从源码构建](#从源码构建)
- [目录结构](#目录结构)
- [配套固件](#配套固件)
- [贡献与许可](#贡献与许可)
- [免责声明](#免责声明)

---

## 为什么做这个

Anycubic 的原厂上位机只支持 Windows，Linux 用户长期只能靠 Pronterface、Cura 的
Monitor 这类通用工具凑合。它们要么界面停留在十年前，要么缺少这台机器真正需要的东西：

- 不知道你的机型有没有热床，一律显示热床控件；
- 打开串口时会拉 DTR，**打印过程中连上去，正在打的件直接报废**；
- 手动调平全靠手敲 G-code

这个项目把这些补齐：**按机型自动适配界面**、**串口永不误复位**、**五点调平向导**

---

## 功能特性

### 温度

- PLA / ABS / PETG 一键预热预设
- 自定义温度输入（0–260℃，上限随机型自动调整）
- **一键降温**：关加热 + 风扇全速 + 降到 50℃ 自动停风

### 风扇

- 无级调速滑条（0–100%）
- 关 / 30% / 50% / 100% 快捷档

### 移动

- 方向键盘（X± / Y± / Z±），步长 0.1 / 1 / 10 / 50 mm
- 全部归零、单轴归零、解锁电机（M84）、限位状态（M119）

### 手动调平向导

- 自动 `G28` 回原点并抬高 Z
- 依次走到 **左前 → 右前 → 右后 → 左后 → 中心** 五个点
- 每点自动降到 0.2mm，配合 A4 纸"轻阻"判断
- Z 轴微调 ±0.05 / ±0.5 mm，转动一格就能感觉到

### 文件与打印

- **SD 卡**：浏览文件、打印、暂停、继续、停止
- **电脑文件**：选择本地 G-code 直接流式发送，带 `ok` 流控与 `resend` 重传

### 其他

- **机型自动识别**：连接即发 `M115`，按机型启用/隐藏热床
- **自动连接**：启动后自动连上次用的串口
- **窗口记忆**：记住尺寸与位置
- 全中文界面

---

## 支持的机型

连接后读取 `M115` 的 `MACHINE_TYPE` 自动匹配档案：

| 机型 | 热床 | 热床上限 | 喷嘴上限 |
|---|---|:---:|:---:|
| Mega Zero | 无（自动隐藏控件） | — | 260℃ |
| Mega Zero 2.0 | 有 | 100℃ | 260℃ |
| Mega S | 有 | 100℃ | 260℃ |
| Chiron | 有 | 100℃ | 260℃ |
| Vyper | 有 | 110℃ | 260℃ |
| Kobra / Kobra Go / Plus / Max | 有 | 100℃ | 260℃ |
| 其它 Marlin 设备 | 有（通用档） | 110℃ | 260℃ |

> 机型档案在 `PROFILES` 表里，新增机型只需加一行正则与参数。

---

## 快速开始

### Linux（Debian / Kali / Ubuntu）

```bash
sudo apt install ./anycubic-ctl_1.11.0_all.deb
```

安装后从应用列表启动「纵维立方有线控制终端」，或终端执行：

```bash
anycubic-ctl
```

依赖：`python3`、`python3-gi`、`gir1.2-gtk-3.0`（发行版基本自带）。

### Windows

**方式 A：自己打包 exe（推荐，产物可分发）**

1. 安装 Python 3.8+（勾选 `Add Python to PATH`）
2. 双击 `build_exe.bat`
3. 产物在 `dist\anycubic-ctl.exe`

该 exe 内置 Python 运行时、tkinter、pyserial，可直接拷到没装 Python 的电脑运行。

**方式 B：直接跑源码**

```bat
pip install pyserial
python anycubic_ctl.py
```

或双击 `run.bat`（首次自动装依赖）。

### 源码运行（任意平台）

```bash
pip install pyserial
python anycubic_ctl.py
```

---

## 界面说明

布局自适应：窗口宽 ≥ 760px 时双列，变窄自动切单列。

```
┌──────────────────────┬──────────────────────┐
│ 状态                  │ 移动                  │
│  喷嘴/热床实时温度     │  方向键盘             │
│  X/Y/Z 坐标           │  步长选择             │
│  打印进度条            │  全部归零 / 单轴归零   │
├──────────────────────┤  解锁电机 / 限位状态   │
│ 温度                  │  手动调平…            │
│  预设预热             ├──────────────────────┤
│  自定义 + 降温         │ SD 卡                 │
├──────────────────────┤  文件列表             │
│ 风扇                  │  打印/暂停/继续/停止   │
│  滑条 + 快捷档         │                      │
├──────────────────────┤                      │
│ 电脑上的 G-code       │                      │
│  选择/发送/中止        │                      │
└──────────────────────┴──────────────────────┘
┌────────────────────────────────────────────┐
│ 控制台：任意 G-code 收发                     │
└────────────────────────────────────────────┘
```

---

## 技术实现

### 串口层：永不误复位

这是本项目最重要的设计决策。

多数 3D 打印机主板（Melzi / CH340 方案）把 USB 转串口芯片的 **DTR 引脚接到了
MCU 的复位脚**。用标准串口库打开端口时，DTR 会从低变高，等于按了一次复位键 ——
**打印过程中连上去，正在打的件立刻中断**。

本项目在打开端口时完全不驱动 DTR/RTS：

- **Linux**：绕过 pyserial，直接 `os.open()` + `termios`，清掉 `HUPCL`，
  控制线保持浮空
- **Windows**：pyserial 打开前设置 `dtr=False`、`rts=False`、`dsrdtr=False`，
  打开后再复位一次状态

> 这是作者在真机上把一次测试打印打断两次之后加的。

### 流控：ok 逐行 + resend 重传

AVR 主板的串口接收缓冲只有 128 字节，一次性灌入大量 G-code 会丢字符，
表现为打印错层、乱走。

发送器采用严格应答式：

```
发一行 → 等 "ok" → 发下一行
收到 "resend N" → 回退一行重发
```

牺牲速度换正确性。大文件建议仍拷到 SD 卡打印。

### 机型识别

```
M115 → FIRMWARE_NAME:Marlin 2.0.9.3 MACHINE_TYPE:Mega Zero ...
                ↓ 正则匹配 PROFILES[].match
        命中档案 → 应用热床存在性、温度上限、成型尺寸
```

### 跨平台 GUI

| 平台 | GUI 工具包 | 原因 |
|---|---|---|
| Linux | GTK3（PyGObject） | 与 GNOME 桌面原生一致，视觉最贴合 |
| Windows | Tkinter（标准库） | GTK 在 Windows 部署依赖繁重，Tk 零安装 |

两版功能与布局完全一致，仅渲染层不同。

---

## 已知限制

1. **Wayland 下窗口位置记忆无效**
   GNOME 的 Wayland 合成器不允许客户端自定位窗口，`move()` 被忽略。
   尺寸与最大化状态正常。需要位置记忆请切 X11 会话。

2. **大文件发送慢**
   逐行应答式流控，每行约 10–20ms。4MB 的 G-code 约需 40 分钟。
   大件请用 SD 卡。

3. **不含网格补偿**
   受下位机 128KB Flash 限制（固件已占 99.8%），本工具只做
   上位机引导的手动五点调平，不生成网格补偿数据。
   换 32 位主板后可接入 Marlin 的 `MESH_BED_LEVELING`。

---

## 常见问题

**Q：下拉框里没有串口？**

- Linux：确认 `ls /dev/ttyUSB*` 存在，用户需在 `dialout` 组
  （`sudo usermod -aG dialout $USER`，重新登录生效）
- Windows：设备管理器里应出现 `USB-SERIAL CH340 (COMx)`，
  没有则装驱动：<https://www.wch.cn/downloads/CH341SER_EXE.html>

**Q：连接失败 / 端口被占用？**

同一时刻只能有一个程序占用串口。关闭 Pronterface、Cura、Arduino IDE 等。

**Q：为什么连上后机器没有重启？**

这是有意为之 —— 见 [串口层：永不误复位](#串口层永不误复位)。

**Q：风扇关不掉？**

若机型风扇使用硬件 PWM 而定时器被步进电机占用，M106 会失效。
需要在固件里启用 `FAN_SOFT_PWM`（本项目配套固件已启用）。

**Q：能控制树脂机（Photon 系列）吗？**

不能。树脂机走 U 盘 + 私有格式，与 FDM 的串口 G-code 协议完全不同。

---

## 从源码构建

### Linux 打 deb

```bash
mkdir -p pkg/DEBIAN pkg/usr/bin pkg/usr/share/applications \
         pkg/usr/share/icons/hicolor/scalable/apps
install -m755 anycubic-ctl.py pkg/usr/bin/anycubic-ctl
# 写入 DEBIAN/control、.desktop、图标后：
dpkg-deb --build pkg anycubic-ctl_1.11.0_all.deb
```

### Windows 打 exe

```bat
python -m pip install pyinstaller pyserial
python -m PyInstaller --onefile --windowed --name anycubic-ctl ^
    --icon anycubic.ico anycubic_ctl.py
```

或直接双击 `build_exe.bat`。

---

## 目录结构

```
anycubic-ctl/
├── anycubic_ctl.py          # 跨平台主体（Tkinter + pyserial）
├── run.bat                  # Windows 直接运行入口
├── build_exe.bat            # Windows 一键打包 exe
├── anycubic.ico             # 应用图标（6 尺寸）
├── requirements.txt         # pyserial
└── README.md
```

Linux 版为单文件 `/usr/bin/anycubic-ctl`（GTK3），结构类似。

---

## 配套固件

本项目在 Anycubic Mega Zero（ATmega1284P / Melzi）上配套做过以下固件工作，
供同机型参考：

- 升级到社区固件 **Marlin 2.0.9.3-kad1**（DKMS 无关，AVR 直接刷写）
- 修复**倒挂风扇不可控**：启用 `FAN_SOFT_PWM`
  （硬件 PWM 引脚 PD4/OC1B 与步进定时器 Timer1 冲突）
- 中文菜单（173 字子集）与自定义开机 Logo
- 因 128KB Flash 限制裁掉非官方硬件功能（换料、网格调平、断料检测等）

> 固件基于 [Marlin](https://github.com/MarlinFirmware/Marlin)（GPL-2.0）
> 与 [kad/Marlin](https://github.com/kad/Marlin) 的 Mega Zero 配置。

---

## 贡献与许可

欢迎提交 Issue / PR，尤其是**新机型的适配参数**：

```python
{"id": "newmodel", "name": "型号名", "match": r"正则",
 "bed": True, "bmax": 100, "nmax": 260, "size": (220, 220, 250)},
```

代码采用 **MIT License**，可自由使用、修改、分发。
配套固件部分遵循 Marlin 的 **GPL-2.0**。

---

## 免责声明

本工具直接向打印机发送 G-code 与加热指令，**存在设备损坏风险**：

- 加热温度由使用者自行设定，请勿超过耗材与热端的安全范围
- 调平与移动前请确认喷头附近无遮挡、Z 轴有足够余量
- 刷写固件有变砖风险，请务必先备份原厂固件

作者不对因使用本工具造成的任何设备损坏或人身伤害承担责任。
使用即表示你理解并接受这些风险。

---

<p align="center">
  <sub>为不想再开 Windows 虚拟机的 Linux 用户而做</sub>
</p>
