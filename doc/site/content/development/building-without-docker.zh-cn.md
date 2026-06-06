+++
title = '不使用 Docker 的构建'
author = 'p2004a'
translator = 'Cru'
+++

本文介绍了如何在主机上直接编译引擎，而无需使用 Docker。如果您刚开始使用，建议采用 [Docker 方法](/development/building-with-docker/)，以便快速启动和运行。

[docker-build-v2](https://github.com/beyond-all-reason/RecoilEngine/tree/master/docker-build-v2) 文件夹是了解如何调用和配置各项内容的唯一真实来源和最佳参考资料。

## 编译

可以说，Recoil 在 Linux 和 Windows 上的编译在 Linux 端更为方便。本文将仅介绍在 Linux 上的编译方法（或通过 Windows 上的 WSL）。另外，您也可以参考 Wiki 页面 ["使用 MSVC 编译"](https://github.com/beyond-all-reason/RecoilEngine/wiki/Building-on-MSVC)，了解如何设置 Visual Studio 环境，该环境非常适合在 Windows 上进行 C++ 开发。

- Windows：有 [WSL](https://docs.microsoft.com/en-us/windows/wsl/)，运行非常顺畅。  
- Linux：为了避免在基础系统中直接安装所有开发依赖项、编译器等，并为了兼容性，你可以使用 [distrobox](https://github.com/89luca89/distrobox) 在其中进行开发。虽然也可以不使用它，但这种方式非常方便。

此指令已在基于 Debian 的系统和 Arch 系统上测试过，但只要弄清楚需要安装的软件包列表，应该也能在其他所有 Linux 发行版上正常工作。

### 安装系统依赖项

#### 基于 Debian 的系统

编译器、基本实用工具以及开发中常用的工具：

```bash
sudo apt-get install -y cmake g++ ccache ninja-build clang mold git clangd socat \
  pipx g++-mingw-w64-x86-64-posix
pipx install compdb
```

Recoil引擎依赖项

```bash
sudo apt-get install -y libsdl2-dev libdevil-dev libcurl4-openssl-dev \
  p7zip-full libopenal-dev libogg-dev libvorbis-dev libunwind-dev libfreetype-dev \
  libglew-dev libminizip-dev libfontconfig-dev
```

#### Arch 系统

编译器、基本实用工具以及开发所需的工具（Debian 基础发行版已安装的 mingw 跨平台编译器除外）：

```bash
sudo pacman -S base-devel cmake ccache git openssh ninja mold socat clang python-pip
sudo pip install compdb
```

Recoil引擎依赖项

```bash
sudo pacman -S curl sdl2 devil p7zip openal libogg libvorbis libunwind freetype2 glew \
  minizip fontconfig jsoncpp
```

并且确保 openal 有可用的音频后端：

```bash
sudo pacman -S libpulse
```

### 获取源码

```bash
git clone https://github.com/beyond-all-reason/RecoilEngine.git --recursive
cd RecoilEngine
# 适用于 Windows 的编译
git clone https://github.com/beyond-all-reason/mingwlibs64.git mingwlibs64
```

### 编译

这部分最令人烦恼：构建配置使用的是 CMake，而命令行内容相当长。

#### 工具链

首先，你需要配置一些工具链。工具链用于选择编译器和目标操作系统。你可以将它们存储在 Recoil 仓库的 `toolchain` 目录中。Linux 工具链使用 `mold` 链接器，因为它速度更快。

`toolchain/clang_x86_64-pc-linux-gnu.cmake`:

```cmake
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_C_COMPILER "clang")
SET(CMAKE_CXX_COMPILER "clang++")
SET(CMAKE_EXE_LINKER_FLAGS_INIT "-fuse-ld=mold")
SET(CMAKE_MODULE_LINKER_FLAGS_INIT "-fuse-ld=mold")
SET(CMAKE_SHARED_LINKER_FLAGS_INIT "-fuse-ld=mold")
```

`toolchain/gcc_x86_64-pc-linux-gnu.cmake`:

```cmake
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_C_COMPILER "gcc")
SET(CMAKE_CXX_COMPILER "g++")
SET(CMAKE_EXE_LINKER_FLAGS_INIT "-fuse-ld=mold")
SET(CMAKE_MODULE_LINKER_FLAGS_INIT "-fuse-ld=mold")
SET(CMAKE_SHARED_LINKER_FLAGS_INIT "-fuse-ld=mold")
```

`toolchain/gcc_x86_64-pc-windows-gnu.cmake`:

```cmake
SET(CMAKE_SYSTEM_NAME Windows)
SET(CMAKE_C_COMPILER "x86_64-w64-mingw32-gcc-posix")
SET(CMAKE_CXX_COMPILER "x86_64-w64-mingw32-g++-posix")
SET(CMAKE_RC_COMPILER "x86_64-w64-mingw32-windres")
SET(WINDRES_BIN "x86_64-w64-mingw32-windres")
SET(CMAKE_DLLTOOL "x86_64-w64-mingw32-dlltool")
SET(DLLTOOL "x86_64-w64-mingw32-dlltool")
```

#### CMake 命令行

使用 CMake 时，我们是在源代码之外进行构建，因此可以创建类似 `builddir-win` 或 `builddir-dbg` 的目录，在这些目录中运行 CMake 命令。由于可能的配置有很多，这里仅列出一些可用作起点的示例。

在所有这些中：

- 我们使用 Ninja 生成器，因为 Ninja 是执行构建过程、扫描更改等最快速的工具。  
- 使用 ccache 以加快下一次编译速度。  
- 安装目录为 `install`，因此在使用 CMake 配置构建后，只需运行 `ninja && ninja install`，即可将所有文件准备就绪，并存放在构建目录下的 `install` 文件夹中，以便使用。

---

基础版本，包含调试信息，共享库，使用 GCC 构建的 Linux 版本：

```bash
cmake --fresh \
	-DCMAKE_TOOLCHAIN_FILE="../toolchain/gcc_x86_64-pc-linux-gnu.cmake" \
	-DCMAKE_CXX_COMPILER_LAUNCHER=ccache \
	-DCMAKE_C_COMPILER_LAUNCHER=ccache \
	-DCMAKE_CXX_FLAGS_RELWITHDEBINFO="-O3 -g -DNDEBUG" \
	-DCMAKE_C_FLAGS_RELWITHDEBINFO="-O3 -g -DNDEBUG" \
	-DCMAKE_BUILD_TYPE=RELWITHDEBINFO \
	-DCMAKE_COLOR_DIAGNOSTICS=ON \
	-DAI_TYPES=NATIVE \
	-DCMAKE_INSTALL_PREFIX="$(dirname $(realpath "$0"))/install" \
	-G Ninja \
	..
```

使用 Clang 构建的快速未优化调试版 Linux 共享库，并生成一个 [`compile_commands.json`](https://clang.llvm.org/docs/JSONCompilationDatabase.html) 文件。

```bash
cmake --fresh \
	-DCMAKE_TOOLCHAIN_FILE="../toolchain/clang_x86_64-pc-linux-gnu.cmake" \
	-DCMAKE_CXX_COMPILER_LAUNCHER=ccache \
	-DCMAKE_C_COMPILER_LAUNCHER=ccache \
	-DCMAKE_CXX_FLAGS_DEBUG="-O1 -g" \
	-DCMAKE_C_FLAGS_DEBUG="-O1 -g" \
	-DCMAKE_BUILD_TYPE=DEBUG \
	-DCMAKE_COLOR_DIAGNOSTICS=ON \
	-DDEBUG_MAX_WARNINGS=OFF \
	-DAI_TYPES=NATIVE \
	-DCMAKE_INSTALL_PREFIX="$(dirname $(realpath "$0"))/install" \
	-DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
	-G Ninja \
	..
```

使用 mingw64 进行静态交叉编译，带有最少的行级调试信息的 Windows 版本：

```bash
cmake --fresh \
	-DCMAKE_TOOLCHAIN_FILE="../toolchain/gcc_x86_64-pc-windows-gnu.cmake" \
	-DCMAKE_CXX_COMPILER_LAUNCHER=ccache \
	-DCMAKE_C_COMPILER_LAUNCHER=ccache \
	-DCMAKE_BUILD_TYPE=RELWITHDEBINFO \
	-DCMAKE_CXX_FLAGS_RELWITHDEBINFO="-O3 -g1 -DNDEBUG" \
	-DCMAKE_C_FLAGS_RELWITHDEBINFO="-O3 -g1 -DNDEBUG" \
	-DCMAKE_COLOR_DIAGNOSTICS=ON \
	-DAI_TYPES=NATIVE \
	-DCMAKE_INSTALL_PREFIX="$(dirname $(realpath "$0"))/install" \
	-GNinja ..
```

---

### 源代码补全

在上述 clang 调试中，我们还启用了生成 `compile_commands.json` 文件的功能，该文件之后可被我认为最优秀的 C++ 语言服务器 [clangd](https://clangd.llvm.org/) 使用。CMake 生成的编译数据库的主要问题是其不包含头文件的条目。这个问题可以通过之前安装的 [compdb](https://github.com/Sarcasm/compdb) 工具来解决。从顶级仓库目录运行：

```bash
compdb -p builddir-clang/ list > compile_commands.json
```

Clangd 然后就会自动处理它。

#### Visual Studio Code

我们推荐使用 [clangd](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd) 扩展。如果你在 distrobox 中开发，可以在 https://distrobox.it/posts/integrate_vscode_distrobox/ 查看多种配置选项的说明。

#### 其他支持 Distrobox 的 IDE

如果在 distrobox 中运行，且 distrobox 内使用 clangd，同时主机上直接有 IDE，您可能需要在支持 LSP 的编辑器中覆盖 clangd 的调用方式，例如：`distrobox enter --no-tty {container_name} -- socat tcp-listen:{port},reuseaddr exec:clangd`

在 Windows 上开发时，使用 Linux 上的 clangd，并将 WSL 容器映射到 `L:` 驱动器，命令可能如下所示：`wsl.exe socat tcp-listen:${port},reuseaddr exec:'clangd --path-mappings=L:/home=/home,L:/usr=/usr'`。唯一的缺点是，在这种设置下，当然 `#ifdef _WIN32` 的条件编译块将无法完成提示。

适用于 Linux 上 Sublime Text 的 `recoil.sublime-project` 示例，使用 [LSP](https://lsp.sublimetext.io/) 服务器包：

```json
{
	"folders": [
		{
			"path": "."
		}
	],
	"settings": {
		"tab_size": 4,
		"LSP": {
			"clangd": {
				"enabled": true,
				"command": [
					"distrobox",
					"enter",
					"--no-tty",
					"recoil",
					"--",
					"socat",
					"tcp-listen:{port},reuseaddr",
					"exec:clangd"
				],
				"tcp_port": 0,
				"scopes": ["source.c", "source.c++"],
				"syntaxes": [
					"Packages/C++/C.sublime-syntax",
					"Packages/C++/C++.sublime-syntax"
				],
				"languageId": "cpp"
			}
		}
	}
}
```
