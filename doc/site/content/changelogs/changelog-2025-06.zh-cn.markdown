+++
title = "发布版 2025.06"
aliases = ['/changelogs/changelog-2025-06']
+++

这是从2025.04版本到**2025.06.21版本**的更新日志，该版本于2026年4月2日发布。

## 注意事项

- 重新计算了被抛向空中地面单位的空气阻力。
- 单位着色器现在接收的是TRS变换而非矩阵，导致着色器功能失效，并无法实现各向异性部件缩放。详见下方“平滑脚本部件动画”部分。
- 非炮塔单位现在在需要时会自动旋转以朝向目标。详情见下文。
- 移除了 `^123` 格式对单位定义图标纹理的引用，适用于 `gl.Texture` 及类似接口。现新增了一组与图标相关的调用提示，详见下文。
- 移除了AI的Python绑定。该功能显然未被维护且已不再使用。
- 移除了 `UpdateWeaponVectorsMT`、`UpdateBoundingVolumeMT` 和 `AnimationMT` 的弹簧设置。这些原本是作为备选方案，但经过一段时间的实际测试后，MT模式已足够安全。
- 移除了 `/AdvModelShading` 命令以及 `AdvUnitShading` 弹簧设置。高级模式现已默认开启。实际上由于GLSL已成为强制要求，因此实际并无差异。
- 移除了 `gl.UnitGL4`、`gl.FeatureGL4`、`gl.FeatureShapeGL4` 和 `gl.UnitShapeGL4`。这些函数并未执行任何操作。
- 伤残（原生收入倍率）不再适用于回收功能。如需替代，请使用 `gadget:Allow{Unit,Feature}BuildStep`。向后兼容性功能检查：`Engine.FeatureSupport.noHandicapForReclaim`。  
- `/group add N` 不再选择整个组，仅添加成员。`/groupN`（无空格）不受影响。功能支持标签：`Engine.FeatureSupport.groupAddDoesntSelect`。  
- 拥有 `canKamikaze` 的不可移动单位不再忽略 `blocking` 单位定义标签。  
- `BeamLaser` 和 `LightningCannon` 类型武器现在会将实际的弹药ID传递给调用函数，而不是使用 `-1`。  
- `BeamLaser` 和 `LightningCannon` 现在能正确遵循椭球体和/或圆柱体目标体积，并正确遵守 `targetBorder` 标签，这会导致有效射程体积略有不同。  
- 新增了大量模组规则，用于配置 Guard 命令单位相对于其守护者移动的方式，详见下文。需要注意的是，默认值也已更改。  
- 重放文件名现在包含完整的地图名称，而非截断至第一个点号。  
- 内联文本颜色代码：0x11 和 0x12（十进制 17 和 18）现在可作为颜色代码指示符使用，此外还支持 255 和 254。  
`Engine.textColorCodes` 仍只列出旧的代码。  
- 引擎不再加载 `shaders/GLSL/GroundDecalsFragProg`。直接使用 `glsl` 和 `shaders/GLSL/GroundDecalsVertProg.glsl`。
现在引擎会加载 `shaders/GLSL/groundDecals.lua`，该文件应返回一个 `{ vertex = "path", fragment = "path" }` 表。
- `LogRepeatLimit` 的默认值从 10 改为 0（参见下文）。
- “区域复活”命令现在会在修复子命令上添加牵引绳（行为变更，同时修复子命令现在有 5 个参数，而非 1 个）。
- 小部件现在可以注册接收 `widget:Explosion` 调用。存在一些复杂的可见性规则，最终表现为比现有的 `widget:ShockFront` 调用暴露更多信息。
- 移除了 10 个修改器限制，现在可以拥有任意数量的修改器（这可能影响假设了该限制的公会、自动主持人等）。
- `Spring.ShareTeamResource` 不再触发 `gadget:AllowResourceTransfer`，如有需要请手动调用。
- rmlUI 纹理现在使用最近邻像素过滤，而非线性过滤。预期效果是文字更清晰，并带有抗锯齿。
- `Spring.SetProjectileTarget` 现在对无效参数报错，不再静默忽略。
- 武器定义中的 `smokePeriod` 默认值从 8 改为 1。
- 设置武器定义的 `stockpileTime` 为负数不再允许。- 完全移除了对32位构建的支持。

