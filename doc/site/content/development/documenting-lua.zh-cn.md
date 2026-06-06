+++
title = "编写 Lua"
+++

本文介绍了面向**引擎开发者**的 Lua 使用方法。如需将该库添加到您的 Recoil 游戏中，请参阅 [Lua 语言服务器指南](/docs/guides/getting-started/lua-language-server)。

Lua 库是一组可包含在 Recoil 游戏中的类型定义，用于提供智能提示和类型检查。该库位于 [recoil-lua-library](https://github.com/beyond-all-reason/recoil-lua-library) 仓库中。

库是根据项目中的文档注释生成的。

## 生成库

### 自动库生成

库由 {{< repo "tree/master/.github/workflows/generate-lua-library.yml" "Generate Lua library workflow" >}} 生成。该工作流会在每次合并拉取请求时自动运行。

### 手动生成库

要测试库生成，你可以本地运行 [lua-doc-extractor](https://github.com/rhys-vdw/lua-doc-extractor)。

安装 lua-doc-extractor：

```bash
npm install -g lua-doc-extractor
```

在根目录下运行：

```bash
# First delete any previously generated output.
rm -rf rts/Lua/library/generated

# Regenerate the library.
lua-doc-extractor rts/Lua/*.cpp --dest rts/Lua/library/generated
```

## 文档注释

### C++文件中的文档注释

特殊注释块由 [lua-doc-extractor](https://github.com/rhys-vdw/lua-doc-extractor) 解析，并转换为 [定义文件](https://luals.github.io/wiki/definition-files/)。

注释块必须以 `/***` 开头，且通常每行以 `*` 开头。

```cpp
/***
 * @function Spring.GetFeaturesInScreenRectangle
 *
 * Get features inside a rectangle area on the map
 *
 * @param left number
 * @param top number
 * @param right number
 * @param bottom number
 * @return nil|number[] featureIDs
 */
int LuaUnsyncedRead::GetFeaturesInScreenRectangle(lua_State* L)
{
 // CPP code
}
```

这些注释大多直接复制到Lua库中，但某些代码生成需要使用[自定义标签](https://github.com/rhys-vdw/lua-doc-extractor?tab=readme-ov-file#custom-tags)。

所有文本均支持 Markdown。

### 授权库文件

手写的库文件存放在 `/rts/Lua/Library/` 目录下。当多个 C++ 文件中使用相同的 Lua 类型时，应使用这些文件。

当工作流运行时，`/rts/Lua/Library/` 下的所有文件都会直接复制到库中。

## 注释

[注释完整列表](https://luals.github.io/wiki/annotations/)

### 常见注释

- [`@function`](https://github.com/rhys-vdw/lua-doc-extractor?tab=readme-ov-file#function-name)
- [`@table`](https://github.com/rhys-vdw/lua-doc-extractor?tab=readme-ov-file#table-name)
- [`@param`](https://luals.github.io/wiki/annotations/#param)
- [`@return`](https://luals.github.io/wiki/annotations/#return)
- [`@field`](https://luals.github.io/wiki/annotations/#field)

### 类型

[类型列表](https://luals.github.io/wiki/annotations/#documenting-types)

#### 常见类型

- `integer`
- `nil`
- `any`
- `boolean`
- `string`
- `number` （用于浮点数）
- `integer`
- `table<,>`

{: .note }

> 字面意思（例如 `true`、`false`、`5`）也可作为类型使用。当表格被用作集合时，`true` 会很有用，例如
>
> ```
> table<string, true>
> ```

#### Unions

联合类型可以用 `|`.

```
string|boolean|number
```

以 `?` 结尾的后缀是与 `nil` 连接的简写，例如 `string?` 等价于 `string|nil`。

#### 数组

数组类型表示为 `type[]`。

- `number[]` — 数字数组。
- `string?[]` — 字符串或 null 的数组。
- `number|string[]` — 单个数字，或字符串数组。
- `(number|string)[]` — 包含数字和字符串混合的数组。

## 示例

### 函数

- 必须以 `@function TableName.FunctionName` 开头。  
- 可以包含任意数量的描述，应添加在第一个之后。  
- 使用 `@param parameterName type Description...` 指定参数。  
- 使用 `@return type name Description...` 指定返回类型。  
- 多个返回值时，每行一个。

{: .warning }

> `@return` 必须在名称 _前_ 指定类型，而 `@param` 则是在类型前指定名称。

````cpp
/***
 * @function Math.Add
 *
 * Add two integers together.
 *
 * This function will add two numbers together and return the result.
 *
 * ```lua
 * local totalHeight = Math.Add(legLength, upperBodyHeight)
 * ```
 *
 * @param a integer The first number.
 * @param b integer The second number.
 * @returns integer result The sum of `a` and `b`.
 */
````

### 类

结构化数据以类的形式表示，它代表一个包含预期键值对的表格。

- 必须以 `@class ClassName` 开头。该名称将成为可在注解中使用的类型。
- 字段通过 `@field fieldName type Description...` 来指定。

```cpp
/***
 * @class Color
 *
 * Describes an RGB color value.
 *
 * @field red number The red value.
 * @field green number The green value.
 * @field blue number The blue value.
 */

/***
 * @function ColorUtility.LerpColor
 * @param from Color
 * @param to Color
 * @param value number The mix (in range `[0,1]`) of colors to combine. `1` will return `to` and `0` will return `from`.
 * @return Color color
 */
```

### 表格

全局表格可以这样定义：

```cpp
/***
 * @table Spring
 */
```

这将定义一个表。该表随后可以在函数名中引用，例如 `@function Spring.MyFunction`。

常量表也可以使用 `@field` 表示：

```cpp
/***
 * @table CoolNumbers
 * @field Pi number
 * @field SixtyNine integer
 * @field FourTwenty integer
 */
```

> [!WARNING]
> table 是一个可以在 Lua 中访问的全局变量，而不是像 `@class` 这样的类型。
