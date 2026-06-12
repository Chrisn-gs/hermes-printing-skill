---
name: printing
description: "Network printer setup and file printing from macOS/Linux. Use when user wants to add a printer, print files from CLI, or render Markdown/HTML to paper."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [printing, cups, ipp, network-printer, macos, lpadmin]
    related_skills: []
---

# Printing

## When to Use

- User wants to add a network printer
- User wants to print files from the terminal
- User wants to render Markdown/HTML properly before printing
- User asks about CUPS, IPP, lpadmin, or printer drivers
- Printer troubleshooting (queue stuck, jobs not printing)

Don't use for:
- PDF generation without printing intent
- 3D printing

## Adding a Network Printer (macOS)

### Step 1: Verify network connectivity

```bash
ping -c 3 <printer-ip>
```

### Step 2: Probe open ports

```bash
for port in 631 9100 515 80; do
  nc -z -w2 <printer-ip> $port && echo "port $port open" || echo "port $port closed"
done
```

Key ports:
- **631** = IPP (Internet Printing Protocol) — preferred, modern
- **9100** = JetDirect/PDL — raw printing, HP standard
- **515** = LPD — old Unix protocol
- **80** = HTTP — printer web management UI

### Step 3: Query printer capabilities via IPP

```bash
ipptool -tv ipp://<printer-ip>:631/ipp/print get-printer-attributes.test
```

Returns: supported media sizes, resolution, color modes, duplex, etc. If this works, IPP is confirmed and the `everywhere` driver will work.

### Step 4: Add printer with lpadmin

```bash
lpadmin -p <queue-name> \
  -E \
  -v "ipp://<printer-ip>:631/ipp/print" \
  -m everywhere \
  -D "<display-name>" \
  -L "<location>"
```

- `-m everywhere` = IPP Everywhere driver. No PPD file needed — queries printer capabilities via IPP. Works for most modern printers.
- `-E` = enable the printer
- macOS `lpadmin` does NOT need sudo (unlike Linux)

### Step 5: Set as default

```bash
lpoptions -d <queue-name>
```

### Step 6: Verify

```bash
lpstat -p <queue-name>    # status
lpstat -v <queue-name>    # device URI
```

## Printing Files

```bash
lp -d <queue-name> <file>
```

Check queue:
```bash
lpstat -o <queue-name>
```

Cancel job:
```bash
cancel <job-id>   # e.g. cancel HP115W-3
```

## Rendering Markdown Before Printing（标准方案：Typst）

**Never print raw .md files** — tables, headings, code blocks all render as plain text with `#`, `|`, `*` visible.

标准流程：Markdown → Pandoc → Typst → PDF → lp 打印。

依赖：`brew install pandoc typst`（已装）

### 标准 Typst 模板（用户验证版）

用户偏好：宋体五号、最小页边距、页码右下角、笔画加粗（stroke 0.02pt）。

```bash
# 完整流程
INPUT="$1"                  # Markdown 文件路径
PRINTER="${2:-HP115W}"      # 打印机名，默认 HP115W
TMP=$(mktemp -d)

# 1. 去掉 YAML frontmatter，转为 Typst
sed '/^---$/,/^---$/d' "$INPUT" | pandoc -f markdown -t typst -o "$TMP/content.typ"

# 2. 写 Typst 模板
cat > "$TMP/template.typ" << 'TYPST'
#set page(
  paper: "a4",
  margin: (top: 0.5cm, bottom: 0.5cm, left: 0.5cm, right: 0.5cm),
  numbering: "1",
  number-align: right,
)
#set text(font: "Songti SC", size: 10.5pt, lang: "zh", stroke: 0.02pt)
#set par(justify: true, leading: 0.65em)

#show heading.where(level: 1): it => {
  set text(size: 14pt, weight: "bold")
  block(above: 1em, below: 0.5em, it.body)
}
#show heading.where(level: 2): it => {
  set text(size: 12pt, weight: "bold")
  block(above: 0.8em, below: 0.4em, it.body)
}
#show heading.where(level: 3): it => {
  set text(size: 11pt, weight: "bold")
  block(above: 0.6em, below: 0.3em, it.body)
}
#show raw: set text(font: "Menlo", size: 9pt)
#show link: underline

#include "content.typ"
TYPST

# 3. 编译 + 打印
typst compile "$TMP/template.typ" "$TMP/output.pdf" && lp -d "$PRINTER" "$TMP/output.pdf"
rm -rf "$TMP"
```

### 模板参数说明

| 参数 | 值 | 说明 |
|------|-----|------|
| font | Songti SC | 宋体，macOS 内置 |
| size | 10.5pt | 中文五号字 |
| stroke | 0.02pt | 加粗笔画，Chrome 默认渲染更粗，此值补偿 |
| margin | 0.5cm | 打印机物理限制，0 会裁切 |
| numbering | "1" | 页码，右下角 |
| leading | 0.65em | 行间距 |
| justify | true | 两端对齐 |

### macOS 可用中文字体（typst fonts 验证）