### 弃用通知

- `wupget:DrawUnit`、`DrawFeature`、`DrawShield` 和 `DrawMaterial` 已被弃用。
行为无变化，但它们长期以来一直是一个性能上的反模式，不建议使用。
请改用着色器替代。
- 已弃用的 `Spring.GetUnitCommands(unitID, 0) → number commandCount`，应改为 `Spring.GetUnitCommandCount(unitID)`。
非零参数的重载版本仍返回命令表，不受影响。
- 同样已弃用的 `Spring.GetFactoryCommands(unitID, 0)`，应使用新的 `Spring.GetFactoryCommandCount`。
- 已弃用的 `Spring.GetCommandQueue`，应使用 `Spring.GetUnitCommands`（该函数始终与之完全等效），仅上述情况除外。
- 已弃用 `ghostedBuildings` 引擎选项。可手动将其作为模组选项读取，并设置所有单位定义不留下幽灵建筑。
- 未来某个时间点，引擎将停止向死队的盟友重新分配单位上限。详情请参见下方的 `FeatureSupport.deadTeamsKeepUnitLimit` 和 `Spring.TransferTeamMaxUnits`。

## 功能

### 非炮塔单位朝向目标旋转

一系列旨在帮助提升近战单位的改动。

- 没有炮塔的单位如果其他所有武器均未通过瞄准检查，将尝试朝目标方向移动。这种情况既可能发生在单位射程内（频率较高），也可能发生在射程外（频率较低）。
- 如果瞄准点被截断在目标内部，则角度射击容错检查将不适用（并非所有情况都会发生，因此近距离近战武器/单位仍需注意该容错机制）。
- 目前，上述更改不会正确处理非炮塔、非正面朝向的武器。
- 可能存在未知的副作用。此为测试性改动。

### 图标图集

- `gl.Texture` 及类似函数不再支持通过 `^123`（单位定义 ID）引用图标纹理。
- 现在可以在 `gamedata/icontypes.lua` 中直接指定图标图集，通过设置 `u0`、`v0`、`u1` 和 `v1`（浮点数范围 0-1）。这种方式不会构建图集，而是直接加载。
- `Spring.AddUnitIcon` 现在可以处理带有子坐标的图集输入纹理（参数 6-9 现在分别对应 u0、v0、u1、v1）。
- 新增 `$icons`、`$icons0` 和 `$icons1` 纹理。它们分别表示游戏加载时看到的初始图标图集，以及某些被替换图标可能创建的备选图集。
- 新增 `Spring.GetUnitIconData`、`Spring.GetIconData`、`Spring.GetAllIconData` 函数，主要用于获取图集坐标。具体细节请参见 API 列表，接口的确切实现仍可能发生变化。

### 相机调焦

- 添加了 `wupget:CameraRotationChanged(rotX, rotY, rotZ) → nil`。
- 添加了 `wupget:CameraPositionChanged(posX, posY, posZ) → nil`。

### 通过小部件聊天

- 新增 `Spring.SendPublicChat(msg) → nil`，用于发送聊天消息，相当于 `/say`。
- 新增 `Spring.SendAllyChat` 和 `Spring.SendSpectatorChat`，接口与上述相同，用于向盟友或旁观者发送聊天消息，相当于使用带有“a:”或“s:”魔法目的地前缀的 `/say`。
- 新增 `Spring.SendPrivateChat(msg, playerID) → nil`，用于向特定玩家ID发送私信（即耳语）。
几乎等同于 `/wbynum`，但不支持魔法目的地 255。

### GL 调试

- 新增了布尔型配置项 `DebugGLReportGroups`，默认为 true。用于在 OpenGL 调试中显示 push/pop 群组。
- `/debugGL` 选项现在可接受一个可选的数值参数，范围为 0 到 15。
值 0 和 1 会控制整个调试视图，而不会影响其他设置（即与之前相同）。
其余值 2 至 15 作为位掩码处理：8 控制堆栈跟踪，4 报告群组，2 控制整个调试的开启/关闭状态，1 被忽略。

