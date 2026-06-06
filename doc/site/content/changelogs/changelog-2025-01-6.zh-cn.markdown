+++
title = "发布版 2025.01"
author = "sprunk"
translator = "Cru"
+++

这是从版本 105-2590 到 **2025.01 版本**（发布于 2025-01-08）之间的更新日志。

# 注意事项
这些条目在迁移时可能需要特别注意：
* 版本控制已改为基于日期的方案，详见下文。  
* 缺失的模型会导致游戏崩溃，以避免不同步（这次是真的）。  
* 为外部检测添加了弹出窗口，请在信息日志中查找错误消息。  
* 在Lua事件中对伤害和死亡进行了大量修改，详见下文。  
* 将天空渲染移至地形渲染之后、`DrawWorldPreUnit`之前进行。  
这将影响由wupgets绘制在预单位图层中的内容。  
* 更改相机模式（如TA、可旋转俯视等）时，现在会尝试保持当前的旋转/位置，而不是恢复到该模式最后一次进入时的旋转/位置。  
* 允许在地图外对单位下达攻击指令。但地面目标仍不允许。  
* 当守卫或守护者处于地图外时，允许下达守卫指令。  
* 拥有 `groundBounce = true` 的武器现在即使 `numBounces` 未定义也会发生反弹。这是因为原先 `numBounces = -1`（默认值）会导致无限次反弹，而不再是0。  
* `GroundDecals` 的Spring设置现在为布尔值。此前该参数作为贴花持续时间乘数的语义已被移除。若需要延长持续时间，请使用新的 `scarTTL` 武器定义标签。* 明确指定的武器定义 `scarTTL` 值现在为实际的 TTL。  
此前该值会乘以地面贴图等级的Spring设置，默认值为 3，因此如果你曾修改过伤疤，现在它们的 TTL 可能会显著不同。  
* 活动移动单位的占地面积现在等于其 movedef 的大小。这会影响诸如响应“bugger-off”、阻挡建筑或内置选择方块大小等行为。请注意，单位定义本身不受影响，且当单位在运行时 movedef 发生变化时，各单元将自动调整。  
* 现在强制支持非 2 的幂次纹理、浮点纹理以及帧缓冲区对象。显然这些功能早在 15 年前就已常见，因此即使使用较老的系统也应安全可用。  
* 已从武器定义中移除 `tdfID`，包括 WeaponDefs 表。所有剩余接口均改为使用 weaponDefID。  
* 归档根目录下以点开头的文件现在在计算校验和时被忽略（类似于 `springignore.txt` 第一行包含 `\..*`，此设置无法覆盖）。  
* 归档扫描器版本已更新至 17，将无法再复用旧的归档缓存。  
请预计归档将重新扫描。* `TooltipGeometry` 的Spring设置默认值将 Y 坐标从 0 改为 0.125。  
* 默认提示框现在会为每种资源单独在一行中显示收入和收获存储信息。

### 弃用通知
* 将 `Spring.UnitIconSetDraw` 重命名为 `Spring.SetUnitIconDraw`。旧拼写仍可暂时使用，但已被弃用。
* 现在提供了 `Spring.GetUnitCommandCount(unitID)` 来获取单位的队列大小。
`Spring.GetUnitCommands(unitID, 0)` 以及等效的 `Spring.GetCommandQueue` 仍可暂时使用，但已被弃用。

# 功能

### 版本控制
* 引擎现在采用基于日期的版本控制方案，这意味着我们从 105 跳转到 2025。  
* 在更多地方将“Spring”改为“Recoil”。  
* `Engine.versionFull`、`Engine.versionPatchSet` 和 `Engine.buildFlags` 现在已同步可用。  
* 新增了 `Engine.versionMajor`、`Engine.versionMinor` 和 `Engine.commitsNumber`。

