+++
title = '撰写网站文章'
date = 2025-05-14T19:17:01-07:00
draft = false
author = 'Slashscreen'
translator = 'Cru'
+++

## 快速入门

本网站使用[Hugo]构建。关于如何撰写文章，几乎所有你需要的信息都可以在他们的网站上找到，以下是一份简要概述：

1. 如果你在 Recoil 仓库的根目录下，请执行 `cd doc/site`。
2. 使用 `hugo new <filepath>/<filename>.md` 可创建一个新页面。例如，`hugo new content/docs/guides/writing-widgets.md`。特别是文件名和文件夹名中包含空格时，工具往往处理困难，因此应避免使用这些名称。你可以手动创建文件，但工具会为你自动生成页眉内容（页面信息部分）。
3. 撰写你的文章。
4. 完成后，请在头部内容中将 `draft` 设置为 `false`，否则将不会被渲染。
5. `hugo server`

需要注意的是，由于默认情况下是增量编译，如果出现异常行为，请重启服务器以重新构建。如果你在操作部分视图或布局时，这种情况可能会频繁发生。

## 需要了解的事项

- 如前所述，我们当前使用的主题似乎在处理文件路径中的空格时存在问题，因此应避免使用空格。
- 如果你正在创建新文件夹，最好为该文件夹创建一个 `_index.md` 文件。

### 模板与短代码

Hugo 内容文件（即 Markdown）无法在其中执行模板。它们确实具备一种称为 [短代码](https://gohugo.io/content-management/shortcodes/) 的有限模板功能，而短代码包含可被 _执行_ 的模板代码。这些代码会在将 Markdown 页面渲染为 HTML 时执行。其语法格式如下：
符号|示例
:--|:--
Markdown|`{{%/* foo */%}} ## Section 1 {{%/* /foo */%}}`
Standard|`{{</* foo */>}} ## Section 2 {{</* /foo */>}}`

请注意，这两种写法都是无效的 Lua 语法，不会自然出现，因此在编写文档时不会意外遇到它们。  
Markdown 格式短代码会在 Markdown 被转换为 HTML 前进行渲染。标准格式则会单独渲染，并在 Markdown 渲染完成后与页面合并，因此任何内容都不会被渲染过程影响。

然而，这确实意味着临时模板的使用较为困难。不过，在大多数情况下，这并不是一个问题。  
作为参考，渲染过程对每个内容文件的处理方式如下：

1. 标准短代码从内容文件（Markdown）中提取。
2. Markdown 短代码已渲染。
3. 内容文件已渲染为HTML。
4. 标准短代码被渲染并合并到文档中。
5. 模板使用内容文档进行渲染。模板通常通过如下类似部分定义：

```
<div class="content">
  {{ .Content }}
</div>
```

这会导致所有模板命令在内容文件被处理之前就被处理，因此内容文件中的任何模板指令都会被忽略。这样一来，你的代码就安全了。

## 短代码、布局及其他库

除了默认的短代码外，您还可以使用网站或我们的主题 Hextra 添加的一些额外短代码。

### Hextra

- 我们主题的短代码可在[文档中](https://imfing.github.io/hextra/docs/guide/shortcodes/)找到  
- Hextra 还支持使用以下内容：
  - 语法高亮  
  - LaTeX  
  - 图表/流程图

请注意，我们更倾向于使用 GitHub 的注释 Markdown 语法，而不是使用高亮标注。

### Recoil 网站

网站专门编写了一些简短的代码片段和布局。它们在大多数情况下可能并不实用，但值得记录下来。

- 短代码 `contributors` 会渲染来自 GitHub 的贡献者列表，用于网站首页。
- 短代码 `latest_release` 会显示 Recoil 最新版本的链接，以及下载链接和发布页面链接。该短代码也用于网站首页。
- 布局 `commands` 会从网站 `data` 目录中提供的数据文件中渲染聊天命令列表。此布局用于 [Unsynced](content\docs\unsynced-commands.md) 和 [Synced](content\docs\synced-commands.md) 页面。它期望一个符合以下结构的 JSON 文件：

```json
{
  "command-name": {
    "arguments": {
      "": "Default command description",
      "arguments": "Action description"
    },
    "cheatRequired": false,
    "command": "CommandName",
    "description": "'twas brillig, and the slithy toves"
  }
}
```

命令在 head matter 中还需要以下信息：

- `params.context`：用于获取数据的名称（即相对于数据文件夹的数据文件路径，不包含扩展名）。例如：`context = "synced_commands"`。
- `layout = "commands"`
- `type = "docs"`

[Hugo]: https://gohugo.io/