### 新网站

- 为了逃避大学期末考试，Slashscreen 完全重写了网站。
- 重新整理了 API 文档。
- 新增了使用指南。
- 记录了 RmlUi 绑定方式。

### CEG 渲染

- 向 `CBitmapMuzzleFlame` 类型的 CEG 粒子添加了以下标签：`particleSpeed`、`particleSpeedSpread`、`airdrag`、`gravity`。  
这些标签的功能与 `CSimpleParticleSystem` 类型粒子相同。  
- 在 CEG 中新增了 `animParams1` 和 `animParams2`，类似于武器的参数设置。  
纹理 1 用于 `CSimpleParticleSystem`，以及 `CBitmapMuzzleFlame` 的“前部”纹理；  
纹理 2 用于 muzzle flame 的“侧面”纹理。  
其他 CEG 类目前尚不支持此功能。  
使用无编号的 `animParams` 作为默认后备方案。  
- 向 `CBitmapMuzzleFlame` 类型的 CEG 添加了 `fixedSideDir`（布尔类型，默认为 false），用于尝试调整侧面平面，使其相对于粒子到摄像机方向向量呈 45 度角，并相互垂直呈 90 度角。

### Infotex 命令

- 新增了 `/showLoS` 命令。
- 新增了 `/showInfoTex foo` 命令，其中 `foo` 可以是现有的信息纹理之一（例如，`/showInfoTex elevation` 等同于 `/showElevation`）。
- 所有 `/showFoo` 命令现在除了支持无参数开关外，还可接受 0/1 作为参数。

### 流畅的脚本片段动画

- 单位着色器现在接收的是 TRS 变换，而不是矩阵，这会破坏着色器以及各向异性片段缩放功能。  
- 这将要求游戏开发者更新那些原本使用引擎提供的变换矩阵 SSBO 的 GL4 着色器。  
- 新方法应更快速，并能实现更平滑的动画（在游戏帧之间进行插值）。  
- 新格式目前未在任何地方直接文档化，但可以参考 basecontent 着色器。  
- 新增了 `Engine.FeatureSupport.transformsInGL4` 用于后向兼容性检查。

### Base64 编码

- 新增了 `Encoding` 全局表（类似于 `Spring` 或 `math` 等），可在任何地方使用。目前包含 base64 编码，后续将添加更多功能（如 JSON）。
- 新增了 `Encoding.EncodeBase64(string plain, bool? stripPadding = true) → string encoded` 方法，可选择性地移除字符串末尾的额外 `=`。
- 新增了 `Encoding.DecodeBase64(string encoded) → string plain` 方法，用于解码 base64 字符串（如果包含无效字符，则仅解析有效前缀）。
- 新增了 `Encoding.IsValidBase64(string) → bool` 方法。
- 在 `Encoding` 类中新增了 `EncodeBase64Url`、`DecodeBase64Url` 和 `IsValidBase64Url` 方法。与上述方法相同，但使用 base64url 字符集。

### 重复的日志

- `LogRepeatLimit` 模块设置，用于控制连续多少次重复日志会最终进入信息日志，现在可以接受 0 表示无限。
- `LogRepeatLimit` 的默认值：10 → 0
- 内置游戏控制台不再过滤重复日志（最多连续 1 次 → 无限）

### 支持GLTF/GLB模型格式

- 现在支持 `.gltf` 和 `.glb` 模型格式。  
- GLTF 解析器会将若干空片段插入为根节点或子根节点。  
- 不支持嵌入在 GLTF 本身中的材质、纹理和顶点颜色，主要原因是 GLTF 预期使用 PBR 工作流，而标准引擎着色器不支持该功能。  
- GLTF 可使用类似 Assimp 的 Lua 元数据文件，也可以通过自定义的 GLTF 模型属性提供相同的数据。支持现有标签：`tex1`、`tex2`、`midpos`、`mins`、`maxs`、`height`、`radius`、`fliptextures`、`invertteamcolor`。  
- 新增 `s3ocompat` 布尔属性，适用于 GLTF 和 Assimp 模型。启用与 s3o 中相同的右手到左手模型翻转功能。此功能尚属实验性，可能存在较多问题。如果您的模型从 s3o 转换后动画显示异常，请尝试启用此选项。  
- 不支持重新父级或烘焙旋转角度。请使用 3D 编辑器进行修改。Assimp 无影响。

