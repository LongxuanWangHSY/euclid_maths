---
name: euclids-gate-figure
description: >
  欧几里德之门数学教学配图引擎。图型路由（几何/组合→TikZ，函数图像→pgfplots，知识地图/方法地图→draw.io）、
  编译管线（pdflatex + pdftoppm → generated_images/）与出图质检纪律（九区盘点、图注一致性、数值核对）。
  触发场景：文章配图、画图、作图、示意图、几何图、函数图像、数轴图、概率树、知识地图、方法地图、配图核查、
  检查图对不对。与 euclids-gate-math-writing 配合使用（后者在配图核查一步调用本技能）。
---

# 欧几里德之门 数学教学配图引擎

配图是文章的一部分，不是插图装饰。一张图的价值在于**让读者一眼看出正文在说什么**——只画"数学上正确但看不出对应"的抽象图案，等于没画（No18 图 2 教训）。

---

## 一、图型路由

先判断画的是哪一类，再动手。**不要拿 TikZ 硬画本该用 draw.io 的东西，反之亦然。**

| 图型 | 工具 | 为什么 |
|---|---|---|
| 几何图：圆、三角形、轨迹、辅助线、共点/共圆 | **TikZ** | 精确坐标可算，与正文符号天然对齐 |
| 函数图像、数形结合、不等式区域、数列散点 | **pgfplots** | 与 TikZ 共用字体线宽，风格统一；本机已装 |
| 组合结构：图、树、棋盘、染色格、路径、网络 | **TikZ**（小规模）/ networkx 出坐标 + TikZ 描线（大规模） | 顶点边数必须与正文严格一致时，手写坐标最可靠 |
| 知识地图、方法地图、概念结构、章节脉络 | **draw.io**（调用全局 `scibox-diagram` 技能） | 这类图靠人工布局，TikZ 画起来极痛苦 |
| 概率树、决策树、状态转移 | **TikZ**（`positioning` + `edge`） | 层级结构用 TikZ 的树语法反而更短 |
| 数值统计图：直方图、热力图、多面板 | 暂缺 | 需 matplotlib，本机未装；真要用先问用户 |

路由判断的一句话：**图上的位置能不能用一个公式算出来？** 能 → TikZ/pgfplots；不能、靠版式直觉 → draw.io。

---

## 二、TikZ 管线（主力）

### 编译链

```
.tex (standalone) → pdflatex → .pdf → pdftoppm → .png → generated_images/
```

- 文档类：`\documentclass[tikz,border=10pt]{standalone}`
- 常用宏包：`amsmath, amssymb, tikz`
- 常用库：`calc, patterns, angles, quotes, arrows.meta, shapes, positioning, intersections, decorations.pathreplacing`
- 编译：`pdflatex -interaction=nonstopmode -output-directory=<dir> <file>.tex`
- 转图：`pdftoppm -png -r 200 <file>.pdf <out>`（dpi ≥ 200）

### 命名与插入

- 文件名：`no{编号}_{图号}_{语义}.png`（如 `no20_g3_radical.png`）
- 插入：`![图 N：描述](../../generated_images/no{编号}_{图号}_{语义}.png)`
  - `generated_images/` 在仓库根目录，文章在 `articles/long/`、`articles/short/`、`articles/other/` 下——从文章里引用要上溯两级（`../../`）；直接写 `generated_images/...` 会解析成 `articles/generated_images/...` 而失效

### 中文约束（硬）

**pdflatex 不支持中文标注——中文会被静默丢弃，图里出现空白。** 图内一律用英文/符号（`$P$`、`$O_1$`、`midpoint`、`axis`），中文说明写进正文图注。

生成后**必须用 `pdftotext` 验证文字层完整**——确认图内标注确实被渲染出来，而不是静默消失。

---

## 三、pgfplots：函数图像

本机 MiKTeX 已装 `pgfplots`，与 TikZ 同一管线、同一套字体线宽，**风格自动统一**。

```latex
\documentclass[tikz,border=10pt]{standalone}
\usepackage{pgfplots}
\pgfplotsset{compat=1.18}
\begin{document}
\begin{tikzpicture}
\begin{axis}[
  axis lines=middle,          % 坐标轴居中，数学课的标准画法
  xlabel={$x$}, ylabel={$y$},
  xmin=-0.5, xmax=3.5, ymin=-0.5, ymax=5.5,
  domain=0:3, samples=200,
  width=9cm, height=7cm,
  tick label style={font=\small},
]
\addplot[thick, blue] {x^2};                    % 曲线
\addplot[only marks, mark=*, red] coordinates {(1,1) (2,4)};  % 关键点
\node[above right, font=\small] at (axis cs:2,4) {$P(2,4)$};
\end{axis}
\end{tikzpicture}
\end{document}
```

**画函数图的规范**：
- 坐标轴居中（`axis lines=middle`），与正文的解析几何记号一致
- 关键点用实心点 + 字母标注，坐标可写的写在标注里
- **只有在讨论具体函数时才写具体数字**；若图用于说明一般性结论（如"任意 $a>0$"），曲线不带刻度数值，坐标轴只留箭头
- 渐近线、定义域边界用虚线；结论所在区域用浅色填充（`fill=blue!10`）
- 多曲线共图时配色不超过 3 条，并在图注里说明哪条是哪条

