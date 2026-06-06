+++
title = '引用纹理'
date = 2025-07-02T21:17:05-00:00
draft = false
author = "mcukstorm"
translator = "Cru"
+++

## 概述
在Lua中，着色器和RmlUi的纹理元素主要通过本文描述的格式之一的字符串来引用纹理。

许多与单位或功能相关的情况中会使用 UnitDefId，该ID在运行时生成，且可能因添加或删除单位定义而发生变化，因此不建议硬编码这些数字，应从单位定义的名称中查找。参见 [单位类型基础指南]({{% ref "/docs/guides/getting-started/unit-types-basics/#the-unitdefs-table-inside-wupgets" %}})

## 术语
- 单位名称 - 为单位分配的唯一字符串标识符，例如在BAR游戏内容中“armcom”表示舰队指挥官  
- 单位定义ID - 运行时由Spring/Recoil引擎为该单位分配的数值ID

## 纹理参考
在Recoil中的纹理通常通过下面列出的格式之一的字符串来引用。

`#<UnitDefId#int>` -- 构建图片 / 用于展示在构造者可建造物品列表中显示的图像，例如 `#101`

`^<UnitDefId#int>` -- 雷达图标用于设备，例如 `^101`

`%<UnitDefId#int>:<texNum#int[0-1]>` -- 单位纹理，对于UnitDefId，每个单位在此引用两个纹理，分别为0和1，例如： `%35:1`

上述单位纹理的参考也可用于特征（例如树木）。特征纹理与单位基本相同，但其 UnitDefId 始终为负值，例如： `%-12:0`

`!<luaTextureId#int>` -- Lua 创建了在纹理创建时提供的ID所引用的纹理

`*<atlasId#int>` -- Atlas/sprite（由LUA CreateTextureAtlas添加）

`$<textureName>` -- 还有一些命名纹理，ssmf_splat_normals、extra 和 info 也包含其下的子纹理。以下是许多可用命名纹理的表格。

**FIXME：此表格仍在制作中**