### 幽灵

- 添加 `leavesGhost` 单位定义标签，用于控制单位是否留下幽灵。建筑默认保留幽灵（保持原有行为）。
- 添加 `leavesGhost` 单位定义条目，功能相同。
- 添加 `Spring.GetUnitLeavesGhost(unitID) → bool`。
- 添加 `Spring.SetUnitLeavesGhost(unitID, bool leavesGhost, bool? existingGhostRemains = false)`。当 `leavesGhost` 为 false 且单位已存在幽灵时，现有幽灵的移除或保留由第二个参数控制。
- 注意：`ghostedBuildings` 引擎选项仍然存在，会使得上述所有接口失效，但计划在后续版本中移除。

### 文本着色

- 新增了 `AllowColorFonts` 布尔型春季设置，默认为 false。允许字体使用彩色显示而非单色显示。实际上这意味着表情符号能够正常渲染，但一个足够精致的字体也可能对普通文本产生影响。
- 行内文字颜色代码：0x11 和 0x12（十进制 17 和 18）现在也可作为颜色代码指示符，除了原有的 0xFF 和 0xFE（十进制 255 和 254）之外。其意义在于，255/254 是来自合理语言中常见的可打印字符，将其视为颜色代码可能会破坏原本正常的文本，因为它们会消耗字符并设置随机颜色。
- `Engine.textColorCodes` 仍仅列出旧的代码。
- 新增了 `TextDisableOldColorIndicators` 布尔型春季设置，默认为 false。如果为 true，则之前的 255 和 254 颜色代码将不再生效，`Engine.textColorCodes` 将列出新的代码。

### 未同步的爆炸事件

- 添加 `Script.SetWatchExplosion(weaponDefID, bool)` 及其对应的 Get 方法，用于使它们未同步。
- 现在小部件可以接收 `Explosion` 调用，参数与小工具的相同，但不能使用返回值来阻止 CEG。由于一些复杂的可见性规则（包括爆炸 CEG 的可见性），最终导致它提供的信息比原先承担类似功能的 `widget:ShockFront` 调用多了一些。

### 队伍单位限制

- 添加 `Spring.TransferTeamMaxUnits(fromTeam, toTeam, amount) -> bool success`，用于跨团队转移最大单位限制。
- 在 `Spring.TransferUnit(unitID, teamID, bool? given, bool? transferLimit)` 中添加第四个可选布尔参数，默认为 false。如果为 true，则同时将双方队伍的单位限制各增加 1，即使双方都已达到上限，也能完成转移。
- 添加 `FeatureSupport.deadTeamsKeepUnitLimit`，布尔值为 false。未来某个时间点，引擎将停止将已死亡队伍的单位限制重新分配给其盟友。

### 死亡金属

- 添加 `Spring.CreateUnitWreck(unitID, int? wreckLevel = 1, bool? emitSmoke = true) → featureID?`。创建一个单位死亡后的残骸效果。
- 添加 `Spring.CreateFeatureWreck(featureID, int? wreckLevel = 1, bool? emitSmoke = false) → featureID?`。创建一个单位死亡后的残骸效果。
- 添加 `Spring.AddFeatureDamage`，其签名与 `Spring.AddUnitDamage` 相同。注意，其中一个参数是瘫痪时间，但特征不会受到瘫痪伤害。
- 向 `Spring.SetFeatureHealth(featureID, health, bool? checkDeath = false)` 添加一个新的布尔参数 `checkDeath`。发现当前行为是：当特征的健康值设置为负数时，不会因生命值减少而死亡，必须通过一次伤害实例才能使其进入死亡状态。默认值为 `false`，保持此行为；将其设为 `true` 可让特征在生命值减少后立即死亡。
- 添加 `Spring.SetFeatureFireTime(featureID, fireTime)`，将特征设置为“着火”状态（类似于使用带有 `firestarter` 标签的武器造成伤害），并设定持续存活的时间（以秒为单位）。设置为 0 即取消该效果。
- 添加 `Spring.SetFeatureSmokeTime(featureID, smokeTime)`，使该特征持续释放烟雾（类似于刚被击杀的残骸）若干秒。请注意，烟雾大小与剩余时间成正比。  
- 为上述内容添加相应的获取器（`Spring.GetFeatureFireTime`、`Spring.GetFeatureSmokeTime`）。

