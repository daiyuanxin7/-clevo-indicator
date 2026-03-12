# Clevo 风扇控制指示器

适用于 Ubuntu 的 Clevo（蓝天）笔记本风扇控制工具，在系统托盘显示温度和风扇信息，支持自动调速和手动控制。

原项目地址：https://github.com/SkyLandTW/clevo-indicator

---

## 更新日志

### 2026-03-12

**新增**
- GPU 风扇控制：现在同时控制 CPU 风扇（EC 端口 `0x01`）和 GPU 风扇（EC 端口 `0x02`），两个风扇始终保持相同转速
- 新增 GPU 风扇转速读取（寄存器 `0xD2/0xD3`）
- 托盘图标动画改为 CPU 和 GPU 风扇转速的平均值

**修复**
- 自动调速模式静默失败：原最小值校验为 60%，但自动算法可能返回 30/40/50%，导致风扇转速无法降低，现已修复
- `MAX_FAN_RPM` 从 4400 调整为 5500，与实际硬件匹配

### 历史版本

- 适配 Ubuntu 22+：从 `appindicator3` 迁移至 `ayatana-appindicator3`

---

## 功能

- 系统托盘实时显示 CPU 和 GPU 温度
- 托盘图标动态反映风扇转速（转速越高图标越满）
- 自动调速模式：根据 CPU/GPU 温度自动调节风扇占空比，取两者较高温度为基准
- 手动模式：通过托盘菜单直接设置风扇占空比（60% ~ 100%）
- CPU 风扇与 GPU 风扇始终同步，统一控制

## 自动调速算法

| 温度 | 目标占空比 |
|------|-----------|
| ≥ 80°C | 100% |
| ≥ 70°C | 90% |
| ≥ 60°C | 80% |
| ≥ 50°C | 70% |
| ≥ 40°C | 60% |
| ≥ 30°C | 50% |
| ≥ 20°C | 40% |
| ≥ 10°C | 30% |

降温时有约 5°C 的滞后区间，防止风扇频繁抖动。

---

## 安装

### 依赖

```bash
sudo apt-get install libayatana-appindicator3-dev libgtk-3-dev
```

### 编译并安装

```bash
git clone https://github.com/daiyuanxin7/-clevo-indicator.git
cd -clevo-indicator
make
make install   # 安装到 /usr/local/bin/，并自动设置 setuid 权限
```

### 开机自启动

```bash
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/clevo-indicator.desktop << EOF
[Desktop Entry]
Type=Application
Name=Clevo Fan Control
Exec=clevo-indicator
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
EOF
```

---

## 使用

```bash
# 启动托盘指示器（需要桌面环境）
clevo-indicator

# 查看当前风扇和温度信息
clevo-indicator -?

# 直接设置风扇占空比（40% ~ 100%）
clevo-indicator 80
```

---

## 注意事项

- 程序使用 setuid=root，但必须由桌面用户启动。内部通过 fork 分成两个进程：子进程以 root 权限操作 EC，父进程以普通用户权限显示托盘图标
- 运行期间请勿使用其他程序通过 `inb/outb` 访问 EC 端口
- 不要用 `kill -9` 强制终止，否则可能导致 EC 命令中断。程序已捕获大多数终止信号以安全退出
- GPU 风扇转速寄存器（`0xD2/0xD3`）在不同 Clevo 机型上可能不同，如读数异常请通过以下命令确认：
  ```bash
  sudo modprobe ec_sys
  sudo od -Ax -t x1 /sys/kernel/debug/ec/ec0/io
  ```
