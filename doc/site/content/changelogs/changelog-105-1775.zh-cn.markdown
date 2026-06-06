+++
title = "发布版 105-1775"
author = "sprunk"
translator = "Cru"
+++

自版本105-1544发布以来，直至2023年6月发布的**版本105-1775**的更新日志。

## 注意事项
* `Spring.GetSelectedUnits{Sorted,Counts}` 不再返回表格内的 `n`，而是现在返回函数的第二个返回值。  
* LuaUI 可在适当情况下通过 `UnitDamaged` 等事件接收攻击者信息。如果不想使用此功能，请在小部件处理程序中将其设置为 `nil`（或直接不传递给小部件）。  
* 移除了已弃用的 `Game.allowTeamColors` 项，该选项原本始终为 `true`。  
* PNG 现已成为默认的 `/screenshot` 格式。  
* 新增了后坐力图标和加载画面。  
* 由于引入了皮肤/骨骼系统（见下文），每个模型的最大部件数量被限制为 254 个。  
* 与皮肤和骨骼相关的单位着色器变更。这需要更新调用 `vbo:ModelsVBO()` 的代码，具体替换如下：
 ```
 layout (location = 5) in uint pieceIndex;
 ```
 与
 ```
layout (location = 5) in uvec2 bonesInfo; // boneIDs, boneWeights
#define pieceIndex (bonesInfo.x & 0x000000FFu)
```

## 功能与修复

### 性能优化
* 现在可通过 springsettings 配置 CPU 使用率用于绘图及每秒最低帧数：`MinSimDrawBalance` 控制分配给绘图的最小 CPU 比例；比例越高，帧率越高但追上速度越慢（默认为 0.15）；`MinDrawFPS` 控制最低期望帧率，低于该值时将不会达到（默认为 2）。  
* 修复了此绘图/模拟比例与“为延迟者减缓游戏”功能之间的交互问题。

### 建模
* 增加了对皮肤和骨骼的支持。不支持 `.s3o` 或 `.3do` 格式，仅支持 `.dae`（引擎支持的其他格式较为少见且未经测试，但能够携带骨骼信息的格式应同样可用）。无需额外工作，使用行业标准工具（例如 Blender）创建的骨骼应可直接使用。
* 需要更改着色器，详见上方注意事项部分。
* 允许在未附加实例 VBO 的情况下进行实例化渲染。可通过 `gl_InstanceID` 作为索引访问 SSBO，或用于算法生成实例属性。
* 将凹凸模型池从 2560 增加到 3840，并在耗尽时记录日志。
* 增大 3do 大图集大小（4k^2）。
* 将默认的 `MaxTextureAtlasSize{X,Y}` 弹簧设置从 2048 更改为 4096。
* 修复 3do 溶解碎片渲染问题（无纹理）。
* 修复 AMD / Windows 系统中无效“正在建设中”模型渲染的问题。
* 修复无效资源校验和计算问题。

### 地形渲染
* 添加 `wupget:DrawShadowPassTransparent() → nil` 和 `wupget:DrawWaterPost() → nil` 调用  
* 信息纹理着色器（F1、F2 等）对驱动程序的兼容性更高  
* 改进地形阴影质量及可见性检查  
* 添加 `/debugVisibility` 用于调试可见的四边形区域（仅对引擎开发者有用，否则基本无用）  
* 添加 `/debugShadowFrustum` 在迷你地图上绘制阴影投影框（同理；需注意，它并不完全对应实际摄像机投影框，因此在非阴影场景中可能因小bug而实用性较低）  
* 修复了当地形不同时，第二代及后续游戏中的概览摄像机未居中问题  
* 修复了 Intel 平台上的凹凸水面和地形渲染问题

### 选择单位
* 为选择添加 `InGroup_N` 过滤器，用于选择指定的控制组。例如：`Visible+_InGroup_1+_ClearSelection_SelectAll`
* 添加 `group select` 和 `group focus` 这两个组子操作。请注意，之前的 `bind X group N` 等价于 `bind X group select N` + `bind X,X group focus N`。旧的 `group N` 仍然有效。

