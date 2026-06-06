+++
title = "发布版 105-2590"
author = "sprunk"
translator = "Cru"
+++

这是从版本2511到2024年9月30日**发布的2590版本**之间的更新日志。

# 注意事项
这些条目在迁移时可能需要特别注意：
* 移除了 `movement.allowDirectionalPathing` 模组规则。路径现在始终为方向性路径，所有已知游戏的路径功能均已启用。
* 如果单位拥有 movedef，则 unitdef `waterline` 将被忽略，并通过其新的 `waterline` 定义从 movedef 中获取，除非 movedef 中将新定义的 `overrideUnitWaterline` 布尔值设置为 false。

# 功能

### 移动
* 移除了 `movement.allowDirectionalPathing` 模组规则。路径现在始终为定向路径，即下坡时不会受到速度惩罚。所有已知游戏的定向路径功能均已启用。
* 新增了布尔型 movedef 参数 `allowDirectionalPathing`，默认值为 false。允许 HAPFS（即旧版）路径查找器考虑方向因素。若设为 false，则在下坡时会避免与上坡爬升相同的程度（注意：此为之前的默认行为）。不适用于 QTPFS，因为 QTPFS 无法使用定向路径。
* 新增了布尔型 movedef 参数 `preferShortestPath`，默认值为 false。使 QTPFS 路径查找器在选择路径时忽略来自任何来源的速度修正因子（如地形类型、坡度、水域等）。但仍会避免 0% 的修正因子（即不会尝试穿越完全无法通行的地形）。不适用于 HAPFS，因为 HAPFS 无法使用此参数，将始终考虑速度修正因子。
* 新增了 movedef 中的数值参数 `waterline`，表示单位在水中的深度（以 elmos 为单位），类似于现有单位定义中的 waterline。默认值为：舰船为 1，潜艇为单位宽度（根据 movedef footprint 转换为 elmos），其他单位均为 0。默认情况下，会覆盖单位定义中的现有 `waterline`。  
* 在移动定义中新增了布尔型字段 `overrideUnitWaterline`，默认值为 true。若将其设为 false，则将使用单位定义中的水线。  
* 新增数值型移动定义字段 `separationDistance`，默认值为 0。该值在移动过程中与其他可动对象发生碰撞时被视为额外半径（即不适用于基于冲力的碰撞或建筑物/地形附近的路径规划）。可用于松开紧密的队形，同时不影响单位的路径选择。

### 标尺图
* 新增了“u”型庭院地图方块：不可建造，但可通行。可作为室内“y”的良好替代方案。  
* 新增了“e”型庭院地图方块：不可建造，仅限出口。单位无法从普通方块通往仅限出口的方块（但相邻的仅限出口方块之间可以通行）。  
该设计适用于工厂建筑区域，但需注意它仅影响地面单位（不包括飞机、残骸等），即使地面单位仍可通过非路径方式进入此类方块（例如通过脉冲）。

### 平滑网格
* 新增了 `Spring.RebuildSmoothMesh() → nil` 同步调用，可立即重建平滑网格。  
* 新增了数值调节项 `system.smoothMeshResDivider`，默认值为 2，用于降低平滑网格的分辨率。增大该值可提升性能，但会牺牲精度。  
* 新增了数值调节项 `system.smoothMeshSmoothRadius`，默认值为 40，表示平滑半径（单位：elmos）。

### Lua 组件 API
* 新增了 `Spring.RebuildSmoothMesh() → nil`，参见上方说明。  
* 新增了 `Spring.GetUnitPhysicalState(unitID) → number bitmask`。请参考引擎源码了解各位的含义。仅同步模式可用。  
* 新增了 `Spring.GetUnitArrayCentroid({unitID, unitID, ...}) → numbers x, y, z`。返回数组的质心（平均位置），若数组为空则返回 nil。  
* 新增了 `Spring.GetUnitMapCentroid({[unitID] = any, [unitID] = any, ...}) → numbers x, y, z`。功能相同，但单位 ID 作为键而非值出现在接受的表中。  
* 新增了 `Spring.GetUnitCosts(unitID) → number buildTime, number metal, number energy`。  
* 新增了 `Spring.GetUnitCostTable(unitID) → { metal = number, energy = number }, number buildTime`。请注意，buildTime 不是常规资源，需单独返回。  
* 新增了 `Spring.GetTeamDamageStats(teamID) → number damageDealt, number damageReceived`。与 `Spring.GetTeamStatsHistory` 中已有的数值相同，但无需承担大部分开销。

### 杂项与修复
* 地形率发生了变化（通过恢复命令，或施工前的地面平整）。
* 多项通用性能改进。
* 修复了当定义了 `wupget:UnitArrivedAtGoal` 时，堆栈警告信息频繁出现的问题。
* 修复了工厂单位配置文件中 `canAssist` 可以设置为 true 的问题。
* 修复了在 Unix/Wayland 系统上使用 `Spring.WarpMouse` 时鼠标无法正确平移的问题。
* 修复了单位配置文件中 `corpse` 字段未被清理的无效尸体名称问题。
