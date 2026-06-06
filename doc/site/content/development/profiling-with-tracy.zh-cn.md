+++
title = 'Tracy 性能分析'
author = 'beherith'
translator = 'Cru'
+++

## 使用预构建的二进制文件：

1. 下载 Tracy，可将其解压到任意位置：https://github.com/wolfpld/tracy/releases/tag/v0.9.1

2. 从 https://engine-builds.beyondallreason.dev/index.html 获取我们当前使用的精确引擎版本

3. 找到您当前使用的精确引擎版本文件夹（例如，对于 BAR 来说就是 `BAR/data/engine`）

4. A. 将旧的 `spring.exe` 重命名为 `spring_vanilla.exe`，并将 `spring.exe` 从压缩包中解压到此引擎文件夹。

5. A. 按照通常方式通过启动器开始游戏。

6. 启动 `Tracy.exe`，然后点击连接。如果出现仪器错误，请重新连接。

![image](https://github.com/beyond-all-reason/RecoilEngine/assets/109391/830e5c6e-b37f-48ab-9adc-cc297cefff46)

7. 分析性能数据。

## 支持Tracy的构建

以下选项可供选择：

- `TRACY_ENABLE`：启用 tracy 代码分析
- `TRACY_ON_DEMAND`：按需分析的开销 _略_ 高，但允许在游戏后期像常规构建一样运行追踪，而常规追踪由于数据量大容易耗尽内存。
- `TRACY_PROFILE_MEMORY`：分析内存分配情况。该功能开销较大，使用原始 malloc 的地方需谨慎处理。
- `RECOIL_DETAILED_TRACY_ZONING`：启用额外的详细 tracy 区域划分（仅用于测试/调试时开启）

例如，启用 Docker 和 Tracy 支持的构建：

```bash
./build.sh -o -t RELEASE -C -DTRACY_ENABLE=1 -p linux-64
```
