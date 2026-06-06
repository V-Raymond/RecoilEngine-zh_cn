+++
title = "发布版 105-2314"
author = "sprunk"
translator = "Cru"
+++

这是从 1775 版本发布 **到2314版本发布**（即 2024年 2月13日）之间的更新日志。

# 注意事项
这些条目在迁移时可能需要特别注意：

### 移除
* 已移除 `gl.GetMatrix` 以及通过返回的矩阵对象可访问的接口。这些接口显然已不再使用，因此目前暂无替代方案说明。
* 已移除 `spairs`、`sipairs` 和 `snext`。这些函数多年来一直与常规的 `pairs`、`ipairs` 和 `next` 相同，建议改用标准版本。
您可以在迁移前替换这些函数，目前已知的部分游戏已收到补丁以支持此更改。
* 已移除 `VFS.MapArchive` 和 `VFS.UnmapArchive`。它们存在严重的同步安全问题。希望未来能恢复使用，但目前尚无具体时间表。在此期间，请使用 `VFS.UseArchive`。
* 已移除 LuaUI 对 `Script.LuaRules` 和 `Script.LuaGaia` 的访问权限。
* 已移除最近添加的 `Spring.MakeGLDBQuery` 和 `Spring.GetGLDBQuery`。

### 行为变更
* QTPFS 经过重大改版，包含多个模块规则和行为的变更。详见下文部分。
* 现在许多无效的定义条目会导致单位被拒绝；但另一方面，许多（尤其是金属成本和武器伤害）现在可以设置为 0。请注意避免除以零的情况！
详细信息请参见下方“定义有效性检查”部分。
* 纳米炮塔（不可移动建筑者）的建造范围现在只需到达建筑物半径边缘即可，而不再需要到达中心位置。可移动建筑者此前已采用此方式。
* 通过 `Spring.SetUnitMoveGoal` 执行的原始移动现在不仅在单位到达目标时视为完成，而且当其触碰到位于目标位置的其他单位时也视为完成。自版本 105-2314 起，目前尚无简便方法检测到这种情况发生。
* 截图将使用 UTC 时间戳作为后缀，而非数字。
* 新增布尔型弹簧设置项 `SMFTextureStreaming`，默认值为 false。若设为 true，则会动态加载和卸载 SMF 散射纹理，有助于节省显存，但会导致性能下降和图像质量变差。
此前的行为等同于 `true`，因此如果遇到显存问题，请尝试更改该设置。
* 现在可以使用固定或随机起始位置，并且允许更多队伍参与，超出地图所设定的限制。额外内容默认从 (0, 0) 角度开始，现在由游戏正确处理此情况。  
* `movement.allowGroundUnitGravity` 模块规则现在默认为 `false`。所有已知游戏都已明确设置该值，因此此更改仅影响新游戏。  
* `/ally` 命令不再通过控制台消息向无关玩家通知此信息。受影响的玩家仍会看到该消息。  
如需公开显示，请使用 `TeamChanged` 调用进行替换。  
* 手动共享单位不再收到 Stop 命令。请使用 `UnitGiven` 调用恢复之前的正常行为。  
* 默认启用 `DumpGameStateOnDesync` Spring设置

### 弃用
目前还没有变化，但这些将在未来发生，并可能造成一些问题。

* `acceleration` 和 `brakeRate` 的单位定义条目已安排在从 elmo/帧更改为 elmo/秒时进行更新。目前尚未更改，但如果您希望避免后续添加处理逻辑，可以考虑分别改为 `maxAcc` 和 `maxDec`（这些将保持为 elmo/帧）。
* `CSphereParticleSpawner`（别名 `simpleparticlespawner`）CEG 类已被安排移除。由于其行为一直相同，仅内部实现不同，因此可完全用 `CSimpleParticleSystem`（别名 `simpleparticlesystem`）替换。使用该类的已知游戏将在此之前收到补丁请求。
* 现在已有明确机制用于生成空白地图（见下文）。现有的“随机”地图生成器（始终生成空白地图）将保持不变，但未来可能会被重新设计为“真实”的随机地图生成器（尽管目前暂无计划）。
* `UsePBO` Spring设置现已弃用。不过它本身实际上并无作用。

