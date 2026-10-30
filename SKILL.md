---
name: exam-zh-exam-transcriber
description: Transcribe photographed or scanned exam papers into compilable exam-zh LaTeX while preserving the original question order, numbering, Japanese/Chinese wording, answer blanks, tables and figures. Use when asked to 识别试卷、把图片题目排版成 LaTeX、还原试卷版式、生成 exam-zh 试卷、或把题目图片转成 question/choices/fillin/textfigure 代码.
metadata:
  author: local
  version: "1.0.0"
---

# exam-zh 试卷转录

把试卷照片还原成可直接用 XeLaTeX 编译的 `exam-zh` 源码。目标是**版式与文字忠实还原**，不是解题，也不是改写题目。

## 何时使用

- 用户给出试卷图片、扫描件、PDF 截图，要求输出 `exam-zh` LaTeX。
- 用户要求“按原图排版”“还原题号/配点/答题栏”“图片用 TikZ 重绘”。
- 已有 `exam-zh-new.cls` 与 `exam-zh-*.sty` 的项目中新增一份试卷 `.tex`。

不适用：只想要题目文字摘要、只想要答案、或需要重新命题。

## 输入与输出契约

输入：一张或多张试卷页图片；同目录或邻近目录中的旧试卷 `.tex` 作为版式参考。
输出：一个完整 `.tex` 文件（或用户只要题目片段时，只给 LaTeX 代码块）。

默认交付：
1. 完整可编译文档（含与参考卷一致的导言区、标题、注意事项、配点表）。
2. 编译产生的 PDF。
3. 如用户明确要求“只返回 latex 代码块”，则只给代码块，不解释、不确认答案。

## 硬性规则

1. **一题一个 `question` 环境**。需要图文绕排时用 `textfigure` 包住一个 `question`。
2. 题干中的 (1)(2)(3) 小题才允许用 `enumerate`；其他场景禁止 `enumerate`。
3. 禁止 `problem` 环境。
4. 选择题选项必须用 `choices`，`columns` 按原图每行选项数设置。
5. 括号答案用 `\pa[答案]`；图片没有给出答案时用 `\pa[]`（空括号），不要猜答案。
6. 填空用 `\fillin*[width=...][答案]`；没有答案时第二参数留空 `[]`。
7. 不大题套大题：`\section` 只放该大题的总说明，具体小问放进各自的 `question`。
8. 忠实保留原文（含日文、错别字、缺失分号、印刷体寄存器名），不修正题目，不补解题过程。
9. 图片一律用 TikZ 重绘；能用图文绕排就不要把图堆在题目下方。
10. 题号 `index` 全卷连续；`points` 只在原卷标了分值或该大题独立成题时填写。

## 版式模板

### 选择题

```latex
\begin{question}[index=1, points=2]
    题干文字\pa[]
    \begin{choices}[columns=2]
        \item 选项A
        \item 选项B
        \item 选项C
        \item 选项D
    \end{choices}
\end{question}
```

### 填空题 / 数值题

```latex
\begin{question}[index=2, points=2]
    16進数 \texttt{4A} を8ビットの2進数に直せ。\fillin*[width=6em][]
\end{question}
```

### 大题（含小问）

```latex
\begin{question}[index=10, points=9]
    题干文字。
    \begin{enumerate}
        \item 小问一\fillin*[width=5em][]
        \item 小问二\fillin*[width=5em][]
        \item 小问三\fillin*[width=12em][]
    \end{enumerate}
\end{question}
```

### 图文绕排

```latex
\textfigure[fig-pos=right-top, text-width=0.78\linewidth]{
    \begin{question}[index=16, points=21]
        题干文字。
    \end{question}
}{
    \begin{tikzpicture}
        % 重绘的图
    \end{tikzpicture}
}
```

### 图 + 表并排（不适合绕排时）