| 字体名 | Typst font 名 | 说明 |
|--------|-------------|------|
| 宋体 | Songti SC / STSong | 默认正文字体 |
| 黑体 | PingFang SC | 无衬线，标题常用 |
| 仿宋 | STFangsong | 公文用 |
| 楷体 | Kaiti SC | 手写风格 |
| 华文黑体 | Heiti SC | 粗黑 |
| 兰亭黑 | Lantinghei SC | 思源系黑体 |
| 艺术宋 | HYGuoTuChuangXinShanHaiJing | 用户安装的创意字体 |
| 等宽 | Menlo / Hack | 代码 |

完整列表：`typst fonts | grep -i "SC\|TC\|Song\|Hei\|Kai\|Fang"`

**安装自定义字体**：将 .ttf/.otf 复制到 `~/Library/Fonts/`，Typst 自动识别，无需重启。用 `typst fonts | grep 关键词` 确认字体名，直接在模板 `#set text(font: "字体名")` 中使用。

**不可用**（需额外安装）：Noto Serif CJK SC、Fira Mono、STIX Two Text、FandolKai

字体安装和排查详见 `references/typst-fonts.md`。

### mdprint 一行命令

```bash
mdprint() {
  local input="$1"
  local printer="${2:-HP115W}"
  local tmp=$(mktemp -d)
  sed '/^---$/,/^---$/d' "$input" | pandoc -f markdown -t typst -o "$tmp/content.typ"
  cat > "$tmp/template.typ" << 'TYPST'
#set page(paper: "a4", margin: (top: 0.5cm, bottom: 0.5cm, left: 0.5cm, right: 0.5cm), numbering: "1", number-align: right)
#set text(font: "Songti SC", size: 10.5pt, lang: "zh", stroke: 0.02pt)
#set par(justify: true, leading: 0.65em)
#show heading.where(level: 1): it => { set text(size: 14pt, weight: "bold"); block(above: 1em, below: 0.5em, it.body) }
#show heading.where(level: 2): it => { set text(size: 12pt, weight: "bold"); block(above: 0.8em, below: 0.4em, it.body) }
#show heading.where(level: 3): it => { set text(size: 11pt, weight: "bold"); block(above: 0.6em, below: 0.3em, it.body) }
#show raw: set text(font: "Menlo", size: 9pt)
#show link: underline
#include "content.typ"
TYPST
  typst compile "$tmp/template.typ" "$tmp/output.pdf" && lp -d "$printer" "$tmp/output.pdf"
  echo "已发送到 $printer"
  rm -rf "$tmp"
}
```

## WSD vs IPP

Windows uses WSD (Web Services for Devices) protocol — macOS/Linux don't support it. But printers that expose WSD almost always also expose IPP on port 631. The user may only know the WSD address (e.g. `192.168.x.x:8018/wsd`). Ignore the port/path and probe port 631 directly.

## Pitfalls

1. **macOS `lpadmin` does not need sudo.** Unlike Linux CUPS, macOS allows user-level printer management. Don't ask for sudo password unnecessarily.

2. **WSD addresses are useless on macOS.** If user gives a WSD address like `192.168.5.17:8018/wsd`, the IP is useful but port 8018 and `/wsd` path are Windows-only. Probe port 631 (IPP) instead.

3. **`-m everywhere` is the modern default.** Don't hunt for model-specific PPD files unless `everywhere` fails. IPP Everywhere queries the printer's capabilities at setup time, so it adapts automatically.

4. **`lp` prints raw text files as-is.** Markdown files will print with `#`, `|`, `*` visible. Always render to PDF first for formatted documents.

5. **Chrome headless may hang on first run.** If it times out, check if the PDF was actually created (`ls -la /tmp/print-output.pdf`). Chrome sometimes completes the PDF but doesn't exit cleanly.

6. **WeasyPrint needs system libraries (pango, gobject).** Don't try `pip install weasyprint` without `brew install pango` first — it will fail with `cannot load library 'libgobject-2.0-0'`. The Chrome headless approach avoids this dependency entirely.

6. **CUPS `lp` rejects HTML with `text/html` error.** The IPP Everywhere driver doesn't include an HTML filter. Convert to PDF before printing.

7. **HP115W 卡纸后不要直接重发任务。** 旧任务还在队列里，重发会导致重复打印（用户实际打了 20 张而非 10 张）。正确流程：先 `cancel HP115W-N` 取消旧任务，确认队列清空（`lpstat -o HP115W`），再 `lp` 重新发送。

8. **用户要求"只打N页"时，先检查页数再提取。** Typst 编译后用 `pypdf` 检查页数（`pip install pypdf`），超过目标页数时用 pypdf 提取指定页面再打印：
```python
from pypdf import PdfReader, PdfWriter
r = PdfReader('output.pdf')
w = PdfWriter()
w.add_page(r.pages[0])  # 只取第1页
w.write('page1.pdf')
```
然后 `lp -d HP115W page1.pdf`。不要直接打多页再告诉用户"只打了第1页"——打印机不支持按页打印参数。