### 单位定义：武器
* 为武器添加 `weaponAimAdjustPriority` 数值标签（注意，不是武器定义！这是单位定义中的表格，例如目标类别等）。该标签用于计算武器当前瞄准方向上获取目标的重要性乘数，默认值为1。  
* 并为武器添加 `fastAutoRetargeting` 布尔标签。当现有目标被消灭时，使单位自动获取新目标（权衡是性能成本，且若已存在单位则会增加增益效果）。默认值为 false（每0.5秒检查一次目标获取情况）。

### 单位航向
* 修改了单位旋转的实现方式，这应该能让蜘蛛类在穿越近乎垂直的峰顶时更好地保持朝向  
* 新增 `Spring.Set{Unit,Feature}HeadingAndUpDir(thingID, heading, x, y, z) → nil`，请注意，之前存在的 `Spring.SetUnitDirection` 只能接收一个向量，因此无法明确设置对象的旋转方向

### 路径
* TKPFS 合并到默认的 HAPFS 中。  
* 新增一个模组规则 `system.pathFinderUpdateRateScale`（默认值为 1.0），用于控制路径查找器的更新速率。请注意，该设置将在 1775 版本之后被移除。  
* 路径覆盖层（F2）的更新速率提高了 4 倍。  
* 引擎在移动单位时现在会尽量保持相对位置（以避免等效单位相互重叠）。  
* 修复了多个路径质量相关问题，并提升了性能。

### 其他 API 变更
* 允许 `Spring.TransferUnit` 中的浅层递归（深度为16级，与其他调用相同）  
* 添加 `Game.footprintScale`，用于设置与默认值相比的足迹缩放倍数  
* 添加 `Game.buildSquareSize`，表示建筑对齐网格的大小  
* 添加 `system.LuaAllocLimit` 模块规则，用于控制全局 Lua 分配限制，默认为 1536（单位兆字节）  
* `Spring.GetPlayerInfo` 现在返回第12个值，即布尔值 `isDesynced`。请注意，如果禁用自定义键表，则第11个值将为 `nil`（即新的第12个值不会“回推”以填补“缺失”的位置）  
* 添加 `Spring.GetFeaturesInScreenRectangle(x1, y1, x2, z2) → {featureID, featureID, ...}`，类似于现有的单位功能获取方法  
* `Spring.SetProjectileCEG` 现在除了接受 CEG 名称外，还可接受数值形式的 cegID  
* 添加 `Game.demoPlayName`，用于获取当前正在播放的存档文件名  
* 在 `Spring.MarkerErasePosition` 中添加第7个布尔参数，当 `localOnly` 为真且当前玩家处于旁观状态时，命令将清除所有标记。这使得旁观者可以删除其他玩家的标记，而无法通过 `/clearmapmarks` 外部操作完成此功能  
* `Spring.SetActiveCommand(nil)` 现在会取消命令（之前的 `-1` 方法仍然有效）* 添加 `StoreDefaultSettings` 这个 springsetting，默认值为 false。如果启用，即使设置与默认值相同，仍会保存这些设置。这意味着 `Spring.GetConfigInt` 等函数无需对默认值进行 nil 检查，同时默认值也不会在发生更改时自动更新，因为它们已经具有具体的值。
* 重新实现 Lua 内存池，由 `UseLuaMemPools` springsettings 控制。

### 杂项
* 增加了对Tracy的调试支持。  
* 默认模拟器最低速度从0.3修改为0.1。  
* 新增`--list-[un]synced-commands`参数，用于直接运行可执行文件时输出命令列表。  
* 修复了`/iconsAsUI 1`导致的已死亡幽灵建筑信息泄露问题。  
* 修复了碎片弹（爆炸单位飞出的残骸）以及武器弹道在`collideFireBase`设置为true且其他`collisionX`设置为false时无法与单位相撞的问题。

## PR 下载器
* 改进检查是否需要更新的性能：在所有操作系统上现在耗时约0.25秒  
* 引入自定义 X-Prd-Retry-Num，用于告知服务器获取文件时是第几次重试  
* 修复了当所有文件均已存在时 sdp 下载的行为  
* 修复了在出现重定向以及总下载大小超过2GiB时进度条的显示问题  
* 修复了 `--disable-all-logging` 参数的实际功能  
* 修复了 Windows 系统下 Unicode 环境变量的处理