```latex
\begin{question}[index=16, points=21]
    题干文字。

    \noindent
    \begin{minipage}[t]{0.42\linewidth}
        \centering
        \begin{tikzpicture}...\end{tikzpicture}
    \end{minipage}\hfill
    \begin{minipage}[t]{0.56\linewidth}
        \centering\footnotesize
        \begin{tabular}{...}...\end{tabular}
    \end{minipage}
\end{question}
```

### 代码 / 汇编清单

```latex
\begin{lstlisting}[basicstyle=\ttfamily\scriptsize,numbers=none]
Loop: slt  $t0, $s2, 2
      bne  $s3, $s4, Else
Exit:
\end{lstlisting}
```

- 原文行号不是题目内容时关闭 `numbers`。
- 两个清单并排时，minipage 宽度合计不要超过 `0.97\linewidth`，并显式留 `\hfill`。
- 空栏用 `(1)`、`(2)` 原样保留；答题表格用 `tabular`，不要把 `\fillin` 放进表格单元格（会触发 `Missing \endgroup`），表格内改用 `\underline{\hspace{4em}}`。

## 工作流程

1. **读全卷**：先按文件名/页序浏览所有图片，确认页数、题号、配点表、是否有缺页。
2. **建题目地图**：列出 `大题 → 小问 → 题号 → 分值 → 是否需要图`。先定编号，再写正文。
3. **放大核对**：对寄存器名、下标、符号、分数线、表格数字做局部裁切放大。照片模糊时优先核对：
   - `$s0` / `$a0`、`$s2` / `$a2` 等形近字符；
   - `2×10^4`、`10.20GHz`、`-0.8125_{10}` 等数字；
   - 表格中哪个格子对应 (1)(2)(3)。
4. **搭文档**：复制参考卷导言区 → 改年份/标题 → 放配点表 → 按题目地图写 `question`。
5. **重绘插图**：用 TikZ 的矩形、梯形、椭圆、箭头重画；保持原图相对位置与标签文字。需要的库：`arrows.meta`、`shapes.geometric`。
6. **编译**：`latexmk -xelatex -interaction=nonstopmode -halt-on-error -outdir=build <file>.tex`。
7. **看渲染**：`pdftoppm -r 130 -png` 后逐页检查，重点看：
   - 清单是否越界/重叠；
   - 图文绕排是否压字；
   - 表格是否超出版心；
   - 填空是否孤行；
   - 题号、分值、页脚是否正确。
8. **只修版式，不改题目**。改完重新编译并再看受影响页面。

## 常见错误

- 在 `question` 外用 `enumerate` 列小问 → 改成多个 `question` 或一个 `question` 内的 `enumerate`。
- 一个 `question` 里塞多个独立题 → 按原卷题号拆开。
- 表格单元格里用 `\fillin` → 改 `\underline{\hspace{...}}`。
- 忘记 `\usetikzlibrary{arrows.meta,shapes.geometric}` → `Unknown arrow tip kind 'Stealth'` / `I do not know the key '/tikz/ellipse'`。
- 汇编清单保留默认行号 → 数字跑到框外，压住相邻清单；加 `numbers=none`。
- 两个 minipage 无间隙 → 左右清单边框相贴；宽度留 3% 余量并加 `\hfill`。
- 猜测未印出的答案 → 一律留空。

## 完成检查表

- [ ] 题号连续且与原卷大题结构对应。
- [ ] 每个 `question` 只有一个主题；小问只用 `enumerate`。
- [ ] 所有括号答案用 `\pa`，所有填空用 `\fillin`（表格内除外）。
- [ ] 图片已重绘，没有 `\includegraphics` 原图。
- [ ] `latexmk` 退出码为 0。
- [ ] 逐页看过渲染，无重叠、越界、孤行。
- [ ] 没有擅自添加答案、解析或“修正”后的题干。

<!-- OB-S:d4WiZnPTS-2OEpiEpMDjzA -->
