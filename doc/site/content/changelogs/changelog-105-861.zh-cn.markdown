+++
title = "发布版 105-861"
author = "sprunk"
translator = "Cru"
+++

自分叉以来至2022年2月**发布的105-861版本**的更新日志。

## 注意事项
* `wupget:GameProgress` 间隔从10秒调整为5秒  
* basecontent 现在提供从 `treetype0` 到 `treetype15` 的功能，包含模型和一些自定义参数  
* 建筑可以放置在其他建筑的开放庭院地图空隙中  
* 演示文件使用UTC时间戳文件名保存（可能导致文件名解析等问题）

## 功能与修复

### 图标作为UI
* 新增Spring设置 `UnitIconsAsUI`，以及游戏内命令 `/iconsAsUI`（以下所有选项均包含不带“Unit”的 `/command` 变体）。将值设为 1，可使雷达图标的行为类似于 UI 元素，而非世界中的对象（例如不会被地形遮挡、距离雾、模糊或水面反射等）。
* 新增Spring设置 `UnitIconScaleUI`，用于在作为 UI 绘制时对图标的尺寸进行缩放。
* 新增Spring设置 `UnitIconsHideWithUI`，用于控制是否通过 F5 隐藏图标。
* 新增Spring设置 `UnitIconFadeStart` 和 `UnitIconFadeVanish`，用于实现图标平滑淡入和淡出效果。

### 草
* `Spring.{Add,Get,Remove}Grass` 现在即使草的渲染被禁用也能正常工作（同时修复了不同步问题）  
* `Spring.AddGrass` 现在新增了第三个参数，用于设置草的强度（由 `Get` 返回；传入 0 等价于使用 `Remove`）

### 递归
允许对以下callout/callins进行浅递归（最多16次深调用）：
* `Spring.{Create,Destroy}{Unit,Feature}`
* `Spring.GiveOrder{,Array}ToUnit{,Array,Map}`

此外：
* `Spring.SpawnExplosion` 在 `Projectile{Created,Destroyed}` 内部有效

### 纹理图集
* 添加 `gl.CreateTextureAtlas(number x, number y, number? allocType, string? atlasName) → string texName`。  
`x` 和 `y` 必须在 256 到Spring设置中 `MaxTextureAtlasSizeX`（Y）的值之间。  
如果未指定，`allocType` 默认为 0。`atlasName` 是可选的图集名称。  
* 添加 `gl.FinalizeTextureAtlas(string id) → bool success`。  
* 添加 `gl.DeleteTextureAtlas(string id) → bool success`。  
* 添加 `gl.AddAtlasTexture(string id, string luaTex, string? subAtlasTexName)`。将纹理 `luaTexStr` 添加到图集 `id` 的子图集中 `subAtlasTextureName`（若未指定则与 `luaTex` 同名）。Lua 纹理必须是常规的 2D RGBA/UNORM 纹理。DDS 纹理也可添加到图集中。  
* 添加 `gl.GetAtlasTexture(string texName, string luaTex) → number s, p, t, q`。查询图集纹理中先前使用 `Spring.AddAtlasTexture` 保存的纹理的 UV 坐标（minU, maxU, minV, maxV）。

### 粒子
* 友军弹药的CEG现在在雾中可见  
* 添加 `/softParticles` 选项和 `SoftParticles` Spring设置，使粒子（尤其是地面闪光）在与地面相交时会略微淡出，而非出现锯齿状边缘  
* 添加 `Spring.SetNanoProjectileParams(r, v, a, randR, randV, randA) → nil`。所有参数均为数值，若未提供则默认为0。前三个参数分别为初始旋转角度（°）、旋转速度（°/s）和旋转加速度（°/s²），其余三个参数相同，但先乘以一个随机数（-1到+1），再加到前三个参数上，并在每个粒子生命周期开始时进行一次滚动  
* 添加 `Spring.GetNanoProjectileParams() → r, v, a, randR, randV, randA`，返回上述参数  
* 添加 `/drawOrderParticles` 命令，使粒子遵循CEG定义中 `drawOrder` 键所指定的绘制顺序  
* 在 `CSimpleParticleSystem` 和 `CBitmapMuzzleFlame` CEG 中添加 `rotParams` 键，用于存储3个浮点数：旋转速度（°/s）、旋转加速度（°/s²）和初始旋转角度（°）

