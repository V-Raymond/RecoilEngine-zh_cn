+++
title = "发布版 2025.04"
aliases = ['/changelogs/changelog-2025-03']
+++

这是从2025.01版本到**2025.04.11版本**的更新日志，该版本于2025年8月16日发布。

# 注意事项
这些条目在迁移时可能需要特别注意：
* 网络协议：`NETMSG_MAPDRAW`（用于涂鸦相关操作：绘制、点、擦除），ID 31，已重命名为 `NETMSG_MAPDRAW_OLD`，不再使用。  
`NETMSG_MAPDRAW` 现在 ID 为 32，并且以 `uint32` 格式发送坐标，而非 `int16`。  
* 在顶点着色器中，对于从 uvec2（骨骼ID、骨骼权重）扩展到 uvec3（低字节骨骼ID、骨骼权重、高字节骨骼ID）的模型，统一位置编号 5 已修改。  
现有着色器通常仍可正常工作。  
* 导弹在过期后现在会遵循 `myGravity`。  
* `math.clamp` 现在如果下界大于上界将抛出错误。  
* 新增了迷你地图旋转 API，可能导致迷你地图显示异常（见下文），若不想处理该功能，请在 LuaUI 入口点将其设为 nil。  
* 新增了 `UnitGhostIconsDimming` 数值Spring设置，用于控制幽灵图标颜色的倍数（主视图和迷你地图均适用）。默认值为 0.8，图标将变暗；设置为 1.0 即恢复旧行为。  
* 服务器不再在 30 秒内无人连接时自动强制启动游戏。  
* 单位不再因目标在地图外超过 5 秒而丢弃护盾指令。如需替换，请查看 basecontent 中的 `control_guard.lua`。* 修复了 `SPRING_LOG_SECTIONS` 环境变量，不再需要在前面加上逗号。
* `gl.GetAtmosphere("skyDir")` 现在返回 nil。skyDir 实际上从未被用于任何用途。
* 更新 Tracy 到 v0.11.1 版本，详见 https://github.com/wolfpld/tracy/releases
* 重写了缓存归档机制。处理速度应会快很多，但会占用更多磁盘空间，目前尚不清楚其稳定性如何。
* 在主引擎的 GL 绘图调用中添加了 GL 调试注释，类似于常规代码中的 Tracy 范围。
需要注意的是，在 `/debugGL` 模式下，这可能会带来较大的性能开销。下一版本计划在此方面进行改进。
* `FontOutlineWidth` 春季设置的默认值已从 3 改为 2。这是由于宽度修复所致，因此不会影响外观。
* 为了安全性和可维护性，Lua 的 `loadstring` 默认不再接受字节码，但在开发模式下仍可正常工作。

