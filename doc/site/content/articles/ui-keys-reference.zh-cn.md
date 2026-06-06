+++
title = '默认 UI 键参考'
date = 2025-05-10T01:14:58-07:00
draft = false
translator = "Cru"
+++

快速说明：

1. 内置的默认绑定始终会被加载。如果您希望覆盖这些绑定，请使用 unbindall 和 unbind 命令在“uikeys.txt”文件开头删除它们。
2. 一个特定的键位组合可以绑定多个操作。对于任意给定的键位组合，操作将按绑定顺序依次尝试，第一个可用且匹配操作的命令将被使用。
3. 对上述说明稍作修改，使用“Any”修饰符的键集将在不使用该修饰符的键集之后进行尝试。
4. 使用 Shift 按键修饰符的快捷键应与未按 Shift 的按键绑定（例如：绑定 "Shift+." 而不是 "Shift+\>"）。


此文件中可使用的命令：

解除所有绑定
- 移除所有绑定，并添加“bind enter chat”（其他取消绑定命令可以移除该命令）  
- 适用于删除默认的绑定

keysym \<name\> \<keycode\>
- 添加自定义键符号（默认键符号无法被覆盖）
- 名称必须以字母开头，且只能包含字母、数字和 '_'
- \<keycode\> 可以是当前已识别的键符号
（例如： "keysym menu 0x13F" 或 "keysym radar r"）

fakemeta \<keysym\>
- 为 Meta 副修饰键分配一个辅助键（“space” 是一个不错的选择）
- 使用 "fakemeta none" 来禁用此功能

bind \<keyset\> \<action\>
- 将操作添加到键集的操作列表中
- 操作可以是简单的命令，也可以是带参数的命令

取消绑定 \<keyset\> \<action\>
- 从键集的动作列表中移除该动作
- 动作和键集必须完全匹配

unbindkeyset \<keyset\>
- 移除所有使用该键集的绑定

unbindaction \<action\>
- 移除所有使用该操作（即命令，此名称不准确）的绑定


* 注意：这些命令也可以通过游戏内的聊天行使用斜杠命令语法（如 /bind、/unbind 等）来执行。




键集格式：

键位组合是指主键及其修饰键的组合。

以下是一些示例：

    bind            a  fake_action
    bind       Ctrl+a  fake_action
    bind          C+a  fake_action
    bind Ctrl+Shift+a  fake_action
    bind          *+a  fake_action

格式如下所示：

    [\<Modifier\>+]...[\<Modifier\>+]\<keysym\>


修饰词（及其缩写）如下：

    Any   (*)
    Alt   (A)
    Ctrl  (C)
    Meta  (M)
    Shift (S)

特殊的“Any”修饰符使得键位集与实际修饰符的当前状态无关，始终匹配。

已知的键符号（key symbols）列在本文件末尾。  
如果您想使用 Spring 不支持的按键，可以使用十六进制表示法。以下是两个等效的绑定方式：

    bind Ctrl+0x20  firestate 0   hold fire
    bind Ctrl+space firestate 0   hold fire



Keychains（98.0新增）：

钥匙链是一组按键的定时组合，
例如“先按A，再按B”。

以下是一些示例：

    bind            a,b  fake_action
    bind Ctrl+a, Ctrl+b  fake_action

格式如下所示：

    \<keyset\>, \<keyset\>, ...




额外运行时命令

/keyload   ：加载 uikeys.txt 绑定（不会清除当前绑定）<br>
/keyreload ：加载 uikeys.txt 绑定（先清除当前绑定）<br>
/keysave   ：将当前绑定保存到 'uikeys.tmp'（注意：'tmp' 与 'txt'）<br>
/keysyms   ：将已知的键码打印到标准输出 <br>
/keycodes  ：将已知的键盘码打印到标准输出 <br>
/keyprint  ：将当前绑定打印到标准输出 <br>
/keydebug  ：将调试信息打印到标准输出（每次按键时）<br>




默认绑定：

参见 rts/Game/UI/KeyBindings.cpp

