# 快速部署

## 依赖

| 依赖 | 版本 |
|---|---|
| g++ | 需支持 C++20 协程（MinGW-w64 GCC 13+，Linux GCC 12+） |
| raylib | 5.x，能链接到 `-lraylib` |

Windows 用户推荐 [MSYS2](https://www.msys2.org/) 里的 MinGW-w64 工具链。

## 编译脚手架

在 Aurinko 源码目录：

```bash
g++ aurinko.cpp -o aurinko -std=c++20
```

产物是 `aurinko`（Windows 上是 `aurinko.exe`）。放到 PATH 里，或直接用 `./aurinko`。

## 创建项目

新建一个空目录作为项目根目录：

```bash
mkdir my-game
cd my-game
aurinko create
```

会依次问：

```
画布宽度: 800
画布高度: 450
```

生成：

```
my-game/
├── project.txt
├── build.txt
├── globals.hpp
├── characters/
└── fonts/
```

## 创建角色

```bash
aurinko create Alice
```

会依次问：

```
图片路径: /path/to/sprite.png     （支持把文件拖进终端）
缩放倍数 (默认 2.0): 2
精灵图帧数 (默认 10): 10
每帧持续秒数 (默认 0.15): 0.15
场景编号 (默认 0): 0
```

生成：

```
characters/Alice/
├── Alice.hpp      # 剧本，你改这个
├── Alice.png      # 精灵图
└── config.txt     # 角色配置
```

> 图片路径可以直接拖拽文件到终端，会自动去掉引号。没有图片就留空，稍后手动放进去。

## 写剧本

编辑 `characters/Alice/Alice.hpp`：

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

**两条规则**：

1. 函数名必须是 `<角色名>Fun`。目录叫 `Alice`，函数就叫 `AliceFun`。
2. `while(true)` 里必须有 `co_await`，否则这一帧永远结束不了。

## 导入字体（可选）

需要显示中文时：

```bash
aurinko addfont C:/Windows/Fonts/msyh.ttc
```

或把 `.ttf / .ttc / .otf` 文件直接放进 `fonts/`。

引用：

```cpp
aur::Text title("你好", 10, 10, 24, BLACK, "fonts/msyh.ttc");
```

## 编译运行

```bash
aurinko update
```

输出：

```
已生成 index.cpp (1 个角色, 1 个场景)
正在编译...
  g++ index.cpp -o game -I. -mwindows -lraylib -lwinmm -lgdi32 -luser32 -std=c++20 -O2 -s
编译成功 -> game.exe
正在生成 release/ ...
release/ 已生成
```

双击 `release/game.exe` 运行（Windows）或 `./release/game`（Linux）。

## 日常迭代

改完 `characters/*/` 下的任何文件后：

```bash
aurinko update
```

就这一步。生成、编译、打包全做完。

## 可选：配置编译参数

`build.txt` 里每行一条编译参数：

```
-lcurl
-lpthread
-I./thirdparty
-DDEBUG_MODE
```

`aurinko update` 会自动读取并拼接。

或者临时在命令行传：

```bash
aurinko update -lcurl -lpthread
```

## 可选：调试模式

```bash
aurinko update -d
```

- 加 `-g -O0`，保留符号
- 不隐藏终端，方便看日志
- 跳过 `release/` 打包

用 gdb 调试：

```bash
gdb game.exe
```

## 目录一览

```
my-game/
├── project.txt              画布尺寸
├── build.txt                附加编译参数
├── globals.hpp              全局变量
├── index.cpp                自动生成，别手改
├── game.exe                 编译产物
├── characters/
│   └── Alice/
│       ├── Alice.hpp        剧本
│       ├── Alice.png        精灵图
│       └── config.txt       角色配置
├── fonts/
│   └── msyh.ttc
└── release/                 可直接分发
    ├── game.exe
    ├── characters/
    └── fonts/
```

## 发布

`release/` 整个目录压缩发给别人即可。里面只有运行需要的文件（exe + 角色资源 + 字体），没有源码。
