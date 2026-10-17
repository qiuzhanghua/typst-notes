# typst-notes

Typst 数理讲义模板的**单一源**。分发到各讲义仓库后，每个仓库保留一份 `template.typ` 副本
（自动生成、随仓库提交），因此**任何人 clone 任意仓库都能直接 `./build` 出 PDF**，
不依赖本机安装任何 Typst 包。

## 目录

```
lib.typ                    模板源码（唯一需要手工编辑的文件）
sync-template.sh           生成 / 校验脚本（bash，Linux / macOS / Git Bash）
sync-template.ps1          同上（Windows PowerShell 5.1+ / 7+）
.github/workflows/         可复用 CI：校验各仓 template.typ 没有漂移
```

## 为什么不用 `@local` 包

`@local/xxx` 是机器级依赖：不进仓库、不进 Git 历史，换台机器或开 CI 就报 unknown package。
本方案把生成物提交进各仓库，换取「clone 即构建」这条性质不变。

---

## 快速开始

一篇讲义最小骨架：

```typst
#import "template.typ": *

#set document(title: [一元二次方程])   // 可选；设置后页眉右侧显示标题

#template(
  heading2-align: left,
  page-footer: "第 1 页",
  equation-numbering: "(1)",
  reset-page: true,
  text-size: 12pt,
)[
  = 一元二次方程                      // 一级标题（居中，24pt）

  == 求根公式                         // 二级标题（20pt）

  #definition[使 $a x^2 + b x + c = 0$ 成立的 $x$ 称为该方程的根。] <def:root>

  #theorem[当 $Delta > 0$ 时方程有两个不等实根。] <thm:delta>

  #proof[
    由配方法可得 $x = frac(-b plus.minus sqrt(b^2 - 4 a c), 2a)$。 #blank() 得证。
  ]

  由 @def:root 与 @thm:delta 可知……    // 自动显示为「定义 1」「定理 1」
]
```

## 参数表

`#template(...)` 的全部参数（除 `body` 外均可省略）：

| 参数 | 默认值 | 说明 |
| --- | --- | --- |
| `body` | — | 正文内容块（位置参数，写在方括号里） |
| `heading2-align` | `center` | 二级标题对齐：`left` / `center` |
| `first-line-indent` | `none` | 首行缩进量，如 `2em`；`none` 表示不缩进 |
| `page-numbering` | `none` | 标准页码格式（Typst 计数格式），如 `"第 1 页"`；`none` 不显示 |
| `page-footer` | `none` | 自定义右对齐页脚；设置后**取代**标准页码（见下方「页码的坑」） |
| `equation-numbering` | 各仓 `.template.conf` 决定 | 公式编号格式，如 `"(1)"`；`none` 表示不编号 |
| `reset-page` | `false` | 是否把页码计数器重置为 1（单篇独立编号时用） |
| `text-size` | `12pt` | 正文字号 |
| `title-size` | `24pt` | 一级标题（文档大标题）字号 |
| `heading-size` | `20pt` | 二级标题（一、二、三…）字号 |

**页码的坑**：`page-footer: none` 时页脚取 `auto`（保留 `page-numbering` 的标准页码）；
一旦给了 `page-footer`，标准页码会被关掉，二者不会叠加。

## 环境与工具箱

模板导出以下可直接使用的名字（`#import "template.typ": *` 之后）：

| 名字 | 用法 | 说明 |
| --- | --- | --- |
| `definition` | `#definition[...] <def:x>` | 「定义」框，浅蓝，自动编号、可 `@def:x` 引用 |
| `theorem` | `#theorem[...] <thm:x>` | 「定理」框，加粗蓝边，自动编号、可引用 |
| `example` | `#example[...] <ex:x>` | 「示例」框，灰底，自动编号、可引用 |
| `remark` | `#remark[...]` / `#remark(title: "级数视角")[...]` | 「注」框，米黄，自动编号、可引用 |
| `proof` | `#proof[...]` | 「证明」段，自动加粗体标题与结尾 □ |
| `important-block` | `#important-block[...]` | 细灰边框提示框（无编号） |
| `problem-box` / `idea-box` | `#problem-box[...]` / `#idea-box[...]` | 绿/黄左侧色条框：题目 / 破题思路 |
| `blank` | `#blank()` / `#blank(2)` / `#blank(5em)` | 填空横线；无参默认 3em，数字按 em 倍数解释 |
| `cjk` | `$x_(#cjk[极值点])$` | 数学模式内指定中文字体（见下方「字体」） |
| `qed` | `#qed` | 右对齐的 □ 结束符 |

四类编号环境（定义 / 定理 / 示例 / 注）**各自独立计数、整篇连续**。