# QTPFS

QTPFS 路径查找器已进行全面升级。在质量方面（如减少设备卡住、省略步骤等情况）和性能方面（处理速度、内存使用，甚至磁盘使用）均有显著提升。QTPFS 的各个方面通常都会表现更佳。快来试试吧！

* 调试路径绘制器现在会绘制到缩略图中，显示等待处理的地图损坏更新。颜色越深，表示仍需处理变更的层数（MoveTypes）越多。  
* 新增模组规则 `system.pfRepathDelayInFrames`，用于控制在检查单位是否已足够前进至当前目的地或是否需要请求新路径之间至少需要经过多少帧。默认值为60（2秒）。可调整以找到单位脱困速度与CPU使用量之间的平衡点。  
较小的时间间隔会增加单位因缓慢而触发不必要的重新路径请求的可能性，但也会降低玩家误以为单位卡住、无法正常移动的风险。  
* 新增模组规则 `system.pfRepathMaxRateInFrames`，用于控制单位在允许请求新路径前必须经过的最少帧数。默认值为150帧（5秒）。  
这主要用于限制请求频率，避免过度消耗CPU资源，因为处理路径请求的成本并不低。  
* 新增模组规则 `system.pfUpdateRateScale`，这是更新速率的倍数因子，默认值为1。增加数值可获得更快的更新，但会增加CPU使用量。  
* 新增配置项 `system.pfRawMoveSpeedThreshold`，用于控制单位在何种速度修正（包括类型映射加成和上下坡修正）下将不再进行原始移动，无论距离等条件如何。默认值为0，表示单位不会尝试进入无法通行的地形（例如类型映射的熔岩、悬崖、水域）。你可以设置一个正数，使单位避开可通行但速度极慢的地形（例如设为0.2，则单位在速度低于20%或更低时将不会进行原始移动，而改用常规路径寻路——这仍可能导致其通过该路径）。  
* 新增配置项 `system.qtMaxNodesSearched`，可用于限制搜索节点的绝对数量。  
* 新增配置项 `system.qtMaxNodesSearchedRelativeToMapOpenNodes`，可用于限制相对于地图上可通行四边形数量的搜索节点数。最终限制以相对限制和绝对限制中较大的值为准。  
* 新增配置项 `system.pfHcostMult`，一个介于0到2之间的浮点数，默认值为0.2。控制路径搜索在朝向目标方向时，对节点的优先级排序程度。  
数值越高，路径成本越低，但可能导致生成退化路径，即单位直接朝向目标前进，随后不得不紧贴墙壁移动。  
* 新增配置项 `system.qtRefreshPathMinDist`，可用于设置路径的最小尺寸，以确保其具备自动重新规划不完整路径的资格。  
* 移除配置项：`system.pfForceUpdateSingleThreaded` 和 `system.pfForceSingleThreaded`。多线程已被证明运行稳定。  
* 移除配置项：`system.pathFinderUpdateRate`，因为 `system.pfUpdateRateScale` 现在承担了相同的功能，但单位不同。  
* 新增地图配置项 `maxNodesSearched`，位于 mapinfo 中的 `(info).pfs.qtpfsConstants` 子表中，用于控制搜索节点的绝对上限。  
仅可增加该配置项限制，不可降低，默认值为 0，适用于包含大范围但无连接区域的地图（例如熔岩与两座小山）。  
* 新增**地图（Map）**配置项 `maxRelativeNodesSearched`，用于控制相对于地图总节点数的相对搜索上限。  
仅可增加该配置项限制，不可降低，默认值为 0。

# 统一定义

单位定义（即 `/units/*.lua`）和 `UnitDefs`（在 wupgets 中）以不同名称引用相同内容，有时甚至使用不同的计量单位，一直是一个令人困惑的问题。  
部分问题已得到缓解，因为许多不匹配的键现在有了统一的命名方式。通常这些名称在分界线的两侧都已存在。

