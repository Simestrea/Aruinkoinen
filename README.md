# 安装（Windows · w64devkit）

[w64devkit](https://github.com/skeeto/w64devkit) 是一个免安装的 MinGW-w64 工具链，解压就能用。以下步骤全部围绕它。

## 装什么

| 组件 | 放到哪 |
|---|---|
| **w64devkit** | 解压到 `C:\w64devkit\` |
| **raylib** | 头文件和库塞进 w64devkit |
| **Aurinko CLI** | 编译出的 `aurinko.exe` 塞进 w64devkit |
| **Aurinko 头文件** | 整个 `aurinko\` 目录塞进 w64devkit |

装完一次性搞定，以后不再动。

---

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
```

**双击 `aurinko.exe` 一闪而过**

它是命令行工具，必须在终端里运行。不要双击。

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