写法提醒：正文里**不要**用 `#underline[#h(3em)]` 画填空线——Typst 不会给空内容画下划线，
什么都不会显示；用 `#blank()`。

## 字体

模板在文件顶部定义两个常量，正文**只**通过它们设置字体：

```typst
#let text-fonts = ("New Computer Modern Math", "Source Han Serif")
#let cjk-font = "Source Han Serif"
```

- `New Computer Modern Math` 负责数学与拉丁字母，`Source Han Serif` 殿后做 CJK 回退——
  Typst 按列表顺序逐字查找，缺字才落到下一项，所以**列表必须是「数学字体在前、中文字体在后」**。
- 文档里**不要**再写 `#set text(font: ...)`。单字体覆盖会替换掉整个列表，中文立刻失去 CJK 回退
  （表现是中文变黑体、与正文宋体不一致）。
- 文件名/家族名陷阱：机器上装的是 `Noto Serif CJK`（同族含 `NotoSerifCJKsc-*.otf`），
  但 Typst 把 `Source Han Serif` 作为**同族别名**解析，PDF 内嵌字体会显示为
  `SourceHanSerifVF-*`。这是正常现象，不是缺字体。
- 想换字体：改顶部两个常量后重打 tag 分发，别在单篇文档里改。
- 备选中文（追加到 `text-fonts` 末尾即可）：`"Noto Sans SC"`、`"Sarasa Gothic SC"`。

## 各仓如何接入

在讲义仓库根目录放两个文件：

`.template.conf`

```
TAG=v1.0.0
EQNUM='"(1)"'      # Typst 字面量：公式编号格式；none 表示不编号
```

`sync-template.sh` / `sync-template.ps1`（从本仓复制，内容完全相同）

然后：

```bash
./sync-template.sh          # 生成 template.typ
./sync-template.sh --check  # 校验：与源不一致则退出码 1（给 CI 用）
```

```powershell
./sync-template.ps1
./sync-template.ps1 -Check
```

脚本按下面顺序找源：`$TYPST_NOTES_DIR/lib.typ` → 同级目录 `../typst-notes/lib.typ` →
GitHub `raw.githubusercontent.com/<repo>/<TAG>/lib.typ`。两个脚本生成的文件**字节一致**，
所以 macOS 和 Windows 同时维护不会互相覆盖成「有改动」。

生成物形如：

```
// ============================================================
// template.typ — 自动生成，请勿手工修改
// 源：typst-notes@v1.0.0（lib.typ）
// 重新生成：./sync-template.sh（Windows 用 ./sync-template.ps1）
// 本仓公式编号默认值："(1)"
// ============================================================

<lib.typ 全文，其中 @@EQNUM@@ 被替换成本仓取值>
```

各仓之间的差异因此只有 4 行内容（说明头 2 行 + 公式编号默认值 2 行），`diff` 出来共 8 行。

## 改模板的流程

1. 在本仓改 `lib.typ`，提交并打新 tag（如 `v1.0.1`）。
2. 各讲义仓库把 `.template.conf` 的 `TAG` 改掉，跑 `./sync-template.sh`，提交。
3. CI 跑 `./sync-template.sh --check`，防止有人手改 `template.typ` 造成漂移。

## CI

本仓提供可复用工作流 `.github/workflows/template-check.yml`。讲义仓库只需加一个 4 行文件
`.github/workflows/template.yml`：

```yaml
name: template
on: [push, pull_request, workflow_dispatch]
jobs:
  check:
    uses: qiuzhanghua/typst-notes/.github/workflows/template-check.yml@v1.0.0
```

它做的事只有一件：在讲义仓库里跑 `./sync-template.sh --check`，退出码非 0 即失败
（信息是「已过期，请运行 ./sync-template.sh」）。tag 固定成 `@v1.0.0` 是刻意的——
校验脚本自身也会随模板一起升级，要换脚本时连调用方的 `@` 版本号一起改。

## 采用本模板的仓库

| 仓库 | 公式编号 | 说明 |
| --- | --- | --- |
| high-school-math | `"(1)"` | 93 篇 |
| high-school-physics | `none` | 2 篇 |
| junior-school-physics | `"(1)"` | 1 篇 |

**未纳入**：

- `junior-high-math`（27 篇）：那一版模板的 `definition` 用的是自绘 `block` 而非
  `figure` + 自动编号，正文默认 16pt，与现模板不是同一代；直接换会改变全部 27 篇的
  版式（字号、环境编号），需要逐篇目视确认，故暂时保留原样。
- `buddha`（9 篇）：佛经排版，用的是 `cover-page` / `sutra-template` 这套完全不同的 API，
  不属于同一模板体系。

## 环境

Typst 0.15.1 实测通过。缺字体时 Typst 会给出 `unknown font family` 警告并回退，版式可能有出入。
