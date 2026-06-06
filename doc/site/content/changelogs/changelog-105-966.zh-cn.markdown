+++
title = "发布版 105-966"
author = "sprunk"
translator = "Cru"
+++

自版本105-941发布以来，直至2022年6月发布的**次要版本105-966**的更新日志。

## 注意事项
* 已移除 LOD 单位渲染（远距离 2D 显示），包括 `/distdraw` 命令和 `UnitLodDist` springconfig 项
* 空中飞行器不再强制保持直立，无论其行驶的地形坡度如何，现在会遵循单位定义中的 `upright` 设置
* 键盘快捷键中的 `Up+` 修改符现已弃用，尝试绑定时将显示警告，但行为无变化

## 功能与修复

### 快捷键
* `wupget:KeyPress` 和 `wupget:KeyRelease` 调用会接收一个额外的 scanCode 参数。  
* 引入了 `sc_foo` 键集，用于扫描码，并与按键码绑定结合使用，同时保持绑定操作的顺序。`Any+` 的顺序被保留（`Any+` 绑定位于底部）。  
* `/keysave` 和 `/keyprint` 现在能保留绑定顺序，提供更清晰的输出。  
* 扩展 `Spring.GetKeyBindings` 以接收额外的键集字符串，用于获取扫描码动作。  
* 以类似 `Spring.GetKeySymbol` 的方式添加 `Spring.GetScanSymbol` 调用。  
* 修改 basecontent wupget 处理器，使其支持扫描码。建议希望通过扫描码支持 Lua 动作的游戏采用这些更改，否则行为应与之前相同。  
* 修复当动作行包含不同注释时，键集可能绑定到重复动作的问题。  
* 修复键码名称检索始终返回非十六进制值的问题，此问题影响了 KeyPress 和 KeyRelease 调用中的 label 参数。  
* 修复 `Spring.GetKeyBindings` 会返回错误的操作，例如，如果 m,n 和 b,n 都绑定了操作，那么 GetKeyBindings('n') 将返回这两个操作，而不管之前按下了哪个键。

### 渲染
* 添加 `gl.GetEngineAtlasTextures("$explosions" | "$groundfx") → { texName = {x1, y1, x2, y2}, texName = {...}, ... }`.
* 当具有相同尺寸的纹理按相同的字母顺序排列时，atlasses 现在具有确定的顺序。这使得 Lua 可以创建 atlasses 的配对，例如多个贴图的漫反射和法线贴图配对。

### 飞行器平滑网格
* 现在平滑高度网格将响应高度图的变化，并在受影响区域重新计算自身。这意味着即使经过长时间的战斗导致地形发生显著变化，使用平滑网格的飞行器仍能继续正常导航。
* 现在可通过布尔型模组规则 `system.enableSmoothMesh` 来禁用平滑网格。

### 原始高度图
Lua 现在可以修改被视为“原始”高度图的内容，主要用于随机地图生成器。这些新接口与当前高度图的现有接口功能相同。
* `Spring.AdjustOriginalHeightMap(x1, z1[, x2, z2], height) → nil`：增加高度。
* `Spring.LevelOriginalHeightMap(x1, z1[, x2, z2], height) → nil`：设置高度。
* `Spring.RevertOriginalHeightMap(x1, z1[, x2, z2], factor) → nil`：将高度恢复为地图文件中定义的“原始”高度图；`factor` 是一个 0 到 1 的值，用于在当前高度和地图文件高度之间进行插值。
* `Spring.SetOriginalHeightMapFunc(func, arg1, arg2, ...) → number totalChange`：调用一个函数，该函数可调用以下两个函数之一。
* `Spring.AddOriginalHeightMap(x, z, height) → newHeight`：仅可通过 `Spring.SetOriginalHeightMapFunc` 调用，用于在指定坐标处增加高度。
* `Spring.SetOriginalHeightMap(x, z, height[, factor=1]) → heightChange`：仅可通过 `Spring.SetOriginalHeightMapFunc` 调用，使用给定因子将指定坐标处的高度向目标值进行插值。

使用示例：
```lua
Spring.SetOriginalHeightMapFunc(function(amplitude, period)
	for z = 0, Game.mapSizeZ, Game.squareSize do
	for x = 0, Game.mapSizeX, Game.squareSize do
		Spring.SetOriginalHeightMap(x, z, 100 + amplitude * math.cos((x + z) / period))
	end end
end, 123, 456) -- args to pass (in this case amplitude and period)
```

### 杂项
* 添加了 `Spring.AddUnitExperience(unitID, delta_xp)`。可以减去经验，但若为负数，结果将被截断为 0。  
* 修复了 `/crash` 命令，使其能正确崩溃。