### 高级GL4（着色器、VAO等）
* 添加 `gl.GetVAO`，该函数返回一个包含多种方法的对象。  
* 添加 `gl.GetVBO`。  
* 在着色器容器中添加 `Spring.GetSelectedUnitsCount`（使用场景：F2路径图替换，当未选择任何单位时可显示通用坡度图）。  
* 暂时添加 `gl.{Unit,Feature}{,Shape}GL4` 函数。  
* 为着色器添加大量统一变量。  
* 添加 `LuaShaders::Set{Unit,Feature}BufferUniforms`，用于GPU端的单位/特征“统一变量”（实际上是SSBO）。  
* `gl.Uniform/gl.UniformInt` 等函数不再需要位置作为第一个参数，名称也可以直接使用。

### 渲染用的Lua常量
* 添加 `Platform.glHave{AMD,NVidia,Intel,GLSL,GL4}` 布尔值  
* 移除 `Platform.glSupport24bitDepthBuffer`，并改为添加 `Platform.glSupportDepthBufferBitDepth`  
* 向 GL 添加以下常量：`LINE_STRIP_ADJACENCY`、`LINES_ADJACENCY`、`TRIANGLE_STRIP_ADJACENCY`、`TRIANGLES_ADJACENCY`、`PATCHES`  
* 向 GL 添加以下常量：`[UNSIGNED_]BYTE`、`[UNSIGNED_]SHORT`、`[UNSIGNED_]INT[_VEC4]`、`FLOAT[_VEC4]`、`FLOAT_MAT4`  
* 向 GL 添加以下常量：`ELEMENT_ARRAY_BUFFER`、`ARRAY_BUFFER`、`UNIFORM_BUFFER`、`SHADER_STORAGE_BUFFER`  
* 向 GL 添加以下常量：`READ_ONLY`、`WRITE_ONLY`、`READ_WRITE`  
* 向 GL 添加大量图像类型规范（如 `RGBA32` 等）  
* 向 GL 添加大量屏障位常量

