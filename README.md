# typst-notes

Typst 讲义模板的**单一源**。分发到各讲义仓库后，每个仓库保留一份 `template.typ` 副本
（自动生成、随仓库提交），因此**任何人 clone 任意仓库都能直接 `./build` 出 PDF**，
不依赖本机安装任何 Typst 包。

## 目录

```
lib.typ              模板源码（唯一需要手工编辑的文件）
sync-template.sh     生成 / 校验脚本（bash，Linux / macOS / Git Bash）
sync-template.ps1    同上（Windows PowerShell）
```

## 为什么不用 `@local` 包

`@local/xxx` 是机器级依赖：不进仓库、不进 Git 历史，换台机器或开 CI 就报 unknown package。
本方案把生成物提交进各仓库，换取「clone 即构建」这条性质不变。

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

脚本按下面顺序找源：`$TYPST_NOTES_DIR/lib.typ` → 同级目录 `../typst-notes/lib.typ` →
GitHub `raw.githubusercontent.com/<repo>/<TAG>/lib.typ`。

生成物形如：

```
// ============================================================
// template.typ — 自动生成，请勿手工修改
// 源：typst-notes@v1.0.0（lib.typ）
// 重新生成：./sync-template.sh
// 本仓公式编号默认值："(1)"
// ============================================================
<lib.typ 全文，其中 @@EQNUM@@ 被替换成本仓取值>
```

各仓之间的差异因此只有 4 行（说明头 2 行 + 公式编号默认值 2 行）。

## 改模板的流程

1. 在本仓改 `lib.typ`，提交并打新 tag（如 `v1.0.1`）。
2. 各讲义仓库把 `.template.conf` 的 `TAG` 改掉，跑 `./sync-template.sh`，提交。
3. CI 里跑 `./sync-template.sh --check`，防止有人手改 `template.typ` 造成漂移。

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

Typst 0.15.1 实测通过。模板依赖字体 `New Computer Modern Math` 与 `Source Han Serif`
（后者由 Noto Serif CJK 提供同名字体族别名）；缺字体时 Typst 会给出
`unknown font family` 警告并回退，版式可能有出入。
