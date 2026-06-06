+++
title = "发布版 105-2511"
+++

这是从版本2314到2024年6月7日**发布的2511版本**之间的更新日志。

# 注意事项
这些条目在迁移时可能需要特别注意：
* 部分动画现在采用多线程处理，不应导致不同步问题，但已提供 `AnimationMT` Spring设置以备不时之需。详情见下文。
* 通过 CMake 构建引擎时，默认仅构建原生 C++ AI。
* 单位定义的 `trackStretch` 值现在与之前相反，即拉伸系数为 2 表示轨道被拉长了两倍（此前为缩短了 0.5 倍）。
* 地面贴花可能因新实现而行为略有不同。目前存在若干已知问题，请参见上方贴图部分。
* 地面贴花可能不再适用于低端硬件。
* 基础内容中默认的爆炸贴图从 4 个（`bitmaps/scars/scarN.bmp`，1-4）变为 2 个新常量映射贴图（`bitmaps/scars/scarN.tga` 和 `bitmaps/scars/scarN_normal.tga`，1-2）。
请注意格式已从 BMP 改为 TGA。
建议检查您的 `gamedata/resources.lua`，确认是否仍引用旧的默认伤疤（无论是显式引用，还是例如从 `bitmaps/scars` 文件夹自动生成）。
您也可以考虑生成更多贴图，以弥补种类减少的问题。* 对已指定单位的默认目标优先级不再包含±30%的随机成分。使用 `gadget:AllowWeaponTarget` 可恢复之前的设定行为。
* 引擎的行进式阵型现在采用单位类型能力来分配部队，而非成本函数（60M+E）。能力默认为60M+E，所有已知游戏均使用Lua自定义阵型。
* 现在可以设置爆破武器遵守射击角度限制。参见下方“爆破”部分。

