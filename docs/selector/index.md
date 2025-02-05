# 高级选择器 {#title}

<script setup>
import ValueField from '/.vitepress/components/ValueField.vue';
</script>

一个类似 CSS 选择器的高级选择器, 能联系节点上下文信息, 更容易也更精确找到目标节点

## 为什么需要选择器 {#why}

我们的目的是查询获取屏幕上的某个节点, 最简单的情况下, 目标节点已经有唯一的 id/text 标识

这时只需要配置 id/text 就能简单执行点击, 但实际情况很多节点都缺失了这些信息, 无法简单查询获取

节点还存在很多额外信息没有被利用到, 节点所有属性可以在 [快照-14045424](https://i.gkd.li/i/14045424) 的属性面板看到

另一种常见的情况是节点没有有效属性, 这时我们需要根据周围的节点去定位

选择器就是为了解决这些问题而开发的, 下面是对选择器的详细介绍

## 语法 {#syntax}

与 CSS 类似, 一个选择器由 属性选择器 和 关系选择器 交叉组成, 并且开头末尾必须是 属性选择器

示例 `div > img` 的结构是 `属性选择器 关系选择器 属性选择器`, 它表示选择父节点是 `div` 的 `img` 节点, 这与 [相同 CSS 语法](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Child_combinator) 语义一致

另外 属性选择器 和 关系选择器 之前必须强制用 空白字符(空格/换行/回车/制表) 隔开

即 `div>img` 非法, 必须写成 `div > img`

下面分别介绍 [属性选择器](#attr) 和 [关系选择器](#connect)

## 属性选择器 {#attr}

它和 CSS 语法的 属性选择器很相似, 但更强大, 如下是一个示例

`@TextView[a=1][b^='2'][c*='a'||d.length>7&&e=false]`

`@` 表示选择此节点, 一条规则最后属性选择器 `@` 生效, 如果没有 `@`, 取最后一个属性选择器

`TextView` 代表节点的 name 属性, 而且与 CSS 相似, `*` 表示匹配任意属性

由于该选择器主要用于 Android 平台, 节点的 name 都是 java 类如 android.text.TextView 这种形式

为了方便书写规则, 上述 `TextView` 等价 `[name='TextView'||name$='.TextView']`

`[]` 内部是一个 逻辑表达式/布尔表达式

- 逻辑表达式 -> `name='TextView'||name$='.TextView'`
- 布尔表达式 -> `name='TextView'`

逻辑表达式 有两个操作符 `||` 和 `&&`. `&&` 优先级更高, 即 `[a>1||b>1&&c>1||d>1]` 等价于 `[a>1||(b>1&&c>1)||d>1]`

布尔表达式 由 [属性名](#attr-name) [操作符](#attr-operator) [值](#attr-value) 顺序构成

### 属性名 {#attr-name}

正则匹配 `^[_a-zA-Z][a-zA-Z0-9_]*(\.[_a-zA-Z][a-zA-Z0-9_]*)*$` 的字符串

类似合法变量名 `a`/`a.length`, 下面可输入属性名测试否是合法

::: raw
<ValueField />
:::

### 操作符 {#attr-operator}

| 操作符 |     名称      |     说明      |
| :----: | :-----------: | :-----------: |
|   =    |     equal     |               |
|   !=   |   notEqual    |               |
|   >    |     more      |               |
|   >=   |   moreEqual   |               |
|   <    |     less      |               |
|   <=   |   lessEqual   |               |
|   ^=   |  startsWith   |   以...开头   |
|  !^=   | notStartsWith |  不以...开头  |
|  \*=   |   contains    |    包含...    |
|  !^=   |  notContains  |   不包含...   |
|   $=   |   endsWith    |   以...结尾   |
|  !$=   |  notEndsWith  |  不以...结尾  |
|   ~=   |    matches    |  正则匹配...  |
|  !~=   |  notMatches   | 正则不匹配... |

附加说明: `matches`/`notMatches` 要求 值 必须是合法的 [Java/Kotlin 正则表达式](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html), 否则提示语法错误

### 值 {#attr-value}

值: 4 种类型, `null`, `boolean`, `string`, `int`

- null
- boolean 使用 `true`/`false`
- int 匹配 `^-?[0-9]$`, 仅支持 10 进制自然数
- string 使用 ' &#96; " 之一成对包裹, 内部字符转义使用 `\`\
   所有的转义字符示例 `\\`, `\'`, `\"`, `` \` ``, `\n`, `\r`, `\t`, `\b`, `\xfF`, `\uffFF`\
   不支持多行字符, 处于 `[0, 0x1F]` 的控制字符必须使用转义字符表示

操作符只能使用在对应的类型的值, 比如 `a>''` 类型不匹配, 将提示 `非法类型`/`非法选择器`

下面表格中 `-` 表示类型不匹配

|      |   null   | boolean  |   int    |  string  |
| :--: | :------: | :------: | :------: | :------: |
|  =   | &#10004; | &#10004; | &#10004; | &#10004; |
|  !=  | &#10004; | &#10004; | &#10004; | &#10004; |
|  >   |    -     |    -     | &#10004; |    -     |
|  >=  |    -     |    -     | &#10004; |    -     |
|  <   |    -     |    -     | &#10004; |    -     |
|  <=  |    -     |    -     | &#10004; |    -     |
|  ^=  |    -     |    -     |    -     | &#10004; |
| !^=  |    -     |    -     |    -     | &#10004; |
| \*=  |    -     |    -     |    -     | &#10004; |
| !\*= |    -     |    -     |    -     | &#10004; |
|  $=  |    -     |    -     |    -     | &#10004; |
| !$=  |    -     |    -     |    -     | &#10004; |
|  ~=  |    -     |    -     |    -     | &#10004; |
| !~=  |    -     |    -     |    -     | &#10004; |

## 关系选择器 {#connect}

用于连接两个属性选择器, 简单示例: `div > a`, 它 表示/约束 两个节点之间的关系

关系选择器 由 关系操作符 和 关系表达式 构成

关系操作符 表示查找节点的方向, 有 5 种关系操作符, `+`, `-`, `>`, `<`, `<<`

关系表达式 有两种

- 元组表达式 `(a1,a2,a3,a_n)`, 其中 a1, a2, a3, a_n 是常量有序递增正整数, 示例 `(1)`, `(2,3,5)`
- 多项式表达式 `(an+b)`, 其中 a 和 b 是常量整数, 它是元组表达式的另一种表示, 这个元组的数字满足集合 `{an+b|an+b>=1,n>=1}` 如果集合为空集则表达式非法\
  当 a<=0 时, 它具有等价的元组表达式\
  示例 `(-n+4)` 等价于 `(1,2,3)`\
  示例 `(-3n+10)` 等价于 `(1,4,7)`\
  当 a>0 时, 它表示无限的元组表达式\
  示例 `(n)`, 它表示 `(1,2,3,...)` 一个无限的元组\
  示例 `(2n-1)`, 它表示 `(1,3,5,...)` 一个无限的元组

将 关系操作符 和 关系表达式 连接起来就得到了 关系选择器

`A +(a1,a2,a3,a_n) B` : A 是 B 的前置兄弟节点, 并且 A.index 满足 B.index-(a_m), 其中 a_m 是元组的任意一个数字

`A -(a1,a2,a3,a_n) B` : A 是 B 的后置兄弟节点, 并且 A.index 满足 B.index+(a_m)

`A >(a1,a2,a3,a_n) B` : A 是 B 的祖先节点, 并且 A.depth 满足 B.depth-(a_m), 根节点的 depth=0

`A <(a1,a2,a3,a_n) B` : A 是 B 的直接子节点, 并且 A.index 满足 a_m-1

`A <<(a1,a2,a3,a_n) B` : A 是 B 的子孙节点, 并且 A.order 满足 a_m-1, A.order 是深度优先先序遍历的索引

一些表达式的简写

当 a=0 或 b=0 时, 括号可以省略, 比如 `A +(3n+0) B` -> `A +(3n) B` -> `A +3n B`, `A +(0n+3) B` -> `A +(+3) B` -> `A +3 B`

当 a=0 且 b=1 时, an+b 可以省略, 比如 `A <(0n+1) B` -> `A < B`, 此外 `A + B`,`A > B` 都与等价的 CSS 语法语义相同

当 a=1 且 b=0 且操作符是 `>`, 可以进一步简写, 比如 `A >(1n+0) B` -> `A >n B` -> `A B`, 这与等价的 CSS 语法语义相同

## 示例 {#example}

```txt
@LinearLayout > TextView[id=`com.byted.pangle:id/tt_item_tv`][text=`不感兴趣`]
```

首先找到 id=&#96;com.byted.pangle:id/tt_item_tv&#96; 和 text=&#96;不感兴趣&#96; 的 TextView, 并且父节点是 LinearLayout 的节点

此时我们得到两个节点 [LinearLayout, TextView] 根据 `@` 知道目标节点是 LinearLayout

实际上它与

```txt
TextView[id=`com.byted.pangle:id/tt_item_tv`][text=`不感兴趣`] <n LinearLayout
```

的目标匹配节点是等价的, 但是在查询算法时间复杂度上, 后者更慢

如下是网页无障碍快照审查工具, 使用它的搜索框的选择器查询可以实时测试编写的选择器

- <https://i.gkd.li/i/14045424>
- <https://i.gkd.li/i/14045424>
- <https://i.gkd.li/i/14045424>
- <https://i.gkd.li/i/14034770>
- <https://i.gkd.li/i/14031920>
- <https://i.gkd.li/i/14018243>
- <https://i.gkd.li/i/14011298>
- <https://i.gkd.li/i/13999908>

## 一些注意 {#note}

### 正则表达式优化 {#regex-optimization}

对 `matches`/`notMatches` 的优化: 如果正则表达式满足下面的条件, 选择器将使用内置的简单的函数匹配, 而不是真正地去运行一个正则表达式

- `[text~="(?is)abc.*"]` -> `startsWith('abc', ignoreCase = true)`
- `[text~="(?is).*abc.*"]` -> `contains('abc', ignoreCase = true)`
- `[text~="(?is).*abc"]` -> `endsWith('abc', ignoreCase = true)`
- `[text!~="(?is)abc.*"]` -> `notStartsWith('abc', ignoreCase = true)`
- `[text!~="(?is).*abc.*"]` -> `notContains('abc', ignoreCase = true)`
- `[text!~="(?is).*abc"]` -> `notEndsWith('abc', ignoreCase = true)`

上面的 `abc` 指代不包含 `\^$.?*|+()[]{}` 这类特殊正则字符的任意字符串, 如 `ikun` 符合, `ikun?` 不符合

简单来说就是如果你只想忽略大小写去简单匹配或不匹配一些字符, 那么直接使用上面的格式

### 正则表达式一致性 {#regex-multiplatform}

由于 选择器 需要同时满足 浏览器/Js(审查工具), Android/Java(GKD) 运行, 而这两个平台的正则表达式的底层实现和语法表示略有不同

因此为了在 Js 端实现和 Java 一致的正则表达式规范, 网页审查工具借助 [Kotlin Wasm](https://kotlinlang.org/docs/wasm-overview.html) 将正则表达式的 matches 函数接口编译为 wasm 提供给 Js 调用

Kotlin Wasm 需要你的浏览器支持 [WasmGC](https://developer.chrome.com/blog/wasmgc?hl=zh-cn), 也就是版本需要满足下列条件

![image](https://github.com/gkd-kit/gkd/assets/38517192/15c8dee9-6480-428b-be4f-45939b4046e5)

如果你的浏览器版本不满足, 正则表达式将自动回退到 Js 端实现, 以下是在 Js 端使用正则表达式需要注意的地方

比如上面的正则表达式优化例子中开头的 `(?is)` 是 Java 正则表达式的 inline flags 语法, 但实际上 Js 并不支持这样写, 只是选择器内部做了一些兼容让它支持

并且选择器的 Js 端只兼容在开头的 flags, 在内部的 flags 不支持, 此外 Java 和 Js 支持的 flags 也有不同, 某些特殊的表达式也表现也不一致

总之不要使用太过复杂(多复杂我也不知道)的正则表达式, 某些正则表达式有可能在审查工具上匹配, 但是在 GKD 上不匹配

如果你能确保正则表达式在 Js 和 Java/Kotlin 的匹配行为一致, 那就没问题

总之两种情况

- 高版本浏览器可以使用 **完整一致** 的正则表达式
- 低版本浏览器使用 **小范围不完整不一致** 的正则表达式

如果你的选择器没有使用正则表达式则不用关心此问题