## 定义新键位
以下单位定义键现在接受与 `UnitDefs` 显示的相同拼写。
旧拼写仍然有效（旧 → 新）。
* metalUse → metalUpkeep
* energyUse → energyUpkeep
* buildCostMetal → metalCost
* buildCostEnergy → energyCost
* unitRestricted → maxThisUnit
* name → humanName

这两个也接受新的拼写形式，旧拼写和新拼写都位于 elmo/frame 中。  
但请注意，请尽快迁移至新拼写形式，因为原始拼写将在未来某个时候被修改为使用 elmo/s。
* acceleration → maxAcc  
* brakeRate → maxDec

以下单位定义键现在可以接受通过 `UnitDefs` 显示的拼写和计量单位（旧 → 新）。旧的拼写仍然有效，且仍使用旧的计量单位。  
* maxVelocity（elmo/帧）→ speed（elmo/秒）  
* maxReverseVelocity（elmo/帧）→ rSpeed（elmo/秒）

以下单位定义键现在支持一种新的拼写，该拼写此前在 `UnitDefs` 中未被提供，但在此更新中已添加到其中。  
旧的拼写仍然有效。  
* losEmitHeight → sightEmitHeight  
* cruiseAlt → cruiseAltitude

添加了 `fastQueryPointUpdate` 到武器（注意，这是单位定义中的条目，同时设置目标类别和方向；不是武器定义！）。  
启用后，脚本中的 `QueryWeapon` 系列函数将每帧调用一次（而不是慢速更新时的每15帧一次），从而解决多管快速射击武器的友军火力问题。

## 新的 `UnitDefs` 成员
以下 `UnitDefs` 键现在接受与单位定义文件中相同的拼写。旧的拼写仍然有效（旧 → 新）。
* tooltip → description
* wreckName → corpse
* buildpicname → buildPic
* canSelfD → canSelfDestruct
* selfDCountdown → selfDestructCountdown
* losRadius → sightDistance
* airLosRadius → airSightDistance
* radarRadius → radarDistance
* jammerRadius → radarDistanceJam
* sonarRadius → sonarDistance
* sonarJamRadius → sonarDistanceJam
* seismicRadius → seismicDistance
* kamikazeDist → kamikazeDistance
* targfac → isTargetingUpgrade

以下键的拼写已更新，这些拼写此前未在单位定义中使用过，但在此更新中已添加。  
旧拼写仍可正常使用。  
* losHeight → sightEmitHeight  
* wantedHeight → cruiseAltitude

在 UnitDefs 中添加了缺失的 `radarEmitHeight`。该单位定义文件中的键也已为 `radarEmitHeight`。

## 有效性检查

部分无效和/或缺失的定义现在处理方式已有所改变。

* 生命值、（反向）速度以及金属/能量/建造时间的负数值现在会导致单位定义被拒绝；此前这些值会被限制为0.1  
* 加速和制动速率的负数值现在会导致单位定义被拒绝；此前需取绝对值  
* 生命值和建造时间的允许范围现在为[0; 0.1]（0仍不被允许）  
* 金属成本的允许范围现在为[0; 0.1)（0现在被允许）  
* 未定义的金属成本现在默认为0，而非1  
* 未定义的生命值和建造时间现在各自默认为100，而非0.1  
* 武器的`edgeEffectiveness`现在可以为1（此前上限为0.999）  
* 单位装甲倍数（即`damageModifier`）现在可以为0（此前上限为0.0001）  
* 武器定义中的伤害值现在可以为0（此前上限为0.0001）  
* 伤害和装甲值再次可以为负数（以便目标被治愈），但请注意，武器始终会攻击敌人而不会攻击盟友，因此请避免在非手动触发场景、死亡爆炸等情况下使用

### 已弃用的 UnitDefs 移除

所有已弃用的 UnitDefs 键（返回零并产生警告）均已移除，列表如下：  
* techLevel  
* harvestStorage  
* extractSquare  
* canHover  
* drag  
* isAirBase  
* cloakTimeout  
* minx  
* miny  
* minz  
* maxx  
* maxy  
* maxz  
* midx  
* midy  
* midz

# 功能与修复

