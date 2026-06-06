+++
title = "发布版 105-1544"
author = "sprunk"
translator = "Cru"
+++

自版本105-1354发布以来，直至2023年2月发布的**版本105-1544**的更新日志。

## 注意事项
* 将 fork 重命名为 Recoil。所有需要重命名且会带来技术影响的地方（如 `spring.exe` 文件名、`Spring.Foo` Lua API 等）均保持原样。
* `Spring.GetConfig{Int,Float,String}` 的第二个参数现在可以为 `nil`，这意味着如果请求的设置未配置，则返回 `nil`。以前分别返回 `0`、`0` 或 `""`。请注意，引擎使用的设置值永远不会为 `nil`（因此此功能仅适用于自定义设置）。要恢复之前的默认行为：
```lua
local originalGetConfigInt = Spring.GetConfigInt
Spring.GetConfigInt = function (key, def)
   return originalGetConfigInt(key, def) or 0
end
-- 浮点数和字符串
```

## 功能与修复

### 通用多线程改进
* 当超线程不可用时，主线程现在可以跨核心移动。这在服务器运行多个无头或OpenGL实例时非常有帮助。
* `LoadingMT` Spring设置不再接受 -1 用于自动检测（这意味着 Intel/Mesa 显卡驱动为 0，其他情况为 1）  
* 添加 `Spring.Yield() → bool hintShouldKeepCallingYield`，用于将游戏加载线程返回的 OpenGL 上下文传递回主线程，以便处理事件并使用 LuaIntro 进行绘制。无论 LoadingMT 的状态如何，调用此函数都是安全的。实际上，这意味着只要返回的提示值为 true，就应该在未同步的 wupget 初始化器中定期调用。常见的实现方式是在 wupget 加载后调用，例如：
```lua
local wantYield = true
for wupget in wupgetsToLoad do
  loadWupget(wupget)
  wantYield = wantYield and Spring.Yield()
end
```

### 多线程模块规则
来自未来的说明：这些选项在1775版本发布后已被移除。  
* 新增 `system.pfForceSingleThreaded`，用于强制路径请求以单线程方式发送。  
* 新增 `system.pfForceUpdateSingleThreaded`，用于强制路径顶点更新以单线程方式处理。  
* 新增 `movement.forceCollisionsSingleThreaded`，用于强制碰撞系统仅使用单个线程。  
* 新增 `movement.forceCollisionAvoidanceSingleThreaded`，用于强制碰撞规避系统仅使用单个线程。

### 调试改进
* 新增布尔型配置项 `DumpGameStateOnDesync`。当服务器端设置为 true 时，会要求所有客户端生成当前游戏状态的本地副本（相当于调用 `/dumpstate`）。若在客户端设置，则无任何效果。  
* 添加 `/debug reset` 命令，用于重置收集的调试统计数据。  
* 新增非调试版本的 `/desync` 命令。  
* 修复了在第300帧模拟时间（10秒）之前无法检测到断联的问题。

### 独立式模型件获取器
与类似现有函数不同，这些函数不需要存在单位/特征实例。
* `Spring.GetModelPieceMap(string modelName) → { body = 0, turret = 1, ... }`
* `Spring.GetModelPieceList(string modelName) → { "body", "turret", ... }}`

### 新的 Selectkeys 过滤器
* `Buildoptions` - 对于有非零建造选项的单位（即建造者和工厂）
* `Resurrect` - 适用于可复活的单位
* `Cloaked` - 对于当前处于隐蔽状态的单位
* `Cloak` - 对于可以隐身的单位（无论当前是否处于隐身状态）
* `Stealth` - 对于默认为隐形类型的单位（无论当前的隐形状态如何）
* （注意，目前没有像隐身效果那样的“当前隐身”过滤器）

### Drawing 相关更改
* 添加 `wupget:DrawPreDecals()` 调用
* 添加布尔型Spring设置 `AlwaysSendDrawGroundEvents`（默认为 false），用于始终发送 `DrawGround{Pre,Post}{Forward,Deferred}` 事件；否则在使用非 Lua 渲染管线时，这些事件将被跳过。
* 添加 `/drawSky` 命令，用于开启或关闭天空绘制（目前主要用于性能测试）  
* 缺失或无效细节纹理的地图不再会显示为红色。改进了缺失地图纹理的报告机制  
* 可通过Spring设置 `ShadowColorMode = 0` 或使用 `/shadows` 开关将阴影颜色缓冲区设为灰度  
* 修复了非 NVIDIA 驱动下的凹凸水渲染问题

### 其他功能
* 添加 `Game.metalMapSquareSize`，用于与 `Spring.GetMetalAmount` 等 API 配合使用  
* 添加动画 CEG 图片的镜像循环（动画向后播放直至回到起点，然后再次反弹），通过将动画速度参数设为负值来启用（例如 `animParams = "4,4,-30"`）。  
适用于 `BitmapMuzzleFlame`、`HeatCloudProjectile`、`SimpleParticleSystem` 和 `GroundFlash` 这些 CEG 类。  
* 添加 `Spring.GiveOrderArrayToUnit`，接受单个单位ID和一个命令数组，其功能与现有的 `GiveOrderXYZ` 系列函数相同

### 各类修复
* 修复了当有活动鼠标所有者（activeReceiver）时，`Spring.GetSelectionBox` 不返回 nil 的问题  
* 修复了多个与移动、路径规划质量及性能相关的问题  
* 修复了加载使用 Circuit/Barbarian AI 的游戏时出现的崩溃问题  
* 扫描存档时（在早期加载阶段），窗口不应再被操作系统视为冻结或无响应  
* `PreloadModels = 0`（非默认值）现在应能正确工作，尽管这会带来一些权衡。

## PR 下载器
* 实现通过单次工具调用同时获取多个模组和地图  
* 将可能过快的 `If-Modified-Since` 缓存替换为 ETag  
* 通过仅在完整成功下载后保存 SDP 文件，增强快速包的下载安全性  
* 实现为从 springfiles 下载的资源生成 `md5sum` 文件，以便未来对完整游戏文件进行验证  
* 从仓库中移除 lsl，并将 RapidTools 分离至独立仓库  
* 修复依赖的 Rapid 归档解析问题  
* 修复 MSVC 下的编译问题，使代码库的部分组件更具跨平台兼容性