### 弃用通知
* 在构建金属提取器时，金属视图自动切换的功能现已弃用。  
请通过 `Spring.SetAutoShowMetal`（见下文）将其禁用，并手动重新实现。已提供替代示例，位于 [`(basecontent)/examples/Widgets/gui_autoshowmetal.lua`](https://github.com/beyond-all-reason/RecoilEngine/blob/master/cont/examples/Widgets/gui_autoshowmetal.lua)。

# 功能

### Lua 语言服务器支持
LDoc 已被兼容 [Lua 语言服务器](https://luals.github.io/) 的注解所取代。这使得在编辑 Lua 代码时能够支持语言服务器功能（例如自动补全和类型检查）。  
类型定义可参考 [Lua 库仓库](https://github.com/beyond-all-reason/recoil-lua-library)，该库旨在作为使用此引擎的项目中的子模块包含。  
[Lua API 文档]({{< base_url >}}/docs/lua-api/) 现已从 LLS 定义中生成，而非以前的 LDoc。这导致文档质量出现退化，所有文档都集中在一个页面上，部分文档信息缺失。目前正考虑对文档进行改进。  
更多详情请参阅 [Lua 语言服务器指南]({{< base_url >}}/docs/guides/getting-started/lua-language-server/)。

### 单位组
* 单位在死亡动画开始时不再从组中移除。  
* 新增了 `Spring.SetUnitNoGroup(unitID, bool noGroup) → nil`，用于设置单位是否可以加入组。  
* 新增了 `Spring.GetUnitNoGroup(unitID) → bool noGroup`，用于获取单位是否可加入组。

### 与基础设施相关的
* 新增了 CLI 参数 `--calc-checksum "Archive Name"`，用于将单个存档的校验和写入存档缓存。可从大厅中使用以预加载内容，但需注意避免并行调用。
* 新增了 `--only-local` CLI 参数（适用于 spring.exe），可防止重放器打开连接。可用于批量重放解析。
* 修复了环境变量 `SPRING_LOG_SECTIONS`，不再需要在前面加上逗号。
* 服务器不再在30秒后无人连接时自动强制启动游戏。

### 更多模型部件
* 模型现在支持 65534 个部件，从之前的 255 个提升。
* 即使是百件部件的模型，其性能通常也很差，因此应将其视为一种辅助工具，并尽可能避免使用。
* 新增 `Engine.FeatureSupport.maxPiecesPerModel` 常量，可通过读取该值获取部件数量。
* 对于从 uvec2（骨骼ID、骨骼权重）扩展为 uvec3（低字节骨骼ID、骨骼权重、高字节骨骼ID）的模型，顶点着色器中的位置 5 应保持一致。
现有着色器通常仍可正常工作。

### 自动金属图视图
* 新增了 `Spring.SetAutoShowMetal(bool) → nil`。如果将其设置为 false，那么选择“build mex”命令时将不会自动启用金属视图（可使用 `(basecontent)/examples/Widgets/gui_autoshowmetal.lua` 中的组件来复现此行为）。如果保持为 true（默认值），引擎会继续自动启用金属视图，同时频繁显示弃用警告。该函数将在未来某个时间点被移除。
* 新增了 `Engine.FeatureSupport.noAutoShowMetal`，初始值为 false。未来某个时间点，该值将改为 true，届时引擎将不再自动为 mexes 启用金属视图（并移除 `Spring.SetAutoShowMetal`）。

### 90°/270°旋转的迷你地图
* 现在可以将迷你地图旋转至90°和270°，并可手动旋转。  
* 新增了 `Spring.SetMiniMapRotation(number angle) → nil` 方法，用于设置迷你地图角度（以弧度为单位），并会自动对齐四个方向。  
仅当Spring设置 `MiniMapCanFlip` 设置为 0 时才有效（若为 1，则与之前一样由引擎自动垂直翻转）。  
保持原有几何结构，可能导致拉伸，可通过 `gl.ConfigMiniMap` 手动重新调整。  
* `Spring.GetMiniMapRotation() → number angle` 现在支持新的旋转值（例如可返回 `π/2` 和 `3π/2`）。

### 杂项渲染
* 添加 `Platform.glVersionNum`，表示 GLSL 版本的 1/10（例如 GL 3.0 → 30）。
* 新增 `gl.ClearAttachmentFBO(number? target = GL.FRAMEBUFFER, string|number attachment, clearValue0, cv1, cv2, cv3) → bool ok` 接口。用于清除当前绑定的 FBO 类型 "target" 的指定 "attachment" 并使用 "clearValues" 清除内容。
Attachment 可以是字符串（如 "color0" 至 "color15"、"depth" 或 "stencil"），或对应的 `GL.COLOR_ATTACHMENT#` 常量。
* 启用超采样抗锯齿后，迷你地图显示更加清晰锐利。
* 在 basecontent 比特图存档中新增了 3D 噪声资源，位于 `bitmaps/noise/recoil_noise_2025_p5_p3_w6_w4_64x64x64_RGBA.dds` 和 `bitmaps/noise/recoil_noise_2025_p5_p3_w6_w4_128x128x128_RGBA.dds`。
这使得在着色器中可靠生成更便宜的 3D 噪声成为可能。
* 新增（立即废弃）的接口：`GL.POINT_SMOOTH` 和 `gl.GetFixedState("pointSmooth") → number pointSmooth`。
请勿使用这些接口，它们只是临时修补旧代码的“创可贴”，新代码应改用着色器实现。

### 赞美太阳吧！
* 阴影纹理（用于迷你地图、水面和草地）现在会立即追踪太阳位置的变化。  
* “现代”天空渲染器现在使用实际的太阳颜色，而不是硬编码的 RGB 253/251/211。

### 旋转天空盒
* 在 `Spring.SetAtmosphere` 中添加了新标签 `skyAxisAngle`，它是一个包含 4 个数字的数组：X、Y、Z（定义一个轴）以及绕该轴的旋转角度。  
现在可以设置天空盒以特定角度绘制，并可能实现旋转效果。复杂的旋转需要自行计算数学运算。  
* 在 `gl.GetAtmosphere` 中添加了 `skyAxisAngle`，格式相同。  
* 在 mapinfo 中添加了 `atmosphere.skyAxisAngle`，格式相同。  
* 从 mapinfo 中移除了 `atmosphere.skyDir`，因为它实际上没有产生任何影响。  
* 现在 `gl.GetAtmosphere("skyDir")` 返回 nil。

### 渲染调试
* 新增 `VBO:GetID() → number`，用于获取内部 OpenGL ID，以便调试使用。  
* 为 `gl.CreateShader` 添加了第二个返回值，现在也会返回内部 ID，供调试使用。  
* 新增 `gl.ObjectLabel(objType, objID, string identifier) → nil`。其中类型为新的 GL 对象类型常量之一（如 `GL.BUFFER`、`GL.PROGRAM_PIPELINE` 等，参见底部列表）。  
在对象上添加文本标签，用于调试工具。根据图形驱动程序等条件可选（需检查是否为 nil）。  
* 新增 `gl.PushDebugGroup(number ID, string message, bool isThirdParty) → nil`。条件性可用（取决于玩家平台，需检查是否为 nil）。在调试堆栈中创建一个带有指定消息的组（本质上类似于 tracy 的作用范围），专用于调试工具，似乎特别适用于“nVidia nSight 2024.04”。当 FBO 被原始绑定时，调试工具可能难以显示注释信息。  
* 新增 `gl.PopDebugGroup() → nil`，条件性可用，从堆栈中弹出先前推入的调试组。  
* 引擎现在原生地在相关作用域内推入一些 GL 调试组，类似于 tracy。需要注意的是，启用 GL 调试时，这可能会带来明显的性能开销。

### 单位方位
* `Spring.GetUnitDirection` 和 `Spring.GetFeatureDirection` 现在返回 9 个值（从 3 开始）。  
最初的三个值保持为单位空间中“前”方向的 XYZ，新的三个值分别是“右”方向和“上”方向的 XYZ。
```diff
- local frontX, frontY, frontZ = Spring.GetUnitDirection(unitID)
+ local frontX, frontY, frontZ, rightX, rightY, rightZ, upX, upY, upZ = Spring.GetUnitDirection(unitID)
```
* `Spring.SetUnitDirection` 和 `Spring.SetFeatureDirection` 现在最多可接受 6 个参数（之前为 3 个）。新的参数包括用于“右侧”方向的 XYZ（因为仅“前方”方向存在歧义），且这些参数为可选（行为与之前相同，即根据地面确定某些方向）。请注意，无法指定“向上”方向（但若已指定“右侧”方向，则方向明确；如果同时有“前方”和“向上”方向，则可以唯一确定“右侧”方向）。
```diff
  Spring.SetUnitDirection(unitID, frontX, frontY, frontZ) -- still works
+ Spring.SetUnitDirection(unitID, frontX, frontY, frontZ, rightX, rightY, rightZ) -- new
```

### 引擎功能支持标签
在 `Engine.FeatureSupport` 标签中添加了若干功能支持条目，这些是为未来预留的。  
由于游戏会根据实际变更发布补丁，因此您很可能无需为此担心。  
* 数字 `gunshipCruiseAltitudeMultiplier`，当前值为 `1.5`。目前直升机巡航高度将乘以该数值。  
* 布尔类型 `noRefundForConstructionDecay`，当前值为 `false`。未来将向 Lua 传递建筑衰减的退款。  
* 布尔类型 `noRefundForFactoryCancel`，当前值为 `false`。未来将向 Lua 传递工厂取消的退款。  
* 布尔类型 `noOffsetForFeatureID`，当前值为 `false`。未来在混合上下文中（例如回收目标ID）的功能ID将不再需要进行偏移。

### 字体渲染性能
* 新增布尔型Spring设置 `UseFontConfigSystemFonts`，默认为 true。用于是否在系统字体中搜索备用字体。
* 新增布尔型Spring设置 `FontConfigSearchAttributes`，默认为 true。用于是否尝试匹配提供的字体属性作为备用选项。
虽然会牺牲一些性能，但可能获得更好的字形匹配效果。
* 新增布尔型Spring设置 `FontConfigApplySubstitutions`，默认为 true。用于是否应用模式配置的替换。
禁用替换可能会导致字体随机失效，但通过这种方式可以节省部分性能开销。
* 新增整型Spring设置 `MaxPinnedFonts`，默认值为 10。缓存中保留多少个字体。
如果使用大量字体，可减少性能突增（需注意玩家可能发送包含多种异域字符的混合聊天内容），但若加载大量字体，可能导致峰值内存使用量增加。
* 即使不修改上述任何设置，也对基础字体性能进行了额外优化。
* Lua 中 `font:Begin(bool? userDefinedBlending)` 现在支持可选的布尔参数，允许使用通过 `gl.BlendFunc` 设置的任意混合模式。
默认情况下，若设置为 `false`，则保持当前行为，始终使用 `GL.SRC_ALPHA, GL.ONE_MINUS_SRC_ALPHA`。* 修复了字体轮廓宽度。此前，2N+1 的值与 2N 看起来相同。  
因此，为保持外观一致，`FontOutlineWidth` 的默认值从 3 改为 2。

### rmlUI
* 为 rmlUI 数据模型添加深度响应能力。  
* 为 rmlUI 添加 SVG 支持。  
* 修复 rmlUI 在抗锯齿时出现模糊的问题。

### 建筑幽灵
* 新增了 `UnitGhostIconsDimming` 数值Spring设置，用于控制幽灵图标颜色的倍数（主视图和缩略图中均适用）。默认值为 0.8，使图标变暗；设为 1.0 即可恢复之前的显示效果。  
* 修复了缩略图中建筑幽灵是否已死亡的显示问题。

### 杂项
* `SendToUnsynced` 现在可以发送（而 `gadget:RecvFromSynced` 可以接收）表格。Unsynced 会接收到一个副本，该副本会递归地移除所有非支持类型且没有元表的元素（与 `SYNCED.` 代理相同）。  
* 添加 `Spring.GetSoundDevices() → { { name = "...", }, { name = "...", }, ... }`。
未来可能会提供更多详细信息，而不仅仅是名称。
* 添加 `wupget:ActiveCommandChanged(cmdID?, cmdType?) → nil`。
* `Spring.GiveOrder` 系列函数现在可接受 `nil` 作为参数（与 `{}` 相同）以及选项（与 `0` 相同）。
* 添加 `Spring.SetUnitStorage(unitID, "m"|"e", value) → nil`。
* 添加 `Spring.GetUnitStorage(unitID) → 金属数量，能量数量`。
* 添加 `Spring.GetProjectilesInSphere(x, y, z, radius, bool excludeWeaponProjectiles = false, bool excludePieceProjectiles = false) → {projID, projID, ...}`。
* 在 `Spring.Set/GetUnitWeaponState` 中添加 "ttl" 字段，表示炮弹的存活时间（秒）。
* 新增 `Game.buildGridResolution`，当前数值为 2。这意味着通过原生建造指令创建的建筑将对齐到 2 个方格。
* 添加 `Spring.GetMouseButtonsPressed(indexA, indexB, ..., indexN) → bool pressedA, pressedB, ..., pressedN`。
例如，`Get(4, 1)` 将返回鼠标 4 和左键（鼠标 1）的状态。必须至少提供一个索引。
* 添加 `Platform.totalRAM`，单位为兆字节。
* 添加 `Spring.SetProjectileTimeToLive(projID, framesTLL) → nil`。
* 添加 `Spring.GetUnitSeismicSignature(unitID) → number`。Set 已经在之前添加过。
* 新增 `Engine.gameSpeed` 和 `Engine.textColorCodes`，与 Game 中现有的条目相同。  
实际效果是：在某些 LuaParser 环境中，Engine 表可被使用，而 Game 无法使用。  
* 导弹在过期后现在会遵循 `myGravity`。  
* 来自 Lua 的 NaN 和无穷大现在有时会被拒绝。覆盖范围尚未全面。  
* 单位不再在目标地图外停留超过 5 秒时掉落 Guard 命令。  
* socket.lua 从原先的松散分布文件（位于 ./socket.lua）移至 basecontent 目录下的 ./LuaSocket/socket.lua。  
* 添加 `experience.experienceGrade` 数值模组规则，其功能与调用 Spring.SetExperienceGrade 相同。  
* 模组规则 `allowHoverUnitStrafing` 现在默认为 `false`。此前 HAPFS 默认为 `false`，QTPFS 默认为 `true`。  
* 滑动贴图水体（即 `/water 4`）现在拥有不同的默认纹理。  
* 向异步数据快照中添加更多数据。额外数据将被保存到名为 `ClientGameState-<random number>-[<desync frame>-<desync frame>].txt` 的文件中，格式与之前相同。  
* 由于安全性和可维护性考虑，Lua 的 `loadstring` 不再接受字节码。

### 修复
* 修复了 `Spring.SetUnitHealth(build < 1)` 不会将单位回退到纳米帧的问题。  
* 修复了 rmlUI 在抗锯齿下出现额外模糊的问题。  
* 修复了迷你地图图标无法正确显示建筑幽灵是否已死亡的问题。  
* 修复了 CPU 钉住问题，不再尝试将自身钉在不合适的选项上（如效率核心、同一物理核心上的超线程、专用服务器上的性能核心）。  
* 修复了因显式传入 nil 而导致的 `Spring.ShareResources(teamID, "units", nil)` 函数崩溃问题。  
* 修复了在大于 64xN 的地图上，脚本注释和标签显示异常的问题。  
* 修复了 basecontent `actions.lua` 提供了错误的 `KeyAction` 处理器问题。  
* 修复了正在建设中的建筑高度未能正确更新的问题。  
* 修复了飞机着陆后在受到电磁脉冲（EMP）影响时开始漂浮的问题。  
* 修复了当重叠的抗推单位停止时，其他单位卡住的问题。  
* 修复了“现代”天空渲染器未根据 `Spring.SetAtmosphere` 的更改进行调整的问题。  
* 修复了从工厂队列中移除数千个单位时出现的冻结问题（或至少大幅减少此类问题）。  
* 修复了加载 7z 压缩包时，若文件未设置 'mod time' 字段而引发的崩溃问题。  
* 修复了加载某些 WAV 音效时出现的崩溃问题。  
* 修复了一些与字体相关的崩溃问题。* 修复了在以旁观者身份调用 `Spring.GiveOrder` 时出现的轻微堆栈损坏问题。  
* 修复了在 Linux 系统上使用 Wayland 时出现的一些图形崩溃问题。  
* 修复了 `Spring.GetUnitCurrentCommand` 实际上不接受负索引的问题。  
* 修复了因未同步的 Lua 路径请求导致的同步问题。  
* 修复了当高数量单位获得移动指令时出现的性能突增问题。

### GL对象类型常量
用于新的 `gl.ObjectLabel`（见上文）：
* `GL.BUFFER`
* `GL.SHADER`
* `GL.PROGRAM`
* `GL.VERTEX_ARRAY`
* `GL.QUERY`
* `GL.PROGRAM_PIPELINE`
* `GL.TRANSFORM_FEEDBACK`
* `GL.RENDERBUFFER`
* `GL.FRAMEBUFFER`
