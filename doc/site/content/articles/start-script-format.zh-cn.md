+++
title = '开始脚本格式'
date = 2025-05-10T01:21:55-07:00
draft = false
translator = "Cru"
+++
```
// 属性键不区分大小写

// == IP 支持 ==
// IP属性接受：
// - IPv4 地址，如 192.168.0.20 或 123.123.123.123  
// - IPv6 地址，如 2001:0db8:85a3:8a2e:: 或 fe80::1  
// - IPv4 任意地址 0.0.0.0（允许对所有本地 IPv4 地址进行入站连接）  
// - IPv6 任意地址 ::（允许对所有本地 IPv6 地址进行入站连接）  
// - 无主机名

// 最小客户端示例
// 如果以客户端方式启动 spring，则需要以下信息才能正常工作
[GAME]
{
	HostIP=xxx.xxx.xxx.xxx; // 服务器所托管的IP地址。请参见本文档开头的“IP支持”部分。
	HostPort=xxx;       //（可选）默认值为 8452
	SourcePort=0;       //（可选）默认值为0。如果要设置不同的源端口（作为客户端），请在此处设置；0表示由操作系统选择，应优先使用。

	MyPlayerName=somename;
	MyPasswd=secretpassword;
	IsHost=0;           // 告诉引擎这是一个客户端
}

// 主机示例
// 请注意，客户端的相同值也必须在此处设置
[GAME]
{
	MapName=;           // 和……在一起。SMF 扩展
	GameType=Balanced Annihilation V6.81; // 主模组名称、快速标签名称或存档名称
	GameStartDelay=4;   // 可选，以秒为单位（无符号整数），默认值：4
	                    // 游戏开始前的秒数，
	                    // 从所有玩家连接并准备就绪的那一刻开始计时。
	StartPosType=x;     // 0 固定，1 随机，2 游戏内选择，3 游戏前选择（参见 StartPosX）

	DemoFile=demo.sdfz; // 如果设置此游戏，它将是一个多人演示回放。
	SaveFile=save.ssf;  // 如果设置，此游戏是保存游戏的延续
	RecordDemo=1;       // 设置为0以禁用演示文件记录

	HostIP=xxx.xxx.xxx.xxx; //（可选）要托管的IP地址。省略或使用空值表示在任意本地IP上托管。参见本文档开头的“IP支持”部分。
	HostPort=xxx;       //（可选）默认值为8452，客户端将连接到该端口。
	SourcePort=0;       // 对主机无影响
	AutohostIP=xxx.xxx.xxx.xxx; // 与 Spring 通信时，需指定 autohost 正在监听的 IP 地址。详情请参见本文档开头的“IP 支持”部分。
	AutohostPort=X;     // 与 Spring 通信，指定您要监听的端口（作为主机）

	MyPlayerName=somename; // 我们的游戏昵称（必须与玩家的 Name= 字段匹配）

	IsHost=1;           // 0：本次实例中不会启动任何服务器
	                    // 1：启动服务器
	NumPlayers=x;       // 非强制要求，但可用于调试目的
	NumTeams=y;         // 我也是这样，设置一下来检查脚本是否正确
	NumAllyTeams=z;     // 见上文

	FixedRNGSeed = 1; // 使用特定种子初始化同步的随机数生成器，以实现可重现的运行，例如用于基准测试或剧情场景。
                          // 默认值 0 将生成一个随机种子。

	// 玩家（控制一支队伍，如果未设置 IsHost，则玩家0为主持人）
	[PLAYER0]
	{
		Name=name;      // 纯信息，例如大厅服务器上的账户名称
		Password=secretpassword; // 只有当玩家正确设置了MyPasswd时，才能连接。
		Spectator=0;
		Team=number;    // 该玩家控制的队伍
		IsFromDemo=0;   // 仅可与Demofile配合使用（见上文）
		CountryCode=;   // 玩家所在国家（如已知）（nl/de/it等）
		Rank=-1;
	}
	// 更多玩家

	// 小规模战斗AI（控制一支队伍）
	[AI0]
	{
		Name=name;     // [可选] 纯信息，例如大厅中设置的名称
		               // 游戏中实际使用的名称为：
		               // "${Name} (owner: ${player.Name})"
		ShortName=RAI; // Skirmish AI 库的简称或名称
		               // 控制这支队伍的LUA AI。
		               // 查看 spring.exe --list-skirmish-ais 以获取可能的值
		Team=number;   // 这个AI控制的队伍
		Host=number;   // 运行此AI的玩家计算机
		               // 例如，对于上面的[PLAYER0]，这将是0
		Version=0.1;   // [可选] 此小规模对战AI的版本
		[OPTIONS]      // [可选] 包含AI特定选项
		{
			difficultyLevel=1;
		}
	}
	// 更多小规模交战AI

	//这些玩家将共享相同的单位（从一名指挥官开始等）。
	[TEAM0]
	{
		TeamLeader=x;   // 作为“领队”的玩家编号
		                // 如果这是一个由AI控制的团队，TeamLeader就是
		                // AI控制队伍的玩家编号
		                // 见AI.Host
		AllyTeam=number;
		RgbColor=red green blue;  // 红绿蓝在范围 [0-1] 内
		Side=Arm/Core;  // 我猜用户模组可能还有其他可能性
		Handicap=0;         // 已弃用，请参见下方的“优势”（Advantage）。以百分比表示的优势，常见范围：[-100, 100]，有效范围：[-100, FLOAT_MAX]
		Advantage=0;        // 优势因子（元数值）。目前仅影响收入倍率（见下文），常见值：[-1.0, 1.0]，有效范围：[-1.0, FLOAT_MAX]
		IncomeMultiplier=1; // 收集资源的乘数因子，常见值：[0.0, 2.0]，有效范围：[0.0, FLOAT_MAX]
		StartPosX=0;    // 请将这些与 StartPosType=3 一起使用
		StartPosZ=0;    // 范围以 unitsync 返回的 map 坐标表示（不同于 StartRectTop 等）
		LuaAI=name;     // 控制该队伍的LUA AI名称
		// 该队伍由[PLAYER]或[AI]控制，或者已设置LuaAI。  
		// 已弃用：TeamLeader字段表示Skirmish AI将运行在哪个计算机上。
	}
	//更多队伍

	// 各队伍必须加入一个同盟团队，且不能打破联盟关系，每支队伍都必须恰好属于一个同盟团队。
	[ALLYTEAM0]
	{
		NumAllies=0;
		Ally0=(AllyTeam number); // 这意味着这个队伍与另一个队伍结盟，不一定相反。

		StartRectTop=0;    // 请将这些与 StartPosType=2 一起使用
		StartRectLeft=0;   //  （即在地图中选择）
		StartRectBottom=1; // 范围是 0-1：0 表示左或上边缘，
		StartRectRight=1;  //   1 是右边缘或下边缘
	}
	// 更多盟友队伍

	// 用于选择要禁用或限制的单元文件的选项

	NumRestrictions=xx;

	[RESTRICT]
	{
		Unit0=armah;
		Limit0=0;       // 将所有应完全禁用的单位设为0
		Unit1=corvp;
		Limit1=50;      // >0 可用于限制，例如在 TA 中的构建限制
		//...
	}
	[MODOPTIONS]        // ModOptions.lua 中定义选项的值
	{
		StartMetal=1000;
		StartEnergy=1000;
		MaxUnits=500;       // 每队
		GameMode=x;         // 0 表示指挥官死亡，游戏继续；1 表示指挥官死亡，游戏结束；2 表示谱系；3 表示已打开
		LimitDGun=0;        // 将限制范围设为起始位置周围固定半径？
		DisableMapDamage=0; // 禁用地图陨石坑？
		GhostedBuildings=1; // 在击败它们后，幽灵敌人建筑会消失
		NoHelperAIs=0;      // 允许使用 GroupAIs 及其他辅助 AI 吗？
		LuaGaia=1;          // 使用 LuaGaia？
		LuaRules=1;         // 使用 LuaRules？
		FixedAllies=1;      // 游戏内联盟是否允许？
		MaxSpeed=3;         // 比赛开始时的速度限制
		MinSpeed=0.3;
	}
	[MAPOPTIONS]        // MapOptions.lua 中定义的选项的值
	{
		PropX=1000;
	}
}
```