# 功能
* `select` 操作现在将 `IdMatches` 过滤器组合为 *OR* 语句，详情请参见 [select 命令]({{< base_url >}}/docs/guides/select-command/#idmatches_string)。
* 在 `Spring.GetUnitHeading` 中新增了一个可选的布尔参数，默认值为 false。如果设置为 true，则返回的值以弧度表示，而非 TA 的 16 位角度单位。
* 新增了调用函数 `GameFramePost(number frame)`，这是模拟帧中的最后一个调用（常规的 `GameFrame` 是第一个）。
* 新增了调用函数 `UnitArrivedAtGoal(unitID, unitDefID, teamID)`，用于当单位到达其移动目标时触发（包括原始移动）。
可用于批量处理在当前帧内发生的事件，并在下一个模拟帧之前发送到非同步帧中用于绘制。
* 新增了 `Spring.GetTeamMaxUnits(teamID) -> number maxUnits, number? currentUnits`。第二个值仅在你对该团队有读取权限时返回（最大值是公开的）。
目前尚无对应的设置函数。
* 新增了 `GAME/ShowServerName` 开始脚本条目。如果该选项不为空，则初始连接界面的“正在连接至：X”消息将显示该选项的值，而不是主机的 IP 地址。
* 新增了 `Spring.GetModOption(string key) -> string? value`。返回单个模组选项；替代了 `Spring.GetModOptions().foo` 的模式，以提高性能。  
* 新增了 `Spring.GetMapOption(string key) -> string? value` 方法，用于获取单个映射选项。  
* 添加了 `Patrolling` 选择过滤器，适用于其前四个命令中包含巡逻指令的单位（注意：巡逻指令会前置战斗指令，而战斗指令又会前置攻击指令）。  
* `SpringDataRoot` 模块设置现在可接受多个数据根路径，通过分号（Windows 系统）或冒号（其他系统）进行分隔。  
* `Spring.GetUnitWorkerTask` 现在也能处理工厂单位。  
* 大幅度提升性能（包括加载时间和 Lua 内存使用情况）。  
* 增加了更多的 Tracy 监测工具。

### 全新地面贴纸
* 为所有贴图（爆炸伤痕、建筑板、坦克履带）添加了法线映射功能。  
将法线贴图命名为与基础贴图相同，但末尾加上 `_normal`，例如当使用 `scar1.tga` 作为贴图的漫反射/透明度时，应使用 `scar1_normal.tga` 作为法线贴图纹理。  
* 法线贴图的Alpha通道现在用于爆炸贴图的发光效果，其亮度随武器伤害值变化。完全不透明（255）表示最亮的部分。  
* 新增了Lua接口，用于创建和编辑贴图。请参阅API文档中的[控制](https://recoilengine.org/ldoc/modules/UnsyncedCtrl.html#Decals) 和[读取](https://recoilengine.org/ldoc/modules/UnsyncedRead.html#Decals)部分。  
* 新增了贴图渲染的着色器接口。目前似乎没有关于uniforms/attributes等的文档，但你可以查看默认的着色器实现（[片段](https://github.com/beyond-all-reason/RecoilEngine/blob/BAR105/cont/base/springcontent/shaders/GLSL/GroundDecalsFragProg.glsl)，[顶点](https://github.com/beyond-all-reason/RecoilEngine/blob/BAR105/cont/base/springcontent/shaders/GLSL/GroundDecalsVertProg.glsl)）。  
* 替换了basecontent中的伤痕位图。详见上方注意事项部分。
* `trackStretch` 单元的值现在与之前相反，即拉伸系数为 2 现在表示轨道被拉长了两倍（此前为缩短了 0.5 倍）。
* 地面贴纸可能不再适用于土豆硬件。
* 已知问题：轨道/足迹在闪烁步长处不再保持纹理偏移。
* 已知问题：建筑贴纸在水下可能无法正确渲染。

### 爆发和发射角度
* 新增了一项武器属性（非武器定义中的武器表，而是单位定义中的数值标签）`burstControlWhenOutOfArc`。将其设置为1或2时，持续的连续射击（从第二次射击开始）现在可以遵循来自`mainDir` + `maxAngleDif`（或等效于`turret = false` + `tolerance`）的发射锥，以及与目标方向相比来自`fireTolerance`的另一锥形区域。
* 若设为0，则后续射击将不遵守发射锥。这是默认行为，即之前的处理方式。
* 若设为1，该武器将无法向锥形范围之外的目标发射，导致这些射击被浪费。重新进入锥形内瞄准后，将继续发射弹药（但不会“返还”已浪费的射击，整体发射周期仍保持不变）。即使最后一次射击被浪费，`EndBurst`脚本事件仍会执行，但每次射击的事件不会对被浪费的射击触发。
* 若设为2，该武器将直接忽略尝试瞄准锥形外的目标，并继续以目标来源部件指向的方向进行射击。

### `defs.lua` 中新增更多接口
以下函数现已在 `defs.lua` 阶段可用：
* `Spring.GetMapOption`（新增，见上文）
* `Spring.GetModOption`（新增，见上文）
* `Spring.GetTeamLuaAI`
* `Spring.GetTeamList`
* `Spring.GetGaiaTeamID`
* `Spring.GetPlayerList`
* `Spring.GetAllyTeamList`
* `Spring.GetTeamInfo`
* `Spring.GetAllyTeamInfo`
* `Spring.GetAIInfo`
* `Spring.GetTeamAllyTeamID`
* `Spring.AreTeamsAllied`
* `Spring.ArePlayersAllied`
* `Spring.GetSideData`

### 水
* 添加 `Spring.GetWaterLevel(x, z) -> number waterHeight`。类似于 `Spring.GetGroundHeight`，但返回该位置的水位高度。
目前所有位置的水位均为 0。在将来 Recoil 支持动态水时可作为预留方案使用，或仅为给原本魔幻常量一个名称。
* 添加 `Spring.GetWaterPlaneLevel() -> number waterPlaneHeight`。功能类似，但表示预期水面为一个平面。
用法与上述相同，但在没有 x/z 坐标的情况下使用。

### 插入游戏时间
* 在未同步的 Lua 中新增了 `Spring.GetGameSecondsInterpolated() -> number` 函数。  
与 `GetGameSeconds` 不同，该函数在渲染过程中持续流动；与 `GetDrawSeconds` 不同，其流动值反映了游戏速度（包括暂停时的停止）。  
与 `GetGameFrame` 和 `GetFrameTimeOffset` 不同，它使用的是自然单位，而非技术上的帧抽象单位。  
* 着色器：将 `timeInfo.z` 统一变量从绘制帧号改为插值后的游戏秒数。

### 滑动
* 单位若受到与自身移动方向相反的足够大的冲击力，就会发生打滑。此前单位仅在撞击侧面时才会发生打滑。
* 新增了 `Spring.SetUnitPhysicalStateBit(number unitID, number stateBit) -> nil`，用于设置单位的物理状态位。目前必须使用魔法常量来表示位值。
例如可用于将单位从地面脱离并触发打滑效果。
* 新增单位定义：`rollingResistanceCoefficient`，用于当单位速度超过正常最大速度时降低其速度。默认值为 0.05。
* 新增单位定义：`groundFrictionCoefficient`，用于在打滑时降低单位速度。默认值为 0.01。
* 新增单位定义：`atmosphericDragCoefficient`，在打滑且速度超过最大速度时降低单位速度。默认值为 1.0。

### 调试工具
* `/debugcolvol` 现在还会以绿色显示选中区域。
* `/track 1 unitID unitID unitID` 可让您通过命令指定要跟踪的单位。如果未指定 unitID，仍按旧方式使用当前选中的单位。
* 新增了布尔型Spring设置 `AnimationMT`，默认为 true。若出现不同步问题，请将其设为 false。该设置将在一段时间后移除，届时 MT 动画将被证明具备同步安全性。
* COB 零件错误会显示造成问题的单位定义名称（因为同一脚本可被多个单位共享，且拥有不同的零件列表）。

# 修复
* 现在通过 `CMD.INSERT` 在工厂中插入带有 SHIFT 和/或 CTRL 副件的“建筑单位”命令（例如 x5/20）将能正确工作（之前被忽略，会变成 x1）。
* 修复了使用 `SFX.FIRE_WEAPON` 杀死某个物体的单位有时仍会继续朝其站立位置射击的问题。
* 修复了处理文件路径的 `VFS` 函数性能异常缓慢的问题。
* 修复了 `Spring.GetTeamUnitsByDefs` 显示的信息超出正常范围的问题。
* `Spring.GetUnitWeaponState(unitID, "burstRate")` 现在能正确返回小数值（此前仅返回整数）。
* 修复了通过鼠标中键旋转摄像机时取消单位追踪模式的问题。
* 修复了超时客户端有时仍能发送数据包并导致不同步的问题。