| 键符          | 键码     |
|---------------|----------|
| !             | 0x021    |
| "             | 0x022    |
| #             | 0x023    |
| $             | 0x024    |
| %             | 0x025    |
| &             | 0x026    |
| '             | 0x027    |
| (             | 0x028    |
| )             | 0x029    |
| *             | 0x02A    |
| +             | 0x02B    |
| ,             | 0x02C    |
| -             | 0x02D    |
| .             | 0x02E    |
| /             | 0x02F    |
| 0             | 0x030    |
| 1             | 0x031    |
| 2             | 0x032    |
| 3             | 0x033    |
| 4             | 0x034    |
| 5             | 0x035    |
| 6             | 0x036    |
| 7             | 0x037    |
| 8             | 0x038    |
| 9             | 0x039    |
| :             | 0x03A    |
| ;             | 0x03B    |
| <             | 0x03C    |
| =             | 0x03D    |
| >             | 0x03E    |
| ?             | 0x03F    |
| @             | 0x040    |
| [             | 0x05B    |
| \             | 0x05C    |
| ]             | 0x05D    |
| ^             | 0x05E    |
| _             | 0x05F    |
| `             | 0x060    |
| a             | 0x061    |
| alt           | 0x134    |
| b             | 0x062    |
| backspace     | 0x008    |
| c             | 0x063    |
| clear         | 0x00C    |
| ctrl          | 0x132    |
| d             | 0x064    |
| delete        | 0x07F    |
| down          | 0x112    |
| e             | 0x065    |
| end           | 0x117    |
| enter         | 0x00D    |
| esc           | 0x01B    |
| escape        | 0x01B    |
| f             | 0x066    |
| f1            | 0x11A    |
| f10           | 0x123    |
| f11           | 0x124    |
| f12           | 0x125    |
| f13           | 0x126    |
| f14           | 0x127    |
| f15           | 0x128    |
| f2            | 0x11B    |
| f3            | 0x11C    |
| f4            | 0x11D    |
| f5            | 0x11E    |
| f6            | 0x11F    |
| f7            | 0x120    |
| f8            | 0x121    |
| f9            | 0x122    |
| g             | 0x067    |
| h             | 0x068    |
| home          | 0x116    |
| i             | 0x069    |
| insert        | 0x115    |
| j             | 0x06A    |
| joy0          | 0x12C    |
| joy1          | 0x12D    |
| joy2          | 0x12E    |
| joy3          | 0x12F    |
| joy4          | 0x130    |
| joy5          | 0x131    |
| joy6          | 0x132    |
| joy7          | 0x133    |
| joydown       | 0x141    |
| joyleft       | 0x142    |
| joyright      | 0x143    |
| joyup         | 0x140    |
| joyw          | 0x193    |
| joyx          | 0x190    |
| joyy          | 0x191    |
| joyz          | 0x192    |
| k             | 0x06B    |
| l             | 0x06C    |
| left          | 0x114    |
| m             | 0x06D    |
| meta          | 0x136    |
| n             | 0x06E    |
| numpad*       | 0x10C    |
| numpad+       | 0x10E    |
| numpad-       | 0x10D    |
| numpad.       | 0x10A    |
| numpad/       | 0x10B    |
| numpad0       | 0x100    |
| numpad1       | 0x101    |
| numpad2       | 0x102    |
| numpad3       | 0x103    |
| numpad4       | 0x104    |
| numpad5       | 0x105    |
| numpad6       | 0x106    |
| numpad7       | 0x107    |
| numpad8       | 0x108    |
| numpad9       | 0x109    |
| numpad=       | 0x110    |
| numpad_enter  | 0x10F    |
| o             | 0x06F    |
| p             | 0x070    |
| pagedown      | 0x119    |
| pageup        | 0x118    |
| pause         | 0x013    |
| printscreen   | 0x13C    |
| q             | 0x071    |
| r             | 0x072    |
| return        | 0x00D    |
| right         | 0x113    |
| s             | 0x073    |
| shift         | 0x130    |
| space         | 0x020    |
| t             | 0x074    |
| tab           | 0x009    |
| u             | 0x075    |
| up            | 0x111    |
| v             | 0x076    |
| w             | 0x077    |
| x             | 0x078    |
| y             | 0x079    |
| z             | 0x07A    |
| {             | 0x07B    |
| &#124;        | 0x07C    |
| }             | 0x07D    |
| ~             | 0x07E    |
