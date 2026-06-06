+++
title = "发布版 105-1354"
author = "sprunk"
translator = "Cru"
+++

自版本105-1214发布以来，直至2022年11月发布的**版本105-1354**的更新日志。

## 注意事项
* Ctrl 和 Alt 作为相机移动修饰键未被硬编码，而是变成了绑定操作。引擎默认会像之前一样将其绑定，但如果你清除了这些绑定并自行处理，可能需要重新绑定它们。
* `/dynamicSky` 命令和 `AdvSky` 弹簧设置已被移除。
* 重生功能不再将生命值设为 5%。请参阅“从 Spring 迁移”指南以获取替代方案。请注意，实现该功能的 Lua 函数直到后续版本才被添加，才能完全正确地工作。
* 在 `gl.Text()` 中应尊重 `gl.Color()` 的使用。可能需要确保设置了正确的颜色，例如通过内联颜色代码进行设置。

## 功能与修复

### 小地图
* 新增布尔型Spring设置 `MiniMapCanFlip`，默认值为 0。启用后，当相机旋转角度在 90 到 270 度之间时（包括旋转相机或使用 `/viewtaflip` 命令翻转，且相机默认不旋转），迷你地图坐标将被翻转。
* 添加 `Spring.GetMiniMapRotation() → rot`，目前仅返回 0 或 π（视是否翻转而定）。
* 迷你地图现在允许在光标位于内部时创建标签，并将其转换到地图位置。
* 新增布尔型Spring设置：`MiniMapCanDraw`，默认值为 0。启用后，其他绘图事件（如绘制线条/擦除）可通过迷你地图触发。

### 相机控制
* 添加 `Spring.GetCameraRotation() → rotX, rotY, rotZ`。此功能也适用于最小地图翻转检测。
* Ctrl 和 Alt 作为相机移动修饰键已从硬编码改为绑定。引擎默认会将它们绑定为与之前相同，但如果你清除了这些绑定并自行处理，则可能需要重新绑定。
* 为此前硬编码的行为添加了 `movetilt`、`movereset` 和 `moverotate` 操作，引擎默认分别对应 `Any+ctrl` 和 `Any+alt`（即之前的默认行为）。
* 当禁用“按住鼠标中键平移相机”功能时，应尊重 `InvertMouse` 的Spring设置。
* 增加配置选项，通过新增的配置值（`CameraMoveFastMult`、`CameraMoveSlowMult`）以及针对特定相机的缩放因子（`CamSpringFastScaleMouseMove`、`CamSpringFastScaleMousewheelMove`、`CamOverheadFastScale`），实现对移动速度的可配置性。
* 修复在某些情况下 `movefast` 和 `moveslow` 操作未被正确识别的问题。

### 快捷键
* 支持在 uikeys 中使用 `keyreload` 和 `keyload` 命令  
* 添加 `/keydefaults` 命令，用于加载默认值  
* 支持 `keyload`、`keyreload` 和 `keysave` 命令的文件名参数

### 天空盒
* 允许从正方形2D纹理加载天空盒  
* 取消弃用 `AdvSky` 的Spring设置  
* 移除过时的 `/dynamicsky` 选项

### 杂项
* 添加 `Platform.cpuLogicalCores` 和 `Platform.cpuPhysicalCores` 常量  
* 新增操作：`group unset`，移除对选定单位的任何分组设置  
* 添加 `Spring.GetTeamAllyTeamID(teamID) → allyTeamID`  
* 添加 `Spring.GetProjectileAllyTeamID(projectileID) → allyTeamID`  
* 添加 `Spring.SetWindow{Min,Max}imized() → bool success`（也包含在 LuaIntro 中）  
* 添加 `Spring.LoadModelTextures(string modelName) -> bool success` 用于预加载模型纹理。如果模型未找到或为 3do 格式，则返回 false  
* 添加 `damage.debris` 模块规则，用于碎片伤害，默认值为 50（与之前相同）  
* 添加 `reclaim.unitDrainHealth` 模块规则，用于判断回收是否消耗生命值。主要用于反向线框方法  
* 在 `Spring.MarkerErasePosition` 中添加第 5 和第 6 个参数：onlyLocal 和 playerID。使擦除操作仅在本地执行，可选择性地模拟由指定玩家执行的效果，与现有的 MarkerAddPoint 和 MarkerAddLine 接口一致。注意：第 4 个参数目前未使用（保留用于半径参数）  
* 修复 `DualScreenMode` 设置的功能：根据 `DualScreenMiniMapOnLeft` 的设置，在最左侧或最右侧显示器的窗口区域内绘制修复了与视图位置相关的引擎问题，并添加了 `DualScreenMiniMapAspectRatio`，以在保持宽高比的同时绘制缩略图。  
* 修复了因踢出玩家导致的崩溃问题
