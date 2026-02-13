# FreeRDP-Headless-AutoInput

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[中文说明 (Chinese)](README_zh.md)

**FreeRDP-Headless-AutoInput** is a deeply modified version of the [FreeRDP](https://github.com/FreeRDP/FreeRDP) Windows client (`wfreerdp.exe`). It transforms the standard RDP client into a **headless, stealthy, and automated command injection tool** for RDP sessions.

## 📖 Introduction

The original `wfreerdp.exe` is designed to display a remote desktop GUI. This project hijacks the main loop and input context to achieve:

1.  **Headless / Stealth Mode**: Removes all GUI rendering logic. The client runs silently in the background without creating a visible window, significantly reducing resource usage.
2.  **Automated Input Injection**: Once the connection is established, it automatically sends `Win+R` and injects pre-configured commands (e.g., executing a specific script) without human intervention.
3.  **Session Keep-Alive**: Prevents the remote session from locking or disconnecting due to inactivity by sending invisible keystrokes (F15).

## 🚀 Core Modifications & Technical Principles

### 1. Automated Execution Thread (`wf_automation_thread`)
This is the core of the automation. We leverage the `rdpContext` pointer exposed by FreeRDP to call internal APIs directly, simulating keyboard events programmatically.

*   **Mechanism**: Starts a dedicated thread upon successful connection.
*   **Workflow**:
    1.  Forcefully takes input focus (`wf_event_focus_in`).
    2.  Sends `Win+R` to open the Run dialog.
    3.  Converts the payload string (e.g., file path) into Scancodes and injects them.
    4.  Sends `Enter` to execute.

### 2. Smart Keep-Alive (`wf_keep_alive_thread`)
Prevents the server from disconnecting the session due to "user inactivity".

*   **Logic**: Sends the **F15 (VK_F15)** key to the server periodically.
*   **Why F15**: F15 is a valid key in the RDP protocol but does not exist on standard keyboards. It is recognized as "user activity" by the server, resetting the idle timer, but triggers no visible action in Windows (unlike Shift or NumLock).

### 3. Stealth & Optimization
To achieve "headless" operation, window creation and rendering logic were refactored:

*   **Message-Only Window**: The window type is changed to `HWND_MESSAGE`. This type of window handles the message loop but has no Z-order, is invisible, and does not appear on the taskbar.
*   **Disabled GDI Rendering**: Calls to `BitBlt` and `InvalidateRect` inside `wf_end_paint` are commented out. Even if the server sends graphical data, the client skips decoding and drawing, minimizing CPU/GPU usage.

## 🛠️ Usage & Compilation

### 1. Prerequisites
You need the FreeRDP source code and a Windows build environment.
*   👉 **[Detailed Windows Compilation Guide](BUILD_WINDOWS.md)** (Mirrored from external source)

### 2. Replace Source File
Replace the original `wf_client.c` in the FreeRDP source tree with the one provided in this repository.
*   Path: `client/Windows/wf_client.c`

### 3. Configure Payload (Important)
Before compiling, open `wf_client.c`, locate the `wf_automation_thread` function, and modify the command you want to inject:

```c
// Find this inside wf_client.c
// Modify 'command' to your desired path or command
const char* command = "Z:\\Debug\\QueryProxy.exe";
```

4. Build
Use CMake to generate the Visual Studio project and build the wfreerdp target.
5. Run
The compiled wfreerdp.exe supports all original command-line arguments.
code
Bash
wfreerdp.exe /v:192.168.1.100 /u:admin /p:password
Upon running, no window will appear, but the remote machine will receive the input automatically.
⚠️ Disclaimer
This project is intended for security research, protocol analysis, and authorized automated operations only.
Do not use this tool for unauthorized access or malicious attacks.
The author assumes no responsibility for any legal consequences or system damage caused by the use of this project.
This project is open-sourced under the Apache 2.0 License (following FreeRDP).
🔗 References
FreeRDP GitHub Repository
Microsoft RDP Protocol Specification