### rmlUI
RmlUI 已上线！这是一个图形用户界面（GUI）框架，可让你使用网页技术（HTML、CSS 等）创建小部件。  
Lua API 可参考 [文档](https://mikke89.github.io/RmlUiDoc/pages/lua_manual/api_reference.html)；在 LuaUI 中可通过 `RmlUi` 全局变量访问。  

与上游版本的差异：  
* `context:LoadDocument(filepath, widget)` 接收第二个参数为小部件本身，因此内联事件处理器可以像这样调用其函数：`onclick="widget:OnReloadClick()"`。  
* 也提供了 `document` 对象。  
* 我们的数据模型仅在第一层深度时才具有响应性。  
所有响应性的键在创建时都必须存在。  
如果同一个数据模型被重复“打开”，其行为未作规定。  
* 提供了一个名为 `texture` 的自定义 HTML 标签，用于访问引擎纹理，例如：`src="$heightmap"`

请在游戏（尤其是BAR）中寻找即将发布的示例。  
API 目前为 v1 版本，可能会有变更，请使用新的 `Engine.FeatureSupport.rmlUiApiVersion` 变量进行兼容性检查（见下文）。

### 功能支持表

有一个新的 `Engine.FeatureSupport` 表，用于包含各种功能支持检查。  
可用于引擎版本兼容性，类似于现有的 `Script.IsEngineMinVersion`，  
但具有自文档特性，且不假设线性可用性或提交编号的连续性。

目前包含三个变量：
* `rmlUiApiVersion` = `1`
* `hasExitOnlyYardmaps` = `true`
* `NegativeGetUnitCurrentCommand` = `true`

未来随着新功能的添加，还将有更多内容加入。

### 死亡事件
* `wupget:UnitDestroyed` 会在单位被回收时，将建造者作为杀手传递。请注意
reclaim 仍然不会生成 `UnitDamaged` 事件。  
* `wupget:UnitDestroyed` 现在会接收一个第七个参数 weaponDefID，用于表示死亡原因。  
无论攻击者是否处于视野范围内，小部件都会接收到此信息（无新魔法，`UnitDamaged` 已经实现了这一功能）。  
* 上述的 weaponDefID 始终不会为 `nil`，所有死亡原因均会被记录，包括例如：  
“正在工厂建造中，被取消”或“因 `isFeature` 定义标签而自动死亡”。  
为此新增了大量 `Game.envDamageTypes` 常量。参见页面底部的表格。  
* 当非 `releaseHeld` 运输工具中的单位因非自毁原因死亡时，原先应用的 1000000 的伤害现在将归入 `UnitDamaged` 中的新 `TransportKilled` 伤害类型，而非之前的 `Killed`。  
* 建造衰变现在会产生一个事件：`wupget:UnitConstructionDecayed(unitID, unitDefID, unitTeam, timeSinceLastBuild, iterationPeriod, part)`。  
其中时间与迭代周期以秒为单位，part 是一个分数。

### 轨道摄影机
添加了一个可通过Lua激活的轨道摄像机，可沿预设路径移动。  
向`Spring`表中添加了以下函数，具体签名和功能请参见主Lua API文档：
* `RunDollyCamera`
* `PauseDollyCamera`
* `ResumeDollyCamera`
* `SetDollyCameraMode`
* `SetDollyCameraPosition`
* `SetDollyCameraCurve`
* `SetDollyCameraLookCurve`
* `SetDollyCameraLookPosition`
* `SetDollyCameraLookUnit`
* `SetDollyCameraRelativeMode`
* `SolveNURBSCurve`

### 其他摄像工作
* 添加 `CamTransitionMode` 整数弹簧设置，用于控制离散相机移动的插值方式。  
0 - 指数衰减，1 - 弹簧阻尼，2 - 时间延迟弹簧阻尼，3 - 线性。默认为 0 - 指数衰减。  
* 添加 `CamSpringHalflife` 浮点数弹簧设置，表示时间延迟弹簧阻尼插值模式应大约在目标位置一半时所经过的时间（单位：毫秒）。  
* 添加 `OverheadMinZoomDistance` 和 `CamSpringMinZoomDistance` 浮点数弹簧设置，用于设定各自相机的最小缩放距离。请注意，这是视锥体朝向观察方向的距离，而非高度。  
* 更改相机模式（如 TA、可旋转俯视等）时，现在会尝试保持当前的旋转/位置，而不是恢复到该模式下上次的旋转/位置。

### 地图外指令
* 允许将中心区域命令施加到地图外的单位上。  
* 允许对地图外的单位下达攻击命令，但地面目标仍不允许。  
* 当守卫或守护者在地图外时，允许下达守卫命令。

### Lua 微优化
* 添加 `math.normalize(x1, x2, ...) → numbers xn1, xn2, ...`。对向量进行归一化处理。可支持任意维度（可分别传入并接收每个维度的值）。  
若传入零向量，则返回零向量。  
* 添加 `table.new(arraySlots, hashSlots) → {}`。返回一个空表，但分配了更多空间。  
当需要创建一个大表并填充已知数量的元素时，可使用此方法作为微优化手段，例如 `#UnitDefs`。  
* `Script.LuaXYZ.Foo()` 不再在 LuaXYZ 环境中未找到该函数时发出警告。实际上这意味着你可以避免每次调用 `Script.LuaXYZ("Foo")`，但仍可考虑手动避免不必要的计算或发出警告。  
* 添加 LUS 的可变参数版本：`Turn`、`Move`、`Spin`、`StopSpin`、`Explode` 和 `SetPieceVisibility`，每个都以“Multi”前缀命名（如 `MultiTurn` 等）。  
与普通函数相比，这些函数可以接受多个完整参数组，从而避免额外的函数调用。

### Lua 命令
* 添加 `Spring.GetUnitCommandCount(unitID) → number commandCount`，可替代 `Spring.GetUnitCommands(unitID, 0)`。
* `Spring.GetUnitCurrentCommand(unitID, -n)` 现在会获取从最后一个命令开始的第 N 个命令。
* 新增 `Engine.FeatureSupport.NegativeGetUnitCurrentCommand` 布尔值，以支持上述功能的向后兼容性。

### 字体
* 新增了 `Spring.AddFallbackFont(string path) → bool ok`，允许你指定备用字体，以防当前使用的字体缺少某些字形。  
你可以使用多个字体，它们将按添加顺序依次搜索。  
* 新增了 `Spring.ClearFallbackFonts() → nil`，用于清除备用字体。  
* 新增了 `wupget:FontsChanged() → nil`，非同步调用，当字体集发生变化时通知 Lua，以便你重新生成包含文本等的位图。  
* 优化了字体加载过程。  
* 修复了从字体中加载未知字形时出现的崩溃问题。  
* 修复了有时会在用户磁盘上创建 `C:\a\_temp\msys\msys64\var\cache\fontconfig` 的问题。

### 空间查询
* 添加 `Spring.TraceRayGroundInDirection(startX, startY, startZ, dirX, dirY, dirZ, maxLength = inf, bool checkWater = true) → 距离, x, y, z`。  
在指定方向上检测地面（或水面），并返回第一个碰撞点。方向无需归一化。  
* 添加 `Spring.TraceRayGroundBetweenPositions(xA, yA, zA, xB, yB, zB, bool checkWater = true) → 距离, x, y, z`。  
检测两点 A 和 B 之间的地面（或水面）。  
* 添加 `SelectThroughGround` 浮点数弹簧设置。控制单位单击穿透地面的深度，默认值为 200，与 elmos 中的行为相同。  
* `Spring.GetUnitsInRectangle` 及类似函数现在能正确获取摇晃的雷达点。  
* 修复了右键点击和区域命令有时无法包含雷达点的问题，特别是当雷达点与真实位置偏离过大时。

### 渲染
* 引擎现在会在 `wupget:DrawWorldPreUnit` 层之前绘制天空。  
* `wupget:DrawWorldPreParticles` 现在包含四个布尔参数，根据所绘制的阶段不同而变化：水面之上、水面之下、反射、折射。这些参数并非互斥。  
* 最小化时，绘制频率现在为每30秒一次（原本已设定但未实际实现）。  
* 添加了 `MinSampleShadingRate` 振动设置，浮点数范围在 0 到 1 之间，默认值为 0。数值为 1 表示帧缓冲区中的每个样本都应独立进行着色；数值为 0 则相当于允许渲染忽略样本率着色。介于 0 和 1 之间的任意值，可让 OpenGL 只对每个覆盖片段内的部分样本进行着色。  
* 在 Lua 纹理选项中新增字符 'm'，用于禁用 DDS 和无压缩纹理的三线性 mipmap 过滤。  
* 粒子（包括地面贴花）现在拥有 4 级 mipmap。  
* 修复了非对称模型的绘制位置问题，不再在不适用时消失。

### 定义
* 添加 `windup` 武器定义标签，用于设置第一发炮弹出现前的延迟（以秒为单位）。  
其机制与 burst 相同（遵循最近的 out-of-arc 标签，且延迟可通过 burst Lua API 设置和读取）。  
* 活动移动单位的足迹现在等同于其 movedef 的大小。这会影响诸如响应“bugger-off”、阻挡建筑或内置选择方块大小等功能。请注意，单位定义本身不受影响，且当单位在运行时 movedef 发生更改时，单个单位会相应调整。  
* 武器定义现在新增了 `animParamsN`（N = 1-4）标签，用于指定纹理 1-4 的翻页动画，格式与 CEG 相同（三个数值：精灵数量 X、Y 和持续时间）。  
* 已移除武器中的 `tdfID` 标签。

### 存档扫描
* 修复了因时区变更（例如夏令时）导致 Windows 系统对归档文件进行完全重新扫描的问题。  
* 加载界面在扫描归档文件时现在会显示更多信息。  
* 对于校验和功能，归档根目录中以点号开头的文件现在将被忽略（类似于 `springignore.txt` 第一行使用 `\..*` 的情况，此设置无法覆盖）。  
* 归档扫描器版本已更新至 17，将无法复用旧版归档缓存。  
预计需要重新扫描归档文件。  
* 重新扫描仅在内部归档扫描器版本更改时发生，不再受引擎版本变化影响。  
* 优化了在硬盘上扫描文件时的性能。  
* 修复了归档扫描器有时因同时打开的文件数超过操作系统允许数量而失败的问题。

### 风
* 添加 `misc.windChangeReportPeriod` 模组规则，单位为秒。风力发电机在经过指定秒数后会接收到“风向更新”事件。默认值为15秒（旧行为）。
* 风力发电机现在在完成时也会接收该事件。
* 风力现在以随机方向启动，不再固定朝东。

### Tracy 性能分析
* 添加 `tracy.LuaTracyPlotConfig(string plotName, string type = "Number"|"Memory"|"Percentage", bool stepwise = true, bool fill = false, integer colorBGR = 0xFFFFFF) → nil`，可创建 Tracy 图表并配置其外观。  
* 添加 `tracy.LuaTracyPlot(string plotName, number value) → nil`，可用于填充图表的数值。  
* 改进了 Tracy 的覆盖范围，并使区域着色更加一致。

### 日志
* 添加一个“已弃用（deprecated）”日志级别，数值优先级为37。您可以在 `wupget:AddConsoleLine` 中查看优先级。
* 现在多个引擎的弃用警告都使用了该日志级别。

### 杂项
* 添加 `construction.insertBuiltUnitMoveCommand` 布尔值模组规则，默认为 true。若设为 false，单位在离开工厂时将不会收到移动指令（请务必使用 bugger off）。
* 添加 `Spring.ForceUnitCollisionUpdate(unitID) → nil`。强制单位正确进行碰撞检测。通常情况下，碰撞会根据 `unitQuadPositionUpdateRate` 模组规则更新，这可能导致某些武器在移动时无法命中。当重要武器的目标（例如 `script.FireWeapon` 中的 hitscan）的模组规则值大于 1 时，请调用此函数以确保可靠的命中检测。
* 添加 `Spring.SetProjectileTimeToLive(projID, number framesTTL) → nil`，设置项目在模拟帧中的剩余存活时间。
* 将 `Spring.UnitIconSetDraw` 重命名为 `Spring.SetUnitIconDraw`。旧拼写仍可暂时使用，但已弃用。
* 内置的终局图现在提供线性缩放的开关，而非线性模式。
* `gl.SaveImage` 现在可以保存为 `.hdr` 格式（似乎如此）。
* `pairs()` 现在会检查表中的 `__pairs` 超级方法，与 Lua 5.2 相同。
* 初始化时的“扫描存档”界面会显示更多信息。* `TooltipGeometry` 的Spring设置默认值将 Y 坐标从 0 改为 0.125。  
* 默认提示框现在会为每种资源在单独的行中显示收入和收获存储。  
* 添加 `/debugquadfield` 命令，用于调试 GUI 中光追与地面四边形的交互。

## 修复
* 修复可能的路径同步问题，尤其是在为 OpenBSD 编译的构建中。
* 修复非对称模型的绘制位置，避免在不适用时消失。
* 修复 `Spring.GetUnitWeaponHaveFreeLineOfFire` 不再正确使用大炮类武器的源 XYZ 坐标。
* 修复流式传输极小声音文件的问题。
* 修复 `Spring.SetCameraTarget` 会回退到旧位置的问题。
* 修复 `/iconsHideWithUI` 命令未能正确保存相关弹簧设置的问题。
* 修复自大约 105-800 版本起 API 中缺少 `Spring.GetFeatureSelectionVolumeData` 的问题。
* 修复 `/groundDecals` 在引擎启动时禁用贴图的情况下无法启用贴图的问题。
* 修复右键点击和区域命令有时无法包含雷达点（当其与真实位置偏离过大时）的问题。
* 修复从字体加载未知字形时出现崩溃的问题。
* 修复运行时金属贴图调整未被正确保存/加载（将使用地图默认值）的问题。
* 修复金属提取器未被正确保存/加载（允许重复）的问题。
* 修复有时用户磁盘上会创建 `C:\a\_temp\msys\msys64\var\cache\fontconfig` 的问题。
* 模块规则 `quadFieldQuadSizeInElmos` 现在仅接受二进制幂，不再在地图边缘断裂。

### 死亡伤害类型列表

| Game.envDamageTypes.??? | 描述	                                                                                                                                                          |
|-------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AircraftCrashed         | 飞机坠落地面			                                                                                                                                          |
| Kamikaze                | 因其神风特攻技能而爆炸的单位			                                                                                                                          |
| SelfD                   | 在执行自定义命令并倒计时后，单位发生爆炸																														  |
| ConstructionDecay       | 废弃的纳米框架正在消失（该过程本身是HP去除）					                                                                                                  |
| Reclaimed               | 通过回收被杀死（该过程本身是HP移除）				                                                                                                              |
| TurnedIntoFeature       | 单位完成时死亡，未爆炸由于 `isFeature` def 标签						                                                                                              |
| TransportKilled         | 该单位处于没有 `releaseHeld` 状态的运输中。如果运输未自毁，该单位在死亡前还会受到1000000点此类伤害。															      |
| FactoryKilled           | 该单位正在一座已倒闭的工厂中建造。目前尚无直接方式得知工厂具体是如何倒闭的，你只能等待工厂的死亡事件发生。														  |
| FactoryCancel           | 该单位原本在工厂内建造，但建造已被取消。					                                                                                                      |
| UnitScript              | COB单位脚本已下令该单位死亡。请注意，LUS可使用正常的击杀/伤害界面。											                                                      |
| SetNegativeHealth       | 单位因非伤害原因（例如 Lua 设置了）的血量低于 0。						                                                                                          |
| OutOfBounds             | 一个单位被抛出了地图范围之外。			                                                                                                                          |
| KilledByCheat           | 使用了 `/remove` 或 `/destroy` 命令。		                                                                                                                      |
| KilledByLua             | 使用 `Spring.DestroyUnit` 时的默认死亡原因。		                                                                                                              |

`KilledByLua` 保证是“最后一个”值，因此你可以为 Lua 机制自定义损坏类型，并确保不会发生重叠，例如通过
```
Game.envDamageTypes.CullingStrike      = Game.KilledByLua - 1
Game.envDamageTypes.SummonTimerExpired = Game.KilledByLua - 2
```
