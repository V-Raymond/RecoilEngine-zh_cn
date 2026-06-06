+++
title = "发布版 105-902"
author = "sprunk"
translator = "Cru"
+++

自版本105-861发布以来，直至2022年4月发布的**次要版本105-902**的更新日志。

## 注意事项
* 默认的树木现在由 basecontent 提供。已移除引擎树包括其生成器和渲染器。这些树木的模型和纹理由 0 A.D. 捐赠，因此已经不再使用。

## 功能与修复

### GLDB 查询

> 警告
> 此功能仅存在于 105-2314 版本之前，现已移除。

添加 `Spring.MakeGLDBQuery(bool forced) → bool ok` 以创建对 OpenGL 驱动程序数据库的查询。通常每次只允许一次查询，强制重写它。

添加 `Spring.GetGLDBQuery(bool blocking)` 以获取上述调用所执行查询的结果。  
blocking 表示会等待响应，因此可能仅在大厅中使用，而不适用于游戏内。  
可能的返回值：  
* 如果没有查询或查询完全失败，则返回 `nil`  
* 如果尚未就绪且调用非阻塞，则返回 `false`  
* 如果已完成且驱动程序无问题，则返回 `true, false`  
* 如果成功完成，则返回 `true, true, number glMajor, number glMinor, string URL, string driver`

### 纹理
* 引擎可加载 HDR 纹理（`.hdr` 格式/扩展名），并自动升级为 FLOAT 内部存储格式（占用内存是原来的 4 倍）。无需用户手动更改  
* LegacyAtlas 现在会生成 mipmaps，此前从未如此  
* 修复了 Assimp 模型中 s3o 纹理重新加载的问题

### 定义
* 修复了 unitdef.useFootPrintCollisionVolume，现在当 `collisionVolumeScales` 为零时，`useFootPrintCollisionVolume` 标签会优先于默认的球体。
* basecontent 的 `setupdefs.lua` 正确填充了 `UnitDefs[x].hasShield`。
* 添加了 modrules 来设置默认的侧翼伤害最小/最大值，尽管这原本已可通过 unitdefs_post 实现（且更佳方式）。

### 杂项
* 添加布尔型Spring设置 `RotOverheadClampMap`，默认为 true，禁用后可让可旋转的上方摄像机移动到地图边界之外。  
* 选择键：新增过滤器 `Guarding`（队列中第一个命令为 guard）和新计数 `SelectClosestToCursor`（始终选择单个单位，不与 `SelectNum_X` 共同使用，且在追加时不会选择次近单位等）。  
* 区域回收：始终忽略正在生成的残骸（现在 CTRL 修改符仅适用于其定义中 `autoreclaimable = false` 的特征；对于正在生成的对象，使用单目标回收）。  
* 修复了 `Spring.TraceScreenRay()` 无法命中地形的问题
