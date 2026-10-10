# 安装（Windows）
Aurinko 由三部分组成，缺一不可：

| 组件 | 内容 | 干什么用 |
|---|---|---|
| **CLI 工具** | 一个可执行文件 `aurinko.exe` | 生成项目 / 编译 / 打包 |
| **头文件库** | `aurinko\` 目录（10+ 个 `.hpp`） | 你项目里 `#include <aurinko/...>` 用的 |
| **raylib** | 第三方库 | 渲染 / 输入 / 音频 |

推荐使用 [MSYS2](https://www.msys2.org/) 里的 **MinGW-w64** 工具链。它是 Windows 上跑 g++ + raylib 最省事的方案。

---

## 1. 装 MSYS2

从 <https://www.msys2.org/> 下载安装包，装到默认位置（比如 `C:\msys64`）。

装完打开 **MSYS2 MinGW 64-bit** 这个快捷方式（**不是** "MSYS2 MSYS"）。之后所有命令都在这个终端里跑。

> **怎么分辨**：提示符应该是 `user@host MINGW64 ~` 而不是 `user@host MSYS ~`。如果是后者，说明开错了终端。

---

## 2. 装 g++ 和 raylib

在 MinGW64 终端里：

```bash
pacman -S mingw-w64-x86_64-gcc
pacman -S mingw-w64-x86_64-raylib
```

验证：

```bash
g++ --version
# 应输出 gcc 13 或更新版本

ls /mingw64/lib/libraylib.a
# 文件存在说明 raylib 装好了
```

---

## 3. 编译 CLI 并放到 PATH

在 Aurinko 源码目录（含 `aurinko.cpp` 的目录）：

```bash
g++ aurinko.cpp -o aurinko.exe -std=c++20
```

把它复制到 MinGW64 的 `bin` 目录（这个目录已经在系统 PATH 里）：

```bash
cp aurinko.exe /mingw64/bin/
```

验证：

```bash
aurinko version
# 输出：aurinko 0.3.5
```

> **另一种做法**：把 `aurinko.exe` 放到任意目录（例如 `D:\tools\`），然后在 Windows 系统设置里把 `D:\tools\` 加进 `PATH`。效果一样。

---

## 4. 安装头文件库

把 `aurinko\` 目录（含所有 `.hpp`）复制到 MinGW64 的 include 目录：

```bash
cp -r aurinko/ /mingw64/include/aurinko/
```

完成后的目录结构：

```
C:\msys64\mingw64\include\aurinko\
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

`/mingw64/include` 是 g++ 的默认搜索路径之一。装完后，任何项目里写 `#include <aurinko/init.hpp>` 都能找到。

---

## 5. 验证安装

开一个空目录测试：

```bash
mkdir /tmp/aurinko-test
cd /tmp/aurinko-test
aurinko create
```

按提示填画布尺寸（比如 `800` 和 `450`）。如果没报错并生成 `project.txt`、`characters\`、`fonts\`，说明 CLI 装好了。

再试一次完整流程：

```bash
aurinko create Alice
aurinko update
```

看到 `编译成功 -> game.exe` 就说明 CLI、头文件、raylib 三者全部就位。

---

## 卸载

删掉这两个位置的文件即可：

```bash
rm /mingw64/bin/aurinko.exe
rm -rf /mingw64/include/aurinko/
```

g++ 和 raylib 是 pacman 装的，用：

```bash
pacman -R mingw-w64-x86_64-raylib
pacman -R mingw-w64-x86_64-gcc
```

如果不打算再用 MSYS2，直接在"添加或删除程序"里卸载 MSYS2 整个。

---

## 常见问题

**`aurinko: command not found`**

CLI 不在 PATH 里。在 MinGW64 终端里检查：

```bash
echo $PATH | tr ':' '\n' | grep mingw64
```

应该能看到 `/mingw64/bin`。看不到说明你开错了终端（开成了 "MSYS2 MSYS" 而不是 "MSYS2 MinGW 64-bit"），或者你根本没把 `aurinko.exe` 放进 `/mingw64/bin/`。

**`fatal error: aurinko/init.hpp: No such file or directory`**

头文件没装到位。检查文件是否真的在：

```bash
ls /mingw64/include/aurinko/init.hpp
```

如果文件不存在，重新执行第 4 步的 `cp`。

**`undefined reference to 'InitWindow'`**

raylib 没装好，或者你用的 g++ 不是 MinGW64 里的那个。检查：

```bash
which g++
# 应该输出 /mingw64/bin/g++ 之类
```

如果不是，说明你误用了 MSYS2 MSYS 环境里的 g++。

**`'coroutine' file not found`**

g++ 版本太老。检查：

```bash
g++ --version
```

如果低于 13，跑：

```bash
pacman -Syu
pacman -S mingw-w64-x86_64-gcc
```

升级到最新。

**双击 `aurinko.exe` 一闪而过**

`aurinko` 是命令行工具，必须在终端里跑：

```bash
aurinko create
```

不要双击。在项目目录开一个 MinGW64 终端，或 PowerShell 里 `cd` 到项目目录后执行。PowerShell 里如果直接跑不起来（因为 PATH 没刷新），可以先执行一次 `refreshenv`，或者重启 PowerShell。

**在 PowerShell / CMD 里也能用吗？**

可以。只要 `C:\msys64\mingw64\bin` 加进了系统 PATH（MSYS2 安装时会问是否加），PowerShell 和 CMD 里也能用 `aurinko`。

如果刚装完没生效，重启一下 PowerShell，或运行：

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```