### 游戏前阶段控制

- 添加 `system.useStartPositionSelecter` 为布尔值的模组规则。如果为 false，引擎在放置阶段将不会处理开始框上的点击事件。
- 添加同步函数 `Spring.SetTeamStartPosition(teamID, x, y, z) -> bool ok`。
- 添加同步函数 `Spring.SetPlayerReadyState(playerID, bool ready) -> bool ok`。
- 添加非同步函数 `Spring.RequestStartPosition(x, y, z, bool? ready) -> nil`，用于向指定位置发送起始请求，相当于原生选择器功能。

### 资源流动
- 添加 modrule 'system.nativeExcessSharing'，用于控制资源共享等级（即“红色滑块”）是否生效。请注意，这也会阻止多余的资源回流到盟友团队。如果你想让多余资源流向盟友，并仅阻止更改共享等级，请使用现有的 `gadget:AllowResourceLevel`。
- 添加 `Game.nativeExcessResource` 以读取上述设置。
- `Spring.ShareTeamResource` 不再触发 `gadget:AllowResourceTransfer`，如有需要请手动调用。

### 单位脚本片段缩放

为单位脚本添加内部缩放函数辅助功能，其语义与现有的Move和Wait API类似。  
请注意，这是一个单一的标量值（无法在一个维度上进行缩放）。

- `Spring.UnitScript.IsInScale(piece) -> boolean`
- `Spring.UnitScript.MultiScale(piece, size, speed, ...) -> boolean`
- `Spring.UnitScript.WaitForScale(piece) -> boolean`，完成后调用 `ScaleFinished(piece)`
- `Spring.UnitScript.GetPieceScale(piece) -> number`
- basecontent LUS 小工具支持上述功能（包括在适当情况下通过包装器实现，与 Move/Turn 相同）。

### 武器行为

- `BeamLaser` 和 `LightningCannon` 类型的武器现在会将实际的 projectileID 传递给调用函数，而不是使用 `-1`。
- `BeamLaser` 和 `LightningCannon` 现在能正确遵循椭球体和/或圆柱体目标体积。
- `BeamLaser` 和 `LightningCannon` 现在拥有正确的有效射程，并且 `targetBorder` 标签的默认值已更正。
- 与近乎垂直的悬崖相撞的弹丸现在会在碰撞位置生成爆炸，而非在悬崖顶部。此效果适用于除 DGun 之外的所有武器类型。

### 储备
- 现在将武器定义中的 `stockpileTime` 设置为 0 将会阻止进度。请使用此设置进行 Lua 重实现。
- 现在不允许将武器定义中的 `stockpileTime` 设置为负值。

### 守卫行为的模组规则

添加了大量模组规则，可配置使用“守卫”指令的单位相对于其守护者如何移动。

- `guard.guardRecalculateThreshold`：守卫在重新计算守护目标前必须移动的距离。  
- `guard.guardStoppedProximityGoal`：当守卫接近已停止的守卫时，会在此距离处停止。  
- `guard.guardMovingProximityGoal`：守卫与已停止的守卫之间额外保持的距离。  
- `guard.guardStoppedExtraDistance`：守卫被视作处于守护范围内并会匹配速度的距离。  
- `guard.guardMovingIntervalMultiplier`：守卫在移动时守护目标的倍率，数值越小，动作细节越高，但性能消耗越大。  
- `guard.guardInterceptionLimit`：当守卫不在守护范围内时，拦截行为的限制值。

