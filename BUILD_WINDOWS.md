# Windows Compilation Guide (Windows 编译指南)

> ⚠️ **Attribution & Disclaimer / 转载声明**
>
> This document is a mirror/transcription of an external tutorial, intended to ensure the build instructions remain accessible. All credit and copyright belong to the original author.
> 本文档是外部教程的存档/转载，旨在防止原链接失效并方便查阅。所有内容版权和解释权归原作者所有。
>
> - **Original Title / 原文标题**: [FreeRDP源码编译(Windows)](https://www.cnblogs.com/brave-x/p/17688574.html)
> - **Original Author / 原作者**: [brave-x](https://www.cnblogs.com/brave-x/)
> - **Source URL / 原文链接**: [https://www.cnblogs.com/brave-x/p/17688574.html](https://www.cnblogs.com/brave-x/p/17688574.html)
>
> *If you are the original author and wish for this content to be removed, please open an issue in this repository.*
> *如果原作者不希望内容被转载，请在本项目提交 Issue 联系删除。*

---

## 1. 前言

FreeRDP 是远程桌面协议（RDP）的一个开源实现版本，对于我们的学习和使用都有很大的帮助作用。最近由于需要 RDP 协议的 Windows 客户端，所以打算在 Windows 上编译下 FreeRDP。

## 2. 编译前的准备工作

### 2.1. Visual Studio 2019
Visual Studio 是 Windows 平台常用的开发 IDE 工具，本次编译的主要开发工具就是 Visual Studio，使用的是 2019 版本（其他新版本大同小异）。

### 2.2. CMake
本次编译需要用到 CMake 程序用于生成 Visual Studio 的开发工程文件。
本次使用的是 CMake 3.27.1 版本。
*   下载地址：[https://cmake.org/download/](https://cmake.org/download/)

### 2.3. libusb
libusb 是编译 FreeRDP 时会用到的头文件和 lib 库文件。本次编译使用的是 v1.0.24 版本。
*   下载地址：[https://github.com/libusb/libusb/releases/tag/v1.0.24](https://github.com/libusb/libusb/releases/tag/v1.0.24) (下载 `libusb-1.0.24.7z` 文件即可)

![](images/o_230908123950_2.3-libusb.png)

### 2.4. openssl
openssl 也是编译需要使用的库。
*   下载地址：[https://slproweb.com/products/Win32OpenSSL.html](https://slproweb.com/products/Win32OpenSSL.html)
*   选择 `Win64 v1.1.1` 版本 (例如 `Win64OpenSSL-1_1_1v.msi`)。

![](images/o_230908123951_2.4-openssl版本文件.png)

安装后，需要在环境变量中新增系统变量（以本地配置为例）：

```cmd
OPENSSL_INCLUDE_DIR = D:\Program Files\OpenSSL-Win64\include
OPENSSL_ROOT_DIR    = D:\Program Files\OpenSSL-Win64
```
2.5. FreeRDP 源码
下载 FreeRDP 最新版本的源码。
下载地址：https://github.com/FreeRDP/FreeRDP
3. 编译 zlib 库
因为最新的 FreeRDP 把 zlib 库当做必选项，所以我们需要 zlib 库的头文件和库文件。我们需要先编译一下 zlib 库。
3.1. zlib 库下载
下载地址：https://github.com/madler/zlib/releases (使用最新的 zlib 1.3 版本)
![alt text](images/o_230908123950_3.1-zlib库下载.png)
3.2. 编译 zlib 库
采用 CMake 生成 VS 工程文件。
配置源代码路径和 build 路径。
![alt text](images/o_230908123950_3.2-zlib编译.png)
点击 Configure 按钮，开始生成配置。
![alt text](images/o_230908123950_3.2-zlib编译1.png)
点击 Generate 生成工程文件，然后点击 Open Project 打开工程文件。
![alt text](images/o_230908123950_3.2-zlib_generate.png)
3.3. 安装 zlib 库
点击 Open Project 后，会自动打开 VS2019，找到 INSTALL 项目。
右键选择：设为启动项目。
![alt text](images/o_230908123950_3.3.-zlib_vs编译1.png)
右键点击 生成。
![alt text](images/o_230908123950_3.3-zlib_vs生成.png)
生成成功详情：
![alt text](images/o_230908123950_3.3-zlib_vs生成详情.png)
注：目前是生成在 C:\Program Files\zlib\ 路径下，如果是其他路径，则可能需要加入到环境变量中，视具体情况而定。
4. 使用 CMake 生成 FreeRDP 的 VS 解决方案
4.1. 配置源码和 build 路径
打开 CMake 软件，配置 FreeRDP 源码和 build 路径。
Source code: 本地源码路径
Build binaries: 源码路径下的 build 目录
![alt text](images/o_230908124020_4.1-cmake创建.png)
点击 Configure 按钮，开始生成配置：
![alt text](images/o_230908124020_4.1-cmake第一次配置.png)
4.2. 配置 LIBUSB 路径
把下载的 libusb-1.0.24 拷贝到 FreeRDP 目录下（其中 include 目录和 VS2019 是我们要用到的目录）。
![alt text](images/o_230908124020_4.2-libusb目录.png)
在 CMake 中搜索 LIBUSB，找到对应的选项进行配置。
![alt text](images/o_230908124020_4.2-cmake_libusb配置.png)
配置完成后的值：
![alt text](images/o_230908124020_4.2-cmake_libusb配置完成.png)
4.3. 处理配置错误
4.3.1. zlib 库找不到错误
第一次生成配置可能会报错，按照第三章步骤生成 zlib 库后，重新 Configure 即可消除该错误。
![alt text](images/o_230908124020_4.3.1-cmake_zlib配置错误1.png)
4.3.2. openssl 库未配置环境变量错误
如果出现如下错误，请检查 2.4 节中的环境变量是否配置正确。配置好后需重启 CMake 以加载环境变量。
![alt text](images/o_230908124020_4.3.2-cmake_openssl错误.png)
4.3.3. 去掉不需要的模块
如果出现 FFmpeg 找不到的错误，由于我们用不到 FFmpeg 模块，可以在编译选项中去掉。
![alt text](images/o_230908124021_4.3.3-cmake_ffmpeg错误.png)
搜索 WITH_FFMPEG，去掉勾选。
![alt text](images/o_230908124020_4.3.3-cmake_with_ffmpeg.png)
类似的模块还有 WITH_SWSCALE、WITH_AAD 等，如果报错均可去掉勾选。WITH_CLIENT_SDL 可以保留。去掉模块后再次点击 Configure。
4.4. 生成工程文件
Configure 无误后，点击 Generate，然后点击 Open Project。
![alt text](images/o_230908124054_4.4-cmake_generate.png)
4.5. 配置工程文件
打开 VS 工程后，需要在 wfreerdp-client (或 freerdp-client) 项目中添加对 libusb 的依赖。
在 Client -> Common 下找到 freerdp-client 项目。
右键选择属性 -> 链接器 -> 输入 -> 附加依赖项。
增加 libusb-1.0.lib 的路径配置（可以使用绝对路径或相对路径）。
![alt text](images/o_230908124055_4.5-freerdp-vsproject-配置1.png)

![alt text](images/o_230908124055_4.5-freerdp-vsproject-配置2.png)

![alt text](images/o_230908124054_4.5-freerdp-vsproject-配置3.png)
4.6. 编译 & 运行程序
找到 wfreerdp-client 项目，右键选择 生成。
在 build\Debug 目录下会生成 wfreerdp.exe。如果运行需要，请将 libusb-1.0.dll 和 zlibd.dll 拷贝到 Debug 目录下。
![alt text](images/o_230908124054_4.6-freerdp-vsproject-生成.png)
5. 结语
以上就是 Windows 下编译 FreeRDP 的步骤。