### 建造行为
* 纳米塔楼（固定建筑者）的建造范围现在只需到达建筑目标半径边缘，而非中心位置。移动建筑者已采用此方式。
* 新增了 `buildeeBuildRadius` 单位定义条目，用于计算建筑时的半径（即放置纳米框架后进行纳米车削）。
负值将使用模型本身的半径（默认值），设置为零则要求建筑者必须到达中心位置。
* 新增了 `Spring.Get/SetUnitBuildeeRadius(unitID)`，用于动态控制上述半径。
* 修复了建筑者无法从最大距离内放置纳米框架的问题。
* 单位离开建造区域（即“bugger off”）时，现在会尝试选择最快的出口路径。
* “bugger off” 现在能正确应用于位于区域外但足部足够大、可阻挡建筑的单位。
* 新增了 `Spring.GetUnitWorkerTask(unitID) → cmdID, targetID`，类似于 `Spring.GetUnitCurrentCommand`，但显示的是该单位当前实际执行的操作，因此当单位处于守卫状态或超出范围时，结果会有所不同。同时解决了“建造”与“维修”的区别，仅显示工人任务（即与纳米车削相关的操作）。
* `gadget:AllowUnitCreation` 现在具有两个返回值。第一个仍然是布尔值，用于决定是否允许创建单位（此处无更改）。  
新的第二个值为布尔值，表示如果创建被禁止时是否丢弃订单（默认为 true，即之前的默认行为）。  
若设置为 false，建造者或工厂将继续重试。  
* 新增 `Spring.GetUnitEffectiveBuildRange(unitID[, buildeeDefID]) → number`。返回指定建造者对目标建造对象中心的有效建造范围，  
即与引擎测量建造距离的方式相同。适用于设置原始移动订单的目标半径。  
目前该函数尚未解决所有已知情况（如未处理特征或土地改造等），这些功能尚待完善；对于当前情况，若 `buildeeDefID` 为 nil，则仅返回建造范围。  
* 新增 `Spring.GetUnitIsBeingBuilt(unitID) → bool beingBuilt, number buildProgress`。请注意，此函数并未带来新能力，因为 `buildProgress` 已经在 `Spring.GetUnitHealth` 的第五个返回值中提供，而 `beingBuilt` 则在 `Spring.GetUnitIsStunned` 的第三个返回值中提供，但此前使用不够方便或直观。  
* 修复了可放置需要地热通风口的单位到任意位置的问题。

### 规则参数
* 增加了玩家规则参数，通过新接口控制：`Spring.SetPlayerRulesParam`、`GetPlayerRulesParam` 和 `GetPlayerRulesParams`，与其他现有规则参数类似。目前有两个可见性级别：公共和私有。一个显著区别是，私有级别仅对当前玩家可见，不包括其队友团队，甚至不包括其（共用）团队；这在部分是出于技术原因，可根据需要进行更改。  
同步系统和规格系统可查看所有内容，但尚未提供给小规模战斗AI接口使用。  
* 为规则参数添加了布尔值支持，包括新的玩家规则参数。  
小规模战斗AI和单位规则参数选择过滤器可通过现有的数值接口读取这些参数，即0和1。

### 地图纹理
* 新增了布尔型Spring设置 `SMFTextureStreaming`，默认值为 false。若设为 true，则会动态加载和卸载 SMF 散射纹理，以节省显存，但会导致性能下降和图像质量变差。  
此前的行为等同于 `true`，因此如果遇到显存问题，建议尝试更改此设置。  
* 新增了数值型弹簧设置 `SMFTextureLodBias`，默认值为 0。当 `SMFTextureStreaming = false` 时，该参数控制散射纹理的 LOD 取样偏移量。  
* 新增了第五个整数参数至 `Spring.GetMapSquareTexture(x, y, lodMin, texName[, lodMax = lodMin])`。该参数用于控制最大 LOD 值，默认值为第三个参数，现在该参数表示最小 LOD（而非最终值）。