### 其他与渲染相关的事项
* 大多数CEG和武器视觉效果会测试纹理有效性，确保带有无效纹理的精灵不会显示。不再需要1x1的空纹理  
* 修复天空盒拉伸和接缝问题（如果天空盒由立方体贴图表示时）  
* 区域命令圆圈更加平滑，顶点从20个增加到100个  
* 添加 `Spring.{Set,Get}{Unit,Feature}AlwaysUpdateMatrix`，使单位“始终更新矩阵”  
* 添加 `/reloadTextures [lua, smf, s3o, ceg]` 命令，用于重新加载所有纹理组（若未指定则全部重载）  
* 填充纳米线框在当前模型高度下有效（即隐藏在上方但向下移动的部件的单位可正确工作）  
* 添加 `globalRenderingInfo.availableVideoModes`  
* 添加 `gl.GetFixedState`，返回一系列不同的GL常量和表，其功能取决于参数  
* 将 `DeprecatedGLWarnLevel` 的默认值从2改为0（以减少垃圾警告信息）  
* 添加 `gl.AlphaToCoverage(bool enable, bool? force) → nil`，仅当平台支持 `GLEW_ARB_multisample` 时才定义；否则该函数本身为nil；`force` 参数即使MSAA级别低于4也强制应用  
* 改进AMD显卡检测（查找更多字符串，例如“radeon”，并在更多位置进行检测）  
* 添加 `gl.ClipDistance(number clipID, bool enabled) → nil，`clipID` 为 1 或 2。  
* 将 `TextureMemPoolSize` 的默认值从 256 更改为 512 MB  
* 将 `windSpeed` 提供给 `Spring.{Get,Set}WaterParams` 使用  
* 新增水纹参数（默认值）：waveOffsetFactor (0.0)，waveLength (0.15)，waveFoamDistortion (0.05)，waveFoamIntensity (0.5)，causticsResolution (75.0)，causticsStrength (0.08)  
* 修复地面天空反射（`skyReflectModTex`）

### 导弹烟迹
* 武器定义：为带有烟雾尾迹的物品添加布尔值 `smokeTrailCastShadow`、数值 `smokePeriod`（默认 8）、数值 `smokeTime`（默认 60）、数值 `smokeSize`（默认 7）和数值 `smokeColor`（默认 0.7，三个RGB通道颜色均为单个值）  
* 武器定义：为弹道本身添加布尔值 `castShadow`

### 相机调整
* 添加整数Spring设置 `SmoothTimeOffset`。该设置用于平滑计算插值绘制帧所使用的 TimeOffset 参数。默认值为 0，即旧行为。建议值为 2，可使实际时间偏移保持在真实时间的 90% 以内。开启垂直同步时效果最佳。
* 添加布尔型弹簧设置 `CamFrameTimeCorrection`。默认值为 false，当前行为；使用 true 可在高负载情况下获得更好的插值效果。

### 建造
* 为“堆叠”建筑新增 yardmap 选项  
* 新增 `Spring.{Set,Get}UnitBuildParams(unitID, "buildDistance"/"buildRange3D", value) → nil`，用于设置单位建造范围（数值）以及是否为球形（布尔值）  
* 新增 `Spring.GetUnitInBuildStance(unitID) → bool` 函数调用  
* 修复：如果水上飞机被设置为浮出而非沉没，则无法在水面上建造的问题

### Lua API 的其他新增功能
* `math.random()` 现在在解析定义时能正确接受参数  
* 允许 `Spring.SetCameraState` 的第一个参数为 `nil`  
* `Spring.SetWMIcon` 新增第二个参数 `force`（布尔值）：忽略 Windows 上 32x32 图标限制  
* 允许 `Spring.GetKeyBindings` 为空参数，以返回所有按键绑定  
* 添加 `Spring.GetUnitsInScreenRectangle`  
* 新增 `Spring.SetWindowGeometry`，便于窗口定位  
* 新增 `Spring.ForceTesselationUpdate(bool normalMesh = true, bool shadowMesh = false) → bool isROAM`，用于 ROAM  
* 无参数的 `Spring.GetUnitFlanking()` 现在有第8个返回值，即收集的侧方优势机动性

### 其他功能
* 选择键：添加 `IdMatches` 过滤器以使用内部名称  
* 默认路径查找器（通过 modrules 中的 0 设置）现在支持多线程。在版本 861 中，旧的单线程路径查找器可通过将 modrule 设置为 2 临时启用，但该功能将在后续版本 1544 中移除，届时多线程问题已修复。  
* 新增 `/setSpeed x` 命令，可直接设置游戏速度，但仍受最小/最大值限制  
* 新增 `/endGraph 2` 命令，可将玩家返回菜单（1 仍退出游戏）  
* 键盘绑定：相同操作但不同参数现在可区分  
* 在原始 spring.exe 菜单中新增“加载游戏”按钮  
* 命令缓存从 1024 增加至 2048  
* 原生 AI 接口：新增 `Feature_getResurrectDef` 和 `Feature_getBuildingFacing`  
* 原生 AI 接口：在 `Get{Friendly,Enemy,Neutral,}{Units,Features}{In,}` 中新增 `spherical` 参数

### 杂项修复
* 修复盟友投射物的CEG（碰撞检测）问题：现在在雾中可见  
* 修复磁盘读取失败问题（出现读取错误的文件将多次重试后才放弃）  
* 修复启用全局地形锁定时未立即显示未看到的地形变化的问题  
* 修复高度范围计算，现在反映实际高度而非历史最小/最大值（例如，若最高点从100 → 90 → 110 → 100，则之前 `Spring.GetGroundExtremes` 返回的是 100 → 100 → 110 → 110）  
* 修复 `Spring.SpawnExplosion` 在某些有效数据下崩溃的问题  
* 修复飞机使用平滑网格更新时，若地形在游戏前被修改而无法正确处理的问题  
* 多处修复了保存与加载功能  
* 修复弹簧专用模式在Windows系统上未重定向时输出显示不正确的问题（spring-headless 已经正常工作）  
* 修复尝试向控制组添加敌方单位时崩溃的问题（例如通过 /godmode 选择）  
* 修复由本地化依赖字符串排序引起的同步问题  
* 修复在当前地图无LuaGaia时，尝试加载使用LuaGaia的地图保存的存档时崩溃的问题  
* 修复由于浮点数精度问题导致资源有时进入负值（约1e-15级别）的问题特别是，这导致原本无需消耗资源的单位因资源不足而无法发射  
* 修复“僵尸鱼雷”在地图下方弹跳的问题  
* 修复有时子弹穿透单位的问题  
* 修复“卸载已死亡单位”时同步崩溃的问题  
* 在 Linux Wayland 上可能有效（目前实现尚未支持硬件光标；后续版本中将添加）