| 纹理名称				 | 描述																										| 注释																			       |
|------------------------|----------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| $units                 |                                                                                                          |                                                                                      |
| $units1                |                                                                                                          |                                                                                      |
| $units2                |                                                                                                          |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| **立方体贴图**		 | 本节中的所有纹理均为 TEXTURE_CUBE_MAP_ARB																|                                                                                      |
| $specular              |                                                                                                          |                                                                                      |
| $reflection            |                                                                                                          |                                                                                      |
| $map_reflection        |                                                                                                          |                                                                                      |
| $sky_reflection        |                                                                                                          |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| **特殊**				 |                                                                                                          |                                                                                      |
| $shadow                |                                                                                                          |                                                                                      |
| $shadow_color          |                                                                                                          |                                                                                      |
| $heightmap             |                                                                                                          |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| **SMF 映射**			 |                                                                                                          |                                                                                      |
| $grass                 |                                                                                                          |                                                                                      |
| $detail                |                                                                                                          |                                                                                      |
| $minimap               |                                                                                                          |                                                                                      |
| $shading               |                                                                                                          |                                                                                      |
| $normals               |                                                                                                          |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| **SSMF 映射**		 |                                                                                                          |                                                                                      |
| $ssmf_normals          |                                                                                                          |                                                                                      |
| $ssmf_specular         |                                                                                                          |                                                                                      |
| $ssmf_splat_distr      |                                                                                                          |                                                                                      |
| $ssmf_splat_detail     |                                                                                                          |                                                                                      |
| $ssmf_splat_normals    | 包含一个按数值索引的点阵法向量数组					                                                    |                                                                                      |
| $ssmf_splat_normals:X  | 其中 X 是一个数值索引	                                                                                |                                                                                      |
| $ssmf_sky_refl         |                                                                                                          |                                                                                      |
| $ssmf_emission         |                                                                                                          |                                                                                      |
| $ssmf_parallax         |                                                                                                          |                                                                                      |
|                        |                                                                                                          |                                                                                      |
|                        | $info下面的项使用_或：分隔符，这些是由Rendering/Map/InfoTexture类产生的									|                                                                                      |
| $info                  | 所有地图叠加层                                                                                           |                                                                                      |
| $info:los              | 视距LOS地图叠加                                                                                          |                                                                                      |
| $info:airlos           |                                                                                                          |                                                                                      |
| $info:height           | 高度图叠加		                                                                                        |                                                                                      |
| $info:metal            | 金属地图叠加		                                                                                        |                                                                                      |
| $info:metalextraction  |                                                                                                          |                                                                                      |
| $info:path             | 路径可通行性-地图叠加		                                                                            |                                                                                      |
| $info:radar            |                                                                                                          |                                                                                      |
| $info:heat             | 尚未实现？路径热力图叠加					                                                                |                                                                                      |
| $info:flow             | 尚未实现？路径流量图叠加					                                                                |                                                                                      |
| $info:pathcost         | 尚未实现？路径成本图叠加					                                                                |                                                                                      |
| $extra                 | 看起来像是 $info 的别名？	                                                                            |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| $map_gbuffer_normtex   | 包含当前视图中地图的平滑法线缓冲区，以世界坐标系表示								                        | 要得到真正的法向量，必须将该向量乘以2然后减去1。									   |
| $map_gbuffer_difftex   | 包含当前视角中地图的散射纹理缓冲区。					                                                    |                                                                                      |
| $map_gbuffer_spectex   | 包含当前视角中地图的镜面纹理。					                                                        |                                                                                      |
| $map_gbuffer_emittex   | 对于发光材料（通常使用“bloom”作为标准用法）。			                                                |                                                                                      |
| $map_gbuffer_misctex   | 用于任意着色器数据。		                                                                                |                                                                                      |
| $map_gbuffer_zvaltex   | 包含当前视图中地图的深度值（z缓冲区）。				                                                    |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| $map_gb_nt             | $map_gbuffer_normtex 的别名	                                                                            |                                                                                      |
| $map_gb_dt             | $map_gbuffer_difftex 的别名	                                                                            |                                                                                      |
| $map_gb_st             | $map_gbuffer_spectex 的别名	                                                                            |                                                                                      |
| $map_gb_et             | $map_gbuffer_emittex 的别名	                                                                            |                                                                                      |
| $map_gb_mt             | $map_gbuffer_misctex 的别名	                                                                            |                                                                                      |
| $map_gb_zt             | $map_gbuffer_zvaltex 的别名	                                                                            |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| $model_gbuffer_normtex | 包含视图中模型的世界坐标系下的平滑法线缓冲区											                    | 要得到真正的法向量，必须将该向量乘以2然后减去1。									   |
| $model_gbuffer_difftex | 包含视图中模型的散射纹理缓冲区。							                                                |                                                                                      |
| $model_gbuffer_spectex | 包含所视模型的镜面纹理。								                                                    |                                                                                      |
| $model_gbuffer_emittex | 对于发光材料（通常使用“bloom”作为标准用法）。			                                                |                                                                                      |
| $model_gbuffer_misctex | 用于任意着色器数据。		                                                                                |                                                                                      |
| $model_gbuffer_zvaltex | 包含视图中模型的深度值（z缓冲区）。						                                                |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| $mdl_gb_nt             | $model_gbuffer_normtex 的别名                                                                            |                                                                                      |
| $mdl_gb_dt             | $model_gbuffer_difftex 的别名                                                                            |                                                                                      |
| $mdl_gb_st             | $model_gbuffer_spectex 的别名                                                                            |                                                                                      |
| $mdl_gb_et             | $model_gbuffer_emittex 的别名                                                                            |                                                                                      |
| $mdl_gb_mt             | $model_gbuffer_misctex 的别名                                                                            |                                                                                      |
| $mdl_gb_zt             | $model_gbuffer_zvaltex 的别名                                                                            |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| $font                  | 字体字形图集	                                                                                            |                                                                                      |
| $smallfont             | 小字体字形图集		                                                                                    |                                                                                      |
| $fontsmall             | $fontsmall 的别名                                                                                        |                                                                                      |
|                        |                                                                                                          |                                                                                      |
| $explosions            | 爆炸效果纹理图集				                                                                            |                                                                                      |
| $groundfx              | 包含地面闪光和地震圈等元素的地面效果纹理图集（游戏专用）												    |                                                                                      |

