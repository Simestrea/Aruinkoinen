# Aurinko

> C++20 + raylib 的 2D 游戏脚手架。用协程写角色剧本，一条命令生成、编译、打包。

[![C++20](https://img.shields.io/badge/C%2B%2B-20-blue.svg)]()
[![raylib](https://img.shields.io/badge/raylib-5.x-black.svg)]()
[![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg)]()
[![Version](https://img.shields.io/badge/version-0.3.5-red.svg)]()

---
Aurinko 是一个基于 C++20 协程和 raylib 的游戏开发框架。你只需要：

1. 写角色的剧本（一个 `.hpp` 文件）
2. 跑 `aurinko update`
3. 得到可发布的游戏

不需要写 `main`，不需要 CMake，不需要手写状态机。

## 特性

- **协程即剧本**。用 `co_await` 线性地写角色行为。
- **一角色一文件**。每个角色一个 `.hpp`，`update` 时自动扫描、注册。
- **一条命令构建**。`aurinko update` 完成生成、编译、打包。
- **虚拟画布**。逻辑分辨率固定，窗口可任意缩放、全屏，画面比例自动保持。
- **内建 UI 组件**。按钮、输入框、面板、进度条、滑块、折线图、表格。
- **中文字体支持**。字体池自动预加载 9000+ 常用码点。

## 快速上手

### 依赖

- g++，需 C++20 协程（推荐 [w64devkit](https://github.com/skeeto/w64devkit)）
- raylib 5.x

### 安装

详细的安装步骤见 [docs/install.md](docs/install.md)。简版：

1. 把 raylib 的 `raylib.h` 和 `libraylib.a` 放进 w64devkit 的 `include/` 和 `lib/`
2. 编译 CLI：`g++ aurinko.cpp -o aurinko.exe -std=c++20`
3. 把 `aurinko.exe` 放进 w64devkit 的 `bin/`
4. 把 `aurinko/` 目录放进 w64devkit 的 `include/`

### 使用

```bash
# 创建项目
aurinko create

# 创建角色
aurinko create Alice

# 编辑 characters/Alice/Alice.hpp
# 编译
aurinko update

# 运行
release/game.exe
```

### 一个完整角色脚本

```cpp
#pragma once
#include <aurinko/character.hpp>
#include <aurinko/coroutine.hpp>
#include <aurinko/scene.hpp>
#include <aurinko/text.hpp>
#include <aurinko/context.hpp>
#include "globals.hpp"

inline aur::Task AliceFun(aur::Character& c, aur::Context& ctx) {
    c.Say("Hello", 1.5f);
    co_await aur::WaitFor{ 1.5f };

    while (true) {
        if (IsKeyDown(KEY_D)) c.ChangeX( 200 * GetFrameTime());
        if (IsKeyDown(KEY_A)) c.ChangeX(-200 * GetFrameTime());
        c.BounceOffEdge(ctx.screen.GetVirtualWidth(),
                        ctx.screen.GetVirtualHeight());
        co_await aur::WaitNextFrame{};
    }
}
```

把这段放进 `characters/Alice/Alice.hpp`，跑 `aurinko update`，角色就能动。

## 核心概念

| 概念 | 说明 |
|---|---|
| **Character** | 一个角色：精灵图、动画、运动、碰撞、剧本 |
| **Scene** | 场景组。只有当前场景的角色才更新和绘制 |
| **Task / Clone** | 协程返回类型。`Task` 自动运行，`Clone` 需手动 `Spawn` |
| **Context** | 脚本访问引擎资源的唯一入口（`ctx.scene` / `ctx.screen` / `ctx.fonts` / ...） |

## 命令行

```
aurinko create                  创建新项目
aurinko create <角色名>         创建新角色
aurinko addfont <路径> [名字]   导入字体到 fonts/
aurinko update [参数...]        生成并编译
aurinko version                 显示版本号
```

`aurinko update` 支持：
- `-d` / `--debug` — debug 编译
- `-l` / `-I` / `-D` — 追加编译参数

或者把参数写在 `build.txt` 里，每行一条。

## 项目结构

```
your-project/
├── project.txt           画布尺寸
├── build.txt             附加编译参数
├── globals.hpp           全局变量
├── index.cpp             自动生成，别手改
├── characters/
│   └── Alice/
│       ├── Alice.hpp     剧本
│       ├── Alice.png     精灵图
│       └── config.txt    角色配置
├── fonts/
│   └── msyh.ttc
└── release/              可发布的目录
    ├── game.exe
    ├── characters/
    └── fonts/
```

## 库结构

```
aurinko/
├── init.hpp              虚拟画布 + 窗口
├── character.hpp         角色 + 纹理池
├── scene.hpp             场景管理
├── coroutine.hpp         协程调度
├── context.hpp           脚本依赖包
├── text.hpp              文字 + 字体池
├── textbox.hpp           输入框
├── button.hpp            按钮
├── ui_root.hpp           UI 容器
├── panel.hpp             面板
├── progress_bar.hpp      进度条
├── draw.hpp              绘制原语
├── input.hpp             输入查询
└── extras/
    ├── plot.hpp          折线图
    ├── slider.hpp        滑块
    └── table.hpp         表格
```

## 文档

- [安装指南](docs/install.md)
- [API 参考](docs/api.md)

## 平台支持

| 平台 | 状态 |
|---|---|
| Windows (w64devkit) | ✅ 主要支持 |
| Linux | 🚧 部分支持 |
| macOS | ❌ 未测试 |

## 许可

[MIT](LICENSE)

---

*© 2026 Aurinko · C++20 · raylib*

## 1. 下载 w64devkit

从 <https://github.com/skeeto/w64devkit/releases> 下载最新版 `w64devkit-x.x.x.zip`。

解压，把整个 `w64devkit\` 文件夹放到 `C:\w64devkit\`。

目录结构：

```
C:\w64devkit\
├── bin\          (g++.exe, gcc.exe, ...)
├── include\      (头文件)
├── lib\          (库文件)
├── share\
└── w64devkit.exe (启动器)
```

**把 `C:\w64devkit\bin` 加进系统 PATH**：

1. Win + R → 输入 `sysdm.cpl` → 回车
2. "高级" 选项卡 → "环境变量"
3. 系统变量里选中 `Path` → "编辑" → "新建" → 填 `C:\w64devkit\bin`
4. 全部确定，重启 PowerShell

验证：开 PowerShell 输入 `g++ --version`，能看到版本号就行。

---

## 2. 装 raylib

从 <https://github.com/raysan5/raylib/releases> 下载最新的 **raylib-x.x.x_win64_mingw-w64.zip**。

解压后里面是：

```
raylib\
├── include\
│   └── raylib.h
├── lib\
│   ├── libraylib.a
│   └── libraylibdll.a
└── ...
```

**把对应的文件放进 w64devkit**：

| 从 raylib 里复制 | 放到 |
|---|---|
| `raylib\include\raylib.h` | `C:\w64devkit\include\` |
| `raylib\lib\libraylib.a` | `C:\w64devkit\lib\` |

**注意**：

- 如果 `C:\w64devkit\include\` 里已有 `raylib.h`，覆盖它
- 只放 `libraylib.a` 就够了，`libraylibdll.a` 是动态库版本，用不到
- 复制的是**文件本身**，不是整个 `include\` 或 `lib\` 文件夹

完成后：

```
C:\w64devkit\include\raylib.h
C:\w64devkit\lib\libraylib.a
```

---

## 3. 编译并安装 Aurinko CLI

在 Aurinko 源码目录（含 `aurinko.cpp`）：

```bash
g++ aurinko.cpp -o aurinko.exe -std=c++20
```

**把 `aurinko.exe` 复制到 `C:\w64devkit\bin\`**。

完成后：

```
C:\w64devkit\bin\aurinko.exe
```

验证：开新 PowerShell，输入：

```powershell
aurinko version
# 输出：aurinko 0.3.5
```

---

## 4. 安装 Aurinko 头文件

把整个 `aurinko\` 目录（含所有 `.hpp`）**复制到** `C:\w64devkit\include\` 下。

复制的是 `aurinko\` **文件夹本身**，不是里面的文件。

完成后：

```
C:\w64devkit\include\aurinko\
├── character.hpp
├── context.hpp
├── coroutine.hpp
├── draw.hpp
├── init.hpp
├── input.hpp
├── scene.hpp
├── text.hpp
├── textbox.hpp
├── ui_root.hpp
├── button.hpp
├── panel.hpp
├── progress_bar.hpp
├── version.hpp
└── extras\
    ├── plot.hpp
    ├── slider.hpp
    └── table.hpp
```

---

## 5. 目录最终样子

四步做完后，w64devkit 的相关目录应该长这样：

```
C:\w64devkit\
├── bin\
│   ├── g++.exe
│   ├── gcc.exe
│   └── aurinko.exe           ← 第 3 步放进来的
├── include\
│   ├── raylib.h              ← 第 2 步放进来的
│   └── aurinko\              ← 第 4 步放进来的
│       ├── init.hpp
│       ├── character.hpp
│       └── ...
└── lib\
    └── libraylib.a           ← 第 2 步放进来的
```

**另外两个必须手动复制的文件**（w64devkit 里的路径不固定，从你实际找到的位置复制）：

- `libwinpthread-1.dll` — 通常在 `w64devkit\bin\` 里已经有了，不用管
- `libgcc_s_seh-1.dll` / `libstdc++-6.dll` — 通常在 `w64devkit\bin\` 里，不用管

w64devkit 的 DLL 都在 `bin\` 里，编译出的程序依赖它们。如果要把 `game.exe` 拷到别的机器上跑，需要连同这几个 DLL 一起拷。或者用 `-static` 静态链接，就不用带 DLL。

---

## 6. 验证安装

开新 PowerShell，找个空目录：

```powershell
mkdir $env:TEMP\aurinko-test
cd $env:TEMP\aurinko-test

aurinko create
# 画布宽度: 800
# 画布高度: 450

aurinko create Alice
# 图片路径: （留空直接回车）
# 缩放倍数: 2
# ... 全部默认

aurinko update
```

看到 `编译成功 -> game.exe` 就代表全部装好了。

---

## 常见问题

**`g++: command not found`**

PATH 没配好，或没重启终端。Win + R → `sysdm.cpl` 检查系统变量 `Path` 里有没有 `C:\w64devkit\bin`。加完必须**开新终端**才生效。

**`fatal error: raylib.h: No such file or directory`**

raylib.h 没放到正确位置。检查：

```powershell
dir C:\w64devkit\include\raylib.h
```

文件必须**直接**在 `include\` 下，不是 `include\raylib\raylib.h`。

**`fatal error: aurinko/init.hpp: No such file or directory`**

Aurinko 头文件没放对。检查：

```powershell
dir C:\w64devkit\include\aurinko\init.hpp
```

必须是 `include\aurinko\init.hpp` 这种两层结构。

**`undefined reference to 'InitWindow'`**

`libraylib.a` 没放对位置。检查：

```powershell
dir C:\w64devkit\lib\libraylib.a
```

**`aurinko: command not found`**

`aurinko.exe` 没放进 `C:\w64devkit\bin\`。检查：

```powershell
dir C:\w64devkit\bin\aurinko.exe
``

---

## 卸载

删掉这四个位置的文件：

```
C:\w64devkit\bin\aurinko.exe
C:\w64devkit\include\raylib.h
C:\w64devkit\include\aurinko\
C:\w64devkit\lib\libraylib.a
```

然后从系统 PATH 里移除 `C:\w64devkit\bin`，整个删掉 `C:\w64devkit\` 即可。

用户自己的项目目录（`project.txt`、`characters\` 等）不受影响，需要单独删。
