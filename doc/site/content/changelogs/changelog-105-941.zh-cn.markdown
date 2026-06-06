+++
title = "发布版 105-941"
author = "sprunk"
translator = "Cru"
+++

自版本105-902发布以来，直至2022年5月发布的**次要版本105-941**的更新日志。

### 图形
* 为 luatextures 启用 3D 和立方体贴图
* `Spring.GetRender{Units,Features}` 现在支持 GL4 CUS
* 添加 `Spring.Clear{Units,Features}PreviousDrawFlag() → nil`
* 添加 `Spring.GetRender{Units,Features}DrawFlagChanged(bool returnMasks = false) → changedIDs[, changedMasks]`

### 定时器
* 新增 `Spring.GetTimerMicros() → number`，用于获取以微秒精度的定时器
* `Spring.DiffTimers` 现在接受第四个参数，若希望以毫秒为单位的微秒级定时器进行计算

### 其他
* 为 yardmaps 新增了 'b' 设定符，用于声明一个可建造但不可通行的区域。这使得当前可升级建筑的方法能够创建锁定模式，而不会破坏路径规划。
* 添加 `group [子命令] N` 操作处理器，其中子命令可以是 `add`、`set`、`selecttoggle`、`selectadd` 或 `selectclear`。这比 `groupN` 提供了更精细的组绑定控制（后者会在操作中检查键盘修饰键）。当子命令未指定时，将选择第 N 个组。
* 短兵相接 AI：在加载事件后调用 `PostLoad()`，仅当 `loadSupported != yes` 时执行。
* 在 `screenshot` 操作中接受整数质量参数，例如 `screenshot [format] [quality]`。