### FFA 支持
* 现在可以使用比地图指定更多的队伍进行固定和随机起始位置的设置。  
额外队伍默认从 (0, 0) 角落开始，游戏需正确处理此情况。  
* `/ally` 命令不再通过控制台消息向无关玩家广播该信息，受影响的玩家仍会看到该消息。  
如需公开显示，请使用 `TeamChanged` 调用来替换。  
* 新增 `system.allowEnginePlayerlist` 参数，默认为 true。若设为 false，则内置的 `/info` 玩家列表将不会显示。  
可用于“匿名玩家”模式，并配合 `Spring.GetPlayerInfo` 毒化功能，或仅用于避免混乱。  
* 新增 `Spring.SetAllyTeamStartBox(allyTeamID, xMin, zMin, xMax, zMax) → nil`，用于在 elmos 中设置指定友方队伍的起始框边框。

### VFS
* 在 `VFS.DirList` 和 `VFS.SubDirs` 中添加了第四个布尔参数，默认值为 false。如果设置为 true，则搜索将递归进行。
* 修复了 `VFS.SubDirs` 在 `VFS.RAW` 模式下，不会仅对 `VFS.RAW` 中的单个文件夹名称应用传入的模式，而是会对整个路径应用该模式的问题。

### 单位选择
* 新增了 `Spring.DeselectUnit(unitID) → nil`。
* 新增了 `Spring.SelectUnit(unitID[, bool append]) → nil`，这是 `Spring.SelectUnit{Array,Map}` 的单个单位版本，无需传入表。
unitID 可以为 nil。
* 新增了 `Spring.DeselectUnitArray({[any] = unitID, [any] = unitID, ...}) → nil` 和 `Spring.DeselectUnitMap({[unitID] = any, [unitID] = any, ...}) → nil`。
这些是现有 `Spring.SelectUnitArray` 和 `Spring.SelectUnitMap` 的对应版本。
* `Spring.SelectUnitArray` 中的表现在可以拥有任意键。此前这些键必须是数字，但该表实际上并不需要是数组。

### 根部部件
* 新增了 `Spring.GetModelRootPiece(modelName) → number pieceID`，用于返回根部件。
* 同样新增了 `Spring.GetUnitRootPiece(unitID) → number pieceID` 和 `Spring.GetFeatureRootPiece(featureID) → number pieceID`。

### 彩色文字
* 添加了内联颜色代码 `\254`，后跟8个字节：RGBARGBA，其中前四个描述文本颜色，后四个描述文本边框。
* 添加了 `Game.textColorCodes` 表，包含常量 `Color`（`\255`）、`ColorAndOutline`（新添加的 `\254`）和 `Reset`（`\008`）。

### 其他附加内容
* 添加 `Spring.IsPosInMap(x, z) → bool inPlayArea, bool inMap`，目前返回的两个值相同，仅用于检查位置是否在地图矩形内。未来或当游戏重写该函数时，可能会出现有限的游戏区域情况（例如 SupCom 单人模式的地图扩展、公元 0 年的圆形地图，或仅仅是外部装饰区域）。
* 添加 `Spring.GetFacingFromHeading(number heading) → number facing` 和 `Spring.GetHeadingFromFacing(number facing) → number heading`，用于单位坐标转换。
* 新增 `wupget:Unit{Entered,Left}Underwater(unitID, unitDefID, teamID) → nil`，类似于现有的 `UnitEnteredWater`。注意：`EnteredWater` 表示单位将脚尖浸入水中，而 `EnteredUnderwater` 表示单位完全被水淹没。
* 新增一个仅限作弊的 `/remove` 命令，与 `/destroy` 类似，但会直接移除所选单位（不造成残骸，也不触发死亡爆炸）。
* 新增启动脚本条目：`FixedRNGSeed`。默认值为 0，表示为同步随机数生成器生成随机种子（当前行为）。否则，使用给定值作为种子。适用于可重现的运行场景（如基准测试、任务剪辑等）。* 新增了 `Script.DelayByFrames(frameDelay, function, args...)`。**请注意**，这是 `Script` 而不是 `Spring`！它会在指定帧数（至少 1）的延迟后执行 `function(args...)`。  
多个函数可以排队到同一个帧上，并按添加顺序在该帧的 `GameFrame` 调用前运行，用于避免在 GameFrame 中手动跟踪。  
* 新增了 `Spring.GetUnitSeismicSignature(unitID) → number` 和 `Spring.SetUnitSeismicSignature(unitID, number newSignature) → nil`。  
* 新增了 `Spring.SetUnitShieldRechargeDelay(unitID, [weaponNum], [seconds]) → nil`。重置单位护盾再生延迟。  
如果单位只有一个护盾，则武器编号为可选参数。定时器值也是可选的：若设为 `nil`，则模拟武器命中。  
请注意，无论是此处的 `nil` 还是“真实”命中，都不会减少剩余计时器，但可能会增加它。  
明确的数值设置将始终将计时器设为相应秒数。  
* 新增了数值型弹簧设置 `MaxFontTries`，默认值为 5。表示渲染文本时搜索字形替换的最大尝试次数（数值越小，外文字形可能无法正确显示；数值越大，搜索外文字形可能导致游戏卡顿）。搜索会优化地优先查找“最佳”位置，因此找到字形的概率不会随搜索时间线性增加。  
* 新增了 `GL.DEPTH_COMPONENT{16,24,32,32F}` 常量。  
* 新增了用于 `gl.BlendEquation` 的以下 `GL` 常量：`FUNC_ADD`、`FUNC_SUBTRACT`、`FUNC_REVERSE_SUBTRACT`、`MIN` 和 `MAX`。  
* 新增了 `Spring.GetWindowDisplayMode()` 方法，返回：宽度、高度、每像素位数、刷新率（Hz）、像素格式名称。  
像素格式名称示例如 "SDL_PIXELFORMAT_RGB565"。  
* 新增了 `/dumpatlas 3do` 命令。

