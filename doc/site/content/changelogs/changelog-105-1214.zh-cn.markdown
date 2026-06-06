+++
title = "发布版 105-1214"
author = "sprunk"
translator = "Cru"
+++

自版本105-966发布以来，直至2022年10月发布的**版本105-1214**的更新日志。

## 注意事项
* 每个单位对现在仅保留一个 `UnitUnitCollision` 事件，且 `UnitUnitCollision` 的返回值将被忽略！
* `vsync` 设置现在默认为 -1
* 船只在从水中拉出时会遵循 `upright = false`（漂浮时仍保持竖直）。
* 大多数 ARB 着色器的回退方案已被移除

## 功能与修复

### 单位碰撞
* 多线程单位转向与碰撞处理。
* 现在每个单位对仅保留一个 `UnitUnitCollision` 事件，且忽略 `UnitUnitCollision` 的返回值！
* 新增模组规则 `movement.groundUnitCollisionAvoidanceUpdateRate`，用于控制转向性能与质量之间的权衡。降低该值可提升质量，但会牺牲性能。默认值为 3，适用于连续 3 个模拟帧内循环所有单位的场景。
* 新增模组规则 `movement.unitQuadPositionUpdateRate`，与上述类似，用于性能与质量的权衡，但影响碰撞精度（包括对射物）。特别是当此值不为 1 时，光束激光或闪电有时会明显穿过单位。默认值为 3。
* 新增模组规则 `movement.maxCollisionPushMultiplier`，限制单位之间相互推挤的速度（相对于其基础速度的倍数）。默认为infinity/unlimited。

### 粒子
* 添加了带有“翻页”动画的CEG：`animParams = "numX, numY, animLength"`。此功能已启用，适用于BitmapMuzzleFlame、HeatCloudProjectile、SimpleParticleSystem和GroundFlash。
* 部分半透明物体可投射彩色阴影
* 修复了具有非标准烟雾周期的烟雾尾迹
* `/dumpatlas` 目前仅支持 `proj` 参数，用于将粒子弹道图集导出到磁盘

### 坡道上的单位  
* 船只在从水中拖出时现在会遵循 `upright = false`（漂浮时仍保持竖直）。  
* 在单位定义中新增了 `upDirSmoothing`，使单位可以比默认方式更平滑地更新其向上方向向量。  
* 可以在海底坡道上建造浮动的移动单位

### 关闭 Bugger ！
* 添加 `Spring.SetFactoryBuggerOff(unitID, bool? active, number? distanceFromFactoryCenter, number? radius, number? relativeHeading, bool? spherical, bool? forced) → bool active`。Forced 表示也会要求抗推单位移动。所有参数均为可选。
* 添加 `Spring.GetFactoryBuggerOff(unitID) → active, distance, radius, relativeHeading, spherical, forced`，与上述相同单位。
* 添加 `Spring.BuggerOff(x, y = groundHeight, z, radius, teamID, spherical = true, forced = true, excludeUnitID = nil, excludeUnitDefIDArray = nil) → nil`，用于在游戏世界中单独关闭 Bugger（例如当单位站在建筑工地时）。
* `/debugcolvol` 现在会以青色显示 Bugger 关闭的状态，独立的 Bugger 关闭则不显示。

### 按键扫描码
* 添加 `Spring.GetPressedScans() → {[scanCode] = true, [scanName] = true, ...}`，类似于 `Spring.GetPressedKeys`，但使用的是扫描码
* 添加 `Spring.GetKeyFromScanSymbol(scanSymbol) → string keyName`，该函数接收一个扫描码，返回当前键盘布局中对应的用户按键名称
* 添加 `wupget:KeyMapChanged() → nil` 调用，用于在用户切换键盘布局时触发，例如切换输入法语言。

### 选择框
* 添加 `Spring.SetBoxSelectionByEngine(bool) → nil`。如果设置为 false，当松开选择框时，引擎将不会应用选择。
* 添加 `Spring.GetSelectionBox() → minX, maxY, maxX, minY`（拼写错误），返回选择框的屏幕坐标。若未拖动选择框，则返回 nil。

### 渲染内部机制
* 许多引擎内部组件（尤其是字体）采用了现代的渲染方法，预计性能将有所提升。
* 优化了阴影投影矩阵剔除算法，使其更加稳健。结果是，在小地图上、相机倾斜角度等情况下，阴影不再会消失。
* 允许跳过使用纹理内存池，从而节省约512MB（默认值）的静态分配内存。请注意，这可能会因内存碎片化而带来负面影响。请参见 `TextureMemPoolSize` Spring设置。
* 移除了窗口位置代码中的大部分合理性检查（恢复为类似105.0版本的代码）。现在引擎不会阻止用户或控件随心所欲地调整窗口大小。虽然仍保留一些规则，但不应过于严格。
* 字体配置在出错时应更健壮，减少因字体配置错误导致的崩溃次数。
* 修复了多个相机相关问题。
* 修复了 Linux/Wayland 上硬件光标显示问题。
* 修正了立方体贴图纹理映射的错误

### 渲染 API
* 添加 `DrawGroundDeferred` 调用，以便从游戏端的贴图渲染器等地方更新绑定的 g-buffers。
* 现在通过 LuaVBO 可以获取投射物的变换矩阵。

添加了以下 `GL.` 常量：
* `TEXTURE_{1,2,3}D`
* `TEXTURE_CUBE_MAP`
* `TEXTURE_2D_MULTISAMPLE`
* `COLOR_ATTACHMENT{0...15}{,_EXT}`
* `DEPTH_ATTACHMENT{,_EXT}`
* `STENCIL_ATTACHMENT{,_EXT}`

### 其他
* 添加 `Spring.GetSyncedGCInfo(bool forceCollectionBeforeReturning = false) → KiB 级别的内存使用量`，用于从非同步状态查询同步内存占用情况并强制垃圾回收  
* 引入 `RapidTagResolutionOrder` 选项，用于控制在同时使用多个快速镜像时解析快速标签的优先级  
* `/dumprng` 切换命令用于输出同步随机数生成器的状态，功能与 `/dumpstate` 类似  
* 增加 skirmish AI 接口：添加 FeatureDef_isAutoreclaimable 字段  
* 修复了玩家或观众断开连接后出现的 1 秒暂停/冻结问题  
* 修复了当 AllyTeams 大于 2 且一个或多个队伍死亡时 SaveLoad 挂起的问题  
* 修复了 SaveLoad 内存泄漏问题

## PR 下载器
* 添加选项，可覆盖 rapid 和 springfiles 的 URL。
* 添加选项，可直接从 Rapid 下载文件，而无需使用 `streamer.cgi`。详情见 https://github.com/beyond-all-reason/pr-downloader/pull/9
* 通过多线程提升文件 I/O 性能（在 Windows 上性能显著提升）
* 修复了在 Windows 上处理 Unicode 路径的问题
* 修复了在多个 Linux 发行版上查找和加载 SSL 证书的问题（Windows 上也切换至系统证书存储）