---

## 四、组合结构图

**小规模（点 ≤ 15）**：TikZ 手写坐标，显式控制每个顶点位置。

```latex
\tikzset{v/.style={circle, fill=black, inner sep=1.6pt}}
\foreach \p in {(0,0),(1,0),(0.5,1)} \node[v] at \p {};
\draw (0,0) -- (1,0) -- (0.5,1) -- cycle;
```

**大规模**：用 networkx 算布局坐标，导出成 TikZ 的 `\draw`/`\node` 语句（不要直接出 PNG——那会引入第二套视觉语言）。

```python
import networkx as nx
pos = nx.spring_layout(G, seed=42)   # 固定 seed，保证可复现
for n, (x, y) in pos.items():
    print(rf"\node[v] at ({x*6:.3f},{y*6:.3f}) {{{n}}};")
```

**校验（硬规则）**：出图后逐一核对**点数、边数、各顶点度数、方向、染色分组**是否与正文完全一致。图论题里画错一条边，整道题就废了。

---

## 五、draw.io：知识地图与方法地图

**这类图调用全局技能 `scibox-diagram`**（含 draw.io XML 编写规范、中文字宽预算、`check_layout.py` 静态检查、导出脚本），本技能不重复造轮子。

在数学教学里的用途：
- **知识地图**：一个专题内各概念的依赖关系（如"圆幂 → 根轴 → 根心"）
- **方法地图**：一类问题的解法分支（对应 No10 的"方法簇"结构）
- **章节脉络**：长文开头的路线预告可视化

项目约定：
- 产物放仓库根目录的 `generated_images/`，与 TikZ 图同目录
- 配色收敛到 5 色以内，同语义同色
- 交付前跑 `check_layout.py`，无 FAIL 才算过

---

## 六、出图质检（硬规则）

**看不见渲染图就不算画完。** XML 和 LaTeX 源码里一切"看着合理"，文字溢出、标签压线、点挤在一起——这些只在渲染图里现形。所以判断权交给两样外部证据：**渲染出来的 PNG**，和**与正文的逐项对照**。

### 6.1 两轮九区盘点

每轮渲染后逐区扫，不要只盯最显眼的一处：

| 区 | 看什么 |
|---|---|
| 标注文字 | 溢出边界？被线穿过？下标下探压线？图内文字有没有变成空白（中文被吞）？ |
| 点与标签 | 点的位置准吗？标签离点够远吗？多个标签互相挤在一起吗？（No20 PABC 教训） |
| 线与关系 | 平行/垂直/相切/共线在图上真的看得出来吗？辅助线是否比主要线条细？ |
| 符号一致 | 图上的字母与正文用的字母**完全一致**吗？有没有正文没有的点/线？（No20 图 2 教训） |
| 数值 | 图上出现的数字抄对了吗？**一般性结论的图里不该有具体数字**（No20 "48" 教训） |
| 与正文对应 | 这张图解释的是哪一段？读者一眼能对上吗？ |
| 颜色语义 | 每种颜色有含义吗（已知/辅助/结论）？还是随便配的？ |
| 风格一致 | 与本篇其他图同一套线宽、字号、配色吗？ |
| 必要性 | 这张图和已有图重复吗？（No20 例 3 图重复教训） |

发现问题**一次改完再重渲**，不要挑最省事的改。

### 6.2 红队复审

改完换身份再看一遍：你不再是作者，是想挑毛病的评审。作者视角会自动忽略自己刚"处理过"的地方。重点盯——
- 几何关系是否真的成立（不是"看起来像"）；
- 数值、符号有没有抄错（**这条比任何排版问题都严重**）；
- 最近一次修改引入的新问题（改 A 撞坏 B 是常态）。

### 6.3 交付前清单

- [ ] 渲染图已逐区看过（不是只看源码）
- [ ] `pdftotext` 确认图内文字层完整，无静默丢失
- [ ] 图上的点数/边数/度数/方向/染色与正文一一对应
- [ ] 图上符号与正文符号完全一致
- [ ] 一般性结论的图里没有多余的具体数值
- [ ] 与本篇其他图无重复，风格统一
- [ ] 中文说明在图注里，图内无中文

---

## 七、与写作流程的衔接

`euclids-gate-math-writing` 的「配图核查」一步调用本技能。写作时的分工：

- **写作阶段**：判断哪里有"可直观呈现的数学结构"必须配图，确定每张图要说明什么
- **作图阶段**：按本文第一节路由到对应工具，按第二节管线出图
- **核查阶段**：按第六节质检，未过不清算

图注写"图 N：描述"，描述里写明**对象 ↔ 点、关系 ↔ 线**的映射（例：黑点 = 组 1、白点 = 组 2、连线 = 认识）——Notation 不映射到图上，读者看不懂。