### 空白地图生成
* 新增一种指定空白地图颜色的方式：`blank_map_color_{r,g,b}`，数值范围为0-255。  
* 新增一个启动脚本条目 `InitBlank`（位于根级别），作为现有 `MapSeed` 的别名。  
* 新增内置地图选项 `blank_map_x` 和 `blank_map_y`，分别对应现有 `new_map_x/y` 的别名。

### 武器修复
* 修复当因目标提前预测导致无法找到物理射击解时，`Cannon` 类型武器瞄准世界坐标 (0, 0) 角落的问题  
* 修复 `Cannon` 类型武器在越过友军开火时，友军伤害规避过于宽松的问题  
* 修复 `MissileLauncher` 武器在高 `trajectoryHeight` 情况下未能正确执行地面和友军规避的问题  
* 修复 `MissileLauncher` 武器在高 `trajectoryHeight`、零 `turnRate` 和高 `wobble` 情况下，向较高海拔目标发射时弹道不稳定并过早下坠的问题  
* 修复 `DGun` 武器类型的弹道方向（此前发射角度基于 `AimFromWeapon` 部件而非 `QueryWeapon` 部件的合理角度）

### 基础内容修复
* 将 `cursornormal` 光标从“Spring光标”存档移至 basecontent。此举的意义在于，现在 `modinfo.lua` 即可确保存档成为一个有效的 Recoil 游戏，不会崩溃。
* 修复了初始生成装置的问题。
* 修复了动作处理程序中的按键按下/释放事件。

### 各类修复
* 修复了悬浮船/飞船移动类型在垂直悬崖上可侵入陆地的问题。  
* 修复了 skirmish AI API 的 getTeamResourcePull 方法（原用于返回最大存储量，现已更正）。  
* `Spring.SetSunDirection` 不再因传入非归一化向量而造成阴影异常。  
* 修复了使用 `/specfullview 0` 时无法拖拽选择单位的问题。  
* 修复了加载 Assimp（`.dae`）模型时出现骨骼异常的错误。  
* 修复了 COB 的 `SetMaxReloadTime` 接收比预期小 10% 的数值问题。  
* 修复了截图保存为 PNG 格式后文件大小膨胀的问题（由于多余的完全不透明 alpha 通道导致）。  
* 修复了加载画面中点击有时被误识别为后续开始框放置的问题。  
* 修复了通过 `Spring.UnitAttach` 自动附着时崩溃的问题。  
* 修复了 `Spring.MoveCtrl.SetMoveDef` 与数值型移动定义 ID 无法正常工作的问题。