### 贴纸
- 添加了 `$decals` 纹理标识符，以便在 Lua 纹理调用中访问贴花图集。  
- `/reloadshaders` 现在可以重新加载贴花着色器。  
- 地面贴花现在有两个新的四边形浮点属性，可用于任意自定义用途：索引 10 的 `in vec4 userDefined1` 和索引 11 的 `in vec4 userDefined2`。这些分别对应从 Lua 设置的第 0 和第 1 个四边形的用户数据值。  
- 新增 `Spring.SetGroundDecalUserData(decalID, number quad, number? u1, number? u2, number? u3, number? u4) -> bool ok` 函数。目前四边形编号必须为 0 或 1。  
- 新增 `Spring.GetGroundDecalUserData(decalID, number quad) -> number u1, u2, u3, u4` 函数。  
- 引擎不再直接加载 `shaders/GLSL/GroundDecalsFragProg.glsl` 和 `shaders/GLSL/GroundDecalsVertProg.glsl`。  
- 引擎现在会加载 `shaders/GLSL/groundDecals.lua`，该文件应返回一个 `{ vertex = "path", fragment = "path" }` 表。  
- basecontent 现在包含一个默认的 `groundDecals.lua` 实现，用于加载旧的着色器。  
- 新增 `Spring.SetGroundDecalGlowParams(decalID, number? glow, number? glowFalloff) -> bool ok` 函数。  
- 新增 `Spring.GetGroundDecalGlowParams(decalID) -> number glow, number glowFalloff` 函数。
- 在 `Spring.GetGroundDecalTextures(bool? mainTex, bool? alsoFilenames = false) → string[] textures, string[]? filenames` 中添加第二个布尔参数和可选的第二个返回值。

### 杂项
- `widget:Update(dt)` 现在接收 `dt`。
- 添加 `Spring.GetUnitMoveDefID(unitID) → number|bool moveDefID, string? name`。当单位有效但没有移动定义（飞机或建筑）时，返回 `false, nil`；否则返回数值 ID 和名称。
- 添加 `Spring.GetPieceProjectileName(pieceProjectileID) -> string name`。返回某个部件投射物的来源部件名称（如“body”、“turret”等）。非部件投射物返回 nil。
- 在 `Spring.UnitAttach` 中添加可选的第四个布尔参数，将其设为 true 时，将忽略运输规则并强制执行附加操作。
- 添加 `Spring.GetAllProjectiles(bool excludeWeaponProjs = false, bool excludeWeaponProjs = false) -> { proID, proID, ... }`。
- 添加 `Spring.SetFeaturePieceMatrix(featureID, m11, m12, ..., m44)`，类似于现有的单位矩阵设置函数。
- 移除了 10 个修改器的限制，现在可以拥有任意数量的修改器。
- Lua 着色器文件（即引擎直接加载用于着色器目的的任何 `foo.lua` 文件）现在可以访问未同步的 `VFS` 函数。
- 添加数值型单位武器标签 `accurateLeading`（注意：不是 weaponDef）。用于控制计算射击时额外进行的精度迭代次数。0：当前行为（单次迭代，无法处理速度差异大和/或角度异常的情况）。  
1：额外一次迭代，足以获得非弹道射击的完美解，通常对非极端弹道射击也足够。  
2+：弹道射击时可增加额外迭代次数，但实际使用中通常无需超过个位数的低数值即可满足需求。  

请注意，当达到1帧分辨率精度时计算将自动停止（因为单位和炮弹移动均为帧离散），因此将此值设置为任意高数值仍应安全且性能合理。  

