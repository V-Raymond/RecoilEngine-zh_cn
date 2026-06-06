+++
title = '设计大规模RTS游戏'
date = 2025-05-08T00:04:56-07:00
draft = false
+++

<div style="text-align: center;">
<h2 data-notoc="">Recoil 是一款经过实战检验的开源RTS引擎，结合灵活的 Lua API，让您能够为游戏实现完美的用户界面和机制，同时支持数千个复杂单位同时运行。</h2>
<h4 data-notoc="">由 Recoil 驱动的一些游戏：</h4>
</div>

{{< cards cols=2 >}}
{{< card title="超越一切" image="/showcase/beyond_all_reason.avif" link="https://beyondallreason.info" >}}
{{< card title="零度K" image="/showcase/zk.jpg" link="https://zero-k.info" >}}
{{< /cards >}}
{{< cards cols=2 >}}
{{< card title="金属派" image="/showcase/metal_factions.jpg" link="https://metalfactions.pt" >}}
{{< card title="原子全力量" image="/showcase/tap.jpg" link="http://fluidplay.co/index.php/games/tap" >}}
{{< /cards >}}
{{< cards >}}
{{< card title="横扫千军" image="/showcase/ta.jpeg" link="https://github.com/techannihilation/TA" >}}
{{< card title="分裂派" image="/showcase/splinter_faction.jpg" link="splinterfaction.info" >}}
{{< card title="机甲指挥官：遗产" image="/showcase/mcl.jpg" link="https://github.com/SpringMCLegacy/SpringMCLegacy/wiki" >}}
{{< /cards >}}

---

> [!NOTE]
> Recoil 是 [Spring](https://github.com/spring/spring) 基于 [105 分支](https://github.com/spring/spring/releases/tag/105.0.1) 的最新硬分叉，其中可能会出现并存在许多对它的引用。总体而言，大多数文档记录的 Spring API 和教程都与 Recoil 兼容，因为它们都基于 105 分支。

## 入门指南

> [!WARNING]
> 本网站尚处于早期开发阶段，因此在编写出自己的指南之前，内容将主要引用 Spring 的文档。

想知道 Recoil 是否适合您的项目？[点击这里了解](articles/choose-recoil/)！

最佳入门方式是访问我们的[入门指南](docs/guides/getting-started/)部分。此外，还有其他资源：

- [Spring 百科]（用于尚未移植到文档的信息）
- [Recoil GitHub 百科]
- [Discord 服务器]

### 下载

{{< latest_release >}}

## 贡献

[![beyond-all-reason/RecoilEngine - GitHub](https://gh-card.dev/repos/beyond-all-reason/RecoilEngine.svg)](https://github.com/beyond-all-reason/RecoilEngine)

请参阅[开发](development)指南，了解如何构建和开发 Recoil。

在为本仓库做出贡献时，请先通过[GitHub issues]、我们的[Discord 服务器]或任何其他方式与本仓库的所有者讨论您想要进行的更改。

### 感谢 Recoil 的贡献者们！

{{< contributors >}}

[Recoil 仓库]: {{% param "repo" %}}
[GitHub issues]: {{% param "repo" %}}issues
[Discord 服务器]: https://discord.gg/GUpRg6Wz3e
[Spring 百科]: https://springrts.com/wiki/Main_Page
[Recoil GitHub 百科]: {{% param "repo" %}}wiki