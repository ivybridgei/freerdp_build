# FreeRDP-Headless-AutoInput

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[English Version](README.md)

**FreeRDP-Headless-AutoInput** 是基于 [FreeRDP](https://github.com/FreeRDP/FreeRDP) 开源项目的深度修改版本。本项目通过修改 Windows 客户端入口文件 (`wf_client.c`)，将标准的 RDP 客户端改造为一个**无头（Headless）、静默运行、具备自动化命令执行与连接保活功能**的 RDP 自动化工具。

## 📖 项目简介

原版的 `wfreerdp.exe` 是一个用于显示远程桌面的 GUI 程序。本项目通过劫持其主循环和输入上下文，实现了以下功能：

1.  **无界面/隐身模式 (Headless Mode)**：移除所有图形渲染逻辑，程序在后台静默运行，不创建可见窗口，极大降低资源占用。
2.  **自动化按键注入 (Automated Input Injection)**：连接建立后，自动发送 `Win+R` 并输入预设指令（如运行特定脚本），无需人工干预。
3.  **连接保活 (Session Keep-Alive)**：通过发送隐形按键（F15）防止远程会话因超时而断开。

## 🚀 核心修改与技术原理

### 1. 自动化执行线程 (`wf_automation_thread`)
这是实现 RDP 自动化的核心。我们利用 FreeRDP 暴露的 `rdpContext` 指针，直接调用内部 API 模拟按键事件，而无需物理键盘参与。

*   **机制**：连接成功后，启动独立线程。
*   **流程**：
    1.  强制接管输入焦点 (`wf_event_focus_in`)。
    2.  发送 `Win+R` 组合键呼出运行框。
    3.  将预设的 Payload 字符串（如路径）转换为 Scancode 序列并发送。
    4.  发送回车键执行。

### 2. 智能保活机制 (`wf_keep_alive_thread`)
防止远程服务器因“用户无操作”而锁定屏幕或断开连接。

*   **原理**：利用 RDP 协议的特性，向服务端发送 **F15 (VK_F15)** 按键。
*   **优势**：F15 是标准键盘上不存在但在协议中有效的按键。它会被服务端识别为“用户活动”从而重置空闲计时器，但不会对正在运行的程序产生任何干扰（不同于发送 Shift 或 NumLock）。

### 3. 隐身与资源优化
为了实现“无头”运行，我们重构了窗口创建和绘图逻辑：

*   **Message-Only Window**：将窗口类型修改为 `HWND_MESSAGE`。这种窗口仅用于处理消息循环，没有 Z 序，不可见，且不接收广播消息。
*   **禁用 GDI 渲染**：注释掉了 `wf_end_paint` 中的 `BitBlt` 和 `InvalidateRect` 调用。即使服务器发送了画面数据，客户端也不会进行任何解码和绘制，显著降低了 CPU/GPU 占用。

## 🛠️ 使用与编译指南

### 1. 准备环境
你需要下载 FreeRDP 源码并配置 Windows 编译环境。
*   👉 **[详细的 Windows 编译教程请点击这里](BUILD_WINDOWS.md)** (转载自外部博客)

### 2. 替换源码
将本项目中的 `wf_client.c` 替换 FreeRDP 源码目录下的对应文件：
*   路径通常位于：`client/Windows/wf_client.c`

### 3. 配置 Payload (重要)
在编译前，请打开 `wf_client.c`，找到 `wf_automation_thread` 函数，修改你要执行的远程命令：

```c
// 在 wf_client.c 中找到此处
// 修改 command 变量为你想要在远程机器上自动输入的路径或命令
const char* command = "Z:\\Debug\\QueryProxy.exe";
```

### 4. 编译
使用 CMake 生成 Visual Studio 工程并编译 `wfreerdp` 目标。

### 5. 运行
编译生成的 `wfreerdp.exe` 支持原版所有命令行参数。

```bash
wfreerdp.exe /v:192.168.1.100 /u:admin /p:password
```

*运行后，你将看不到任何窗口，但远程机器会自动接收到按键输入。*

## ⚠️ 免责声明 (Disclaimer)

本项目仅供**安全研究、协议分析及授权的自动化运维测试**使用。

*   请勿将此工具用于未经授权的系统访问或攻击行为。
*   作者不对使用本项目造成的任何法律后果或系统损坏承担责任。
*   本项目基于 Apache 2.0 协议开源（遵循 FreeRDP 原协议）。

## 🔗 参考资料

*   [FreeRDP GitHub Repository](https://github.com/FreeRDP/FreeRDP)
*   [Microsoft RDP Protocol Specification](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpbcgr/5073f4ed-1e93-45e1-b039-6e30c385867c)
