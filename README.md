# Clevo Fan Control Indicator for Ubuntu (Updated for Ubuntu 22+)

这个项目已经9年没有更新了，这是一个用于控制Clevo（蓝天主板）笔记本的风扇工具。在新的Ubuntu版本里面已经无法安装运行了，因为新版本Ubuntu使用的是Ayatana AppIndicator而不是传统的libappindicator3。

我修改了源码让它适配Ubuntu 22+运行。

原项目地址：https://github.com/SkyLandTW/clevo-indicator

## 更新日志

### 2026-03-12

**新增**
- GPU 风扇控制：现在同时控制 CPU 风扇（EC 端口 `0x01`）和 GPU 风扇（EC 端口 `0x02`），两个风扇始终保持相同转速
- 新增 GPU 风扇转速读取（寄存器 `0xD2/0xD3`）
- 托盘图标动画改为 CPU 和 GPU 风扇转速的平均值

**修复**
- 自动调速模式静默失败：原最小值校验为 60%，但自动算法可能返回 30/40/50%，导致风扇转速无法降低，现已修复
- `MAX_FAN_RPM` 从 4400 调整为 5500，与实际硬件匹配

### Ubuntu 22+ 兼容性更新

- 从 `appindicator3` 迁移至 `ayatana-appindicator3`，支持 Ubuntu 22 及以上版本

---

## 功能说明

这个程序是一个Ubuntu指示器，用于控制Clevo笔记本的风扇，使用从ECView逆向工程得到的端口信息。

它在指示器中显示CPU温度（左）和GPU温度（右），并提供手动控制菜单。

## 修改内容

为了适配Ubuntu 22+，进行了以下修改：

1. 将源代码中的头文件从 `#include <libappindicator/app-indicator.h>` 更改为 `#include <libayatana-appindicator/app-indicator.h>`
2. 更新Makefile中的pkg-config配置，从 `appindicator3-0.1` 更改为 `ayatana-appindicator3-0.1`
3. 更新相关的编译配置以支持Ayatana AppIndicator库

## 构建和安装

```shell
# 安装依赖（Ubuntu 22+）
sudo apt-get install libayatana-appindicator3-dev libgtk-3-dev

# 编译
cd clevo-indicator
make install

# 设置权限（可选）
sudo make test

# 安装（可选）
sudo make install
```

## 使用方法

运行程序(默认安装到/usr/local/bin/)：
```shell
/usr/local/bin/clevo-indicator
```

命令行使用，使用 `-h` 显示帮助，或输入数字（40%到100%）来控制风扇转速百分比。

## 注意事项

该可执行文件具有setuid标志，但必须由当前桌面用户运行，因为只有桌面用户才能在Ubuntu中显示桌面指示器，而非root用户不允许通过低级IO端口控制Clevo EC。setuid=root创建了一种特殊情况，该程序可以分叉自己并在两个用户下运行（一个用于桌面/指示器，另一个用于EC控制）。

请注意不要同时使用其他程序通过低级IO系统调用（inb/outb）访问EC。程序还试图通过捕获除SIGKILL之外的所有终止信号来防止在发出命令时中止 - 除非绝对必要，否则不要使用"kill -9"杀死指示器。
