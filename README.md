# exam-zh-exam-transcriber

An agent skill that transcribes photographed or scanned exam papers into compilable
[`exam-zh`](https://github.com/xiaoae/exam-zh) LaTeX, preserving the original question
order, numbering, answer blanks, tables and figures.

It is a layout-faithful transcriber, not a solver: no answers are invented, no questions
are rewritten, and every figure is redrawn in TikZ instead of being cropped from the photo.

## What it enforces

- One `question` environment per printed question; `textfigure` for text/figure wrapping.
- `enumerate` only for (1)(2)(3) sub-parts.
- `choices` for multiple-choice options, with the column count matching the scan.
- `\pa[]` for bracketed answers, `\fillin*[width=...][]` for blanks — left empty when the
  paper does not print an answer.
- TikZ redrawing of diagrams; no `\includegraphics` of the original photograph.
- A compile-and-look loop: `latexmk -xelatex`, then page-by-page render checks for
  overlaps, overflow, and orphaned blanks.

## Install

The file `SKILL.md` is the whole skill. Drop the directory into your agent's skills root:

```bash
git clone https://github.com/Lvshujun0918/exam-zh-exam-transcriber.git \
  ~/.agents/skills/exam-zh-exam-transcriber
```

Or copy it manually into `~/.claude/skills/`, `~/.codex/skills/`, or any other
`skills/<name>/SKILL.md` discovery path your harness reads.

## Requirements

- A TeX distribution with XeLaTeX, `latexmk`, and TikZ (`arrows.meta`, `shapes.geometric`).
- The project you are transcribing into already provides `exam-zh-new.cls` and the
  matching `exam-zh-*.sty` files, plus an older exam `.tex` to copy the preamble from.

## Usage

Point the agent at the page images and say what you want, for example:

- "把这些试卷照片还原成 exam-zh 的 LaTeX"
- "按原图排版，还原题号和配点"
- "图片用 TikZ 重绘"

Expected deliverables are the complete `.tex` document, the compiled PDF, and a render
check. If you only want the code, ask for the LaTeX block alone.

## Contents

```
SKILL.md   the skill instructions (triggers, hard rules, templates, workflow, checklist)
README.md  this file
```

## License

No license file is included; all rights reserved by the author. Open an issue if you want
a specific license applied.