- LuaParser 环境中现已支持数学函数扩展（如 `math.hypot`、`math.normalize`、位运算等）。  
- 新增 `Spring.GetFactoryCommandCount(unitID) → number` 检查工厂建造队列中的队列长度（注意：`GetUnitCommandCount` 用于工厂的集结队列）。  
- 新增 `ThreadPinPolicy` Spring设置，用于控制使用哪些CPU。（0）= 关闭；（1）= 系统默认；（2）= 排他性性能核心；（3）= 共享性能核心。  
- 新增CEG影响滤镜：`shield` 和 `intercepted`。- 对于可以“空袭”（即地雷）的不可移动单位，不再忽略“阻挡”单位的定义标签。
- 重新计算了被抛向空中地面单位的空气阻力。
- 添加布尔型Spring设置 `MiniMapDrawPings`，默认为 true。表示当放置标签时，引擎是否在缩略图上渲染脉动的白色方块。
- 如果工厂将建造顺序“更改”为相同类型的建造顺序（例如通过命令插入），则不再重置建造进度。
- `/group add N` 不再选择整个组，仅添加。`/groupN`（无空格）不受影响。功能支持标签：`Engine.FeatureSupport.groupAddDoesntSelect`。
- 将 rmlUI 数据模型背后的表暴露为 `__GetTable()`。
- 添加 `gl.GetEngineModelUniformDataSize(number index) → number sizeInElements, number sizeInBytesOnCPU`，用于获取 CPU 端模型统一变量缓冲区的大小。
- 添加 `LuaVBO:CopyTo(otherVBO, sizeInBytes) → bool ok`，用于将 GPU 端内容从当前 LuaVBO 复制到另一个。
- 在 `mouse10` 中添加 `mouse2` 作为可绑定的鼠标按钮键，同样适用于 `sc_mouse2-10` 扫描码。
- LMB（`mouse1` / `sc_mouse1`）计划稍后实现可绑定功能。- rmlUI 纹理使用最近邻像素过滤，而非线性过滤。预期效果是抗锯齿后的文字更加清晰。
- 添加了 GL 段式操作 Lua 常量（`KEEP`、`INCR`、`DECR`、`INCR_WRAP`、`DECR_WRAP`，除已有的 `ZERO`、`INVERT`、`REPLACE` 外）。
- 新增了仅适用于 Windows 的数值Spring设置 `DWMFlush`：在每次 SDL_GL_SwapWindow 之前强制执行 Windows 桌面合成器的 DWMFlush，防止画面卡顿（可使用 nVidia FrameView 验证丢帧情况，或使用 BARs Jitter Timer 小部件）。
值为 1 表示在 SwapBuffers 之前执行 DWMFlush，值为 2 表示在 SwapBuffers 之后执行 DWMFlush。
- 启用开发模式（通过启动脚本或运行时启用）会向同步的 Lua 添加 `debug.*` 函数。
- 即使禁用作弊功能，`/smoothmesh` 渲染器仍保持启用状态。
- 通过 Lua 尝试设置已弃用或不存在的Spring设置，现在会发出警告。
- `/give` 现在能正确提示 @x,y,z 的使用方法。
- `Spring.GetTeamList(allyTeamID?)` 若接收 2 个以上参数，不再崩溃（但仍会忽略这些参数，无法获取多个友方队伍的合并团队列表）。
- 新增了 Lua 常量 `GL.TEXTURE_2D_ARRAY`。
- `Spring.SetProjectileTarget` 现在对无效参数会抛出错误。
- 武器定义标签 `smokePeriod` 的默认值：8 → 1。

## 修复

- 修复“拔出蓝牙耳机时无声音”问题。  
- 修复多个 QTPFS 异步问题。  
- 修复 GLTF 模型中 tex1 和 tex2 纹理覆盖问题，确保其不会从 `*.gltf.lua` 元文件中读取。  
- 可能修复了使用错误解析器处理模型资源的罕见 bug。  
- 修复在 Linux 上检测到离线 CPU 时 CPU 数量错误的问题。  
- 修复当单位数量超过 2048 个时出现的崩溃问题。  
- 修复某些单位本应完全位于仅允许出口区域，但退出后出现混乱的问题。  
- 可能修复了与图标渲染相关的崩溃问题。  
- 修复 `Spring.GetMapStartPositions` 返回的团队 ID 偏移为 1 的问题。  
- 修复飞机有时弹起地面却未死亡导致的崩溃问题。  
- 禁用作弊功能不再禁用调试空气网格视图。  
- 修复重新加载包含脚本的 head 部分文档时 RmlUi 出现的崩溃问题。  
- 修复部分图形界面在无头构建中产生无害警告/错误的问题。  
- 修复部分 RmlUI 崩溃问题。  
- 修复部分字体性能问题。  
- 修复部分路径规划/QTPFS 问题。  
- 修复 Windows 系统下在大多数空闲和/或低速多人游戏中出现的抖动问题。  
- 修复游戏开始前模型在世界原点渲染时的一些问题。
