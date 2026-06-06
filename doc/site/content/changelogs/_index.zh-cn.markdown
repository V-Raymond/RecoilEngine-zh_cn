+++
title = "更新日志"
[cascade]
  [cascade.params]
    type = "docs"
+++

这是自2025.06版本以来的前沿更新日志，适用于**2026.06预发布版**。

## 注意事项

- 现在支持 UTF-8 文件路径。  
- 部分文件访问现在区分大小写。  
- 使用的 rmlUI 版本从 6.0 更新为 6.2。  
- ARM64 架构构建现已提供基本支持。  
- `/aicontrol` 默认被阻止，需调用 `/aiCtrl PlayerName` 或 `/aiCtrlByNum 123` 才能启用。  
- `script:AimWeapon` 现在接收相对于单位的朝向和俯仰角，而非世界空间坐标。这意味着位于斜坡上的单位将获得不同的数值。  
- 朝向转为弧度后，返回值范围变为 \[-pi; +pi) 而非 \[0; tau)。  
- Lua 环境沙盒化方面有小幅调整，详见下方“Lua 环境沙盒化”部分。  
- 始终将日志输出到标准输出。  
- 移除了 `CSphereParticleSpawner` 粒子类，其功能与 `CSimpleParticleSystem` 相同。  
- 归档缓存版本从 20 更新为 21。

## 功能

### RmlUi

- 使用的 rmlUI 版本从 6.0 更新至 6.2  
- 增加对 pairs 的数据模型支持：`pairs(dm_handle)`  
- 增加对 ipairs 的数据模型支持：`dm_handle:__ipairs()`  
- 支持通过 `dm_handle.__raw()` 访问底层数据模型表  
- 允许数据模型进行自引用赋值，例如：`dm_handle.property = dm_handle.another_property`  
- 支持获取数据模型属性长度：`dm_handle.property.__len()`  
- 修复数据模型数组访问问题  
- 修复 rml 元素中的 `data-value` 绑定  
- 新增 `RmlUi.GetDocumentPathRequests(string docPath) -> {"filePath", "filePath", ...}`，用于跟踪 RmlUi LoadDocument 调用所打开的所有文件  
- 新增 `RmlUi.ClearDocumentPathRequests(string docPath) -> nil`，用于清除已跟踪的 LoadDocument 文件

### 雷达图标 Lua API
- 新增了 `Spring.SetUnitIcon(unitID, string? iconName)`。传入 `nil` 以重置为默认值。
- 新增了 `Spring.GetUnitIcon(unitID) → string iconName`。

### 小地图补丁
- 添加了 `wupget:MiniMapRotationChanged(rotation, previousRotation)` 未同步调用，单位为弧度。  
- 添加了 `wupget:MiniMapGeometryChanged(x, y, sizeX, sizeY, prevX, prevY, prevSizeX, prevSizeY)` 未同步调用，单位为像素。  
- 添加了 `wupget:MiniMapStateChanged(isMinimized, isMaximized, isSlaved)` 未同步调用。

### Lua 环境沙盒化
- LuaSocket 不再在 Lua 沙盒化之前初始化。  
- 未同步的 LuaRules（包括未同步的 LuaGaia）现在可以访问 `io` 和 `os` 库。  
- 未同步的 LuaRules（包括未同步的 LuaGaia）现在默认可访问 `debug` 库（不再需要开发模式）。

### 回放路径获取器
- 添加 `Spring.GetReplayFilePath() → string?`，返回正在观看的回放文件路径。  
- 添加 `Spring.GetReplayRecordingFilePath() → string?`，返回要生成的回放文件路径。请注意，这只是一个潜在的文件路径（在匹配结束前不会写入任何内容），并且有可能对回放进行二次录制。

### 构建命令
- 当建造指令到达队列前端时，建造者现在会立即执行额外的方块检查。这与现有的周期性检查（约 0.4 Hz）并行进行。如果建造区域被硬阻塞（即“红色方块”，而非可回收/移动单位对应的“黄色方块”），则会取消该建造指令。
- 添加 `Spring.SetEngineBuildSquareRendering(bool) → nil`，用于在选择建造指令时禁用原生的建筑网格渲染。
- 添加未同步的调用 `wupget:DrawBuildSquare(unitDefID, x, z, facing, statuses) → nil`，当选择建造指令时触发。Statuses 是一个一维数组，用于表示每个方块的状态：0 表示被阻塞（红色），1 表示已占用（黄色），2 表示可回收（黄色），3 表示开放（绿色）。即使启用了原生绘制，此函数也会被触发。

### 杂项

- 现在支持 UTF-8 文件路径。  
- 部分文件访问现在区分大小写。  
- `/aicontrol` 默认被阻止。请使用 `/aiCtrl PlayerName` 或 `/aiCtrlByNum 123` 启用。  
- `script:AimWeapon` 现在接收相对于单位的朝向和俯仰角，而非世界空间坐标。这意味着位于斜坡上的单位将获得不同的数值。  
- 朝向转换为弧度后，返回值范围变为 \[-pi; +pi) 而非 \[0; tau)。  
- `VFS.GetAvailableAIs()` 返回的条目现在包含新的布尔值 `isLuaAI`。  
- 添加 `Platform.architecture` 字符串，通常为 "x86_64"，目前正在进行支持 "arm64" 的工作。  
- 添加 `Spring.SetCheatingEnabled(bool)`。  
- 添加 `Spring.SetGodMode(bool? controlAllies, bool? controlEnemies)`。  
- 在 LuaUI 中添加 `Spring.GetClosestEnemyUnit(x, y, z, range = inf) → unitID?`。  
- 在 LuaRules 中添加 `Spring.GetClosestEnemyUnit(x, y, z, range = inf, allyTeamID, bool useLoS = true, bool spherical = false, bool requireEnemyToSeePos = false) → unitID?`。  
- 大量 QTPFS 性能优化。  
- 始终将日志输出到标准输出。  
- 添加 `Engine.isHeadless`，仅在未同步时可用。  
- 档案缓存版本从 20 提升至 21。

## 修复

- 修复退出时日志有时未刷新的问题  
- 修复 `CMD[20]` 和 `CMD[105]` 返回旧式别名而非命令的标准名称的问题
