# Hermes Printing Skill

从终端打印 Markdown 文件的 Hermes Agent skill。支持网络打印机管理、Markdown → PDF 渲染、字体选择、页数控制。

## 功能

- 🖨️ 网络打印机管理（添加、删除、状态查看）
- 📄 Markdown 渲染后打印（Typst 排版，不是打原始文本）
- 🔤 中文字体支持（宋体、黑体、楷体、仿宋 + 自定义字体）
- 📐 A4 纸最小页边距，页码右下角
- 📑 页数控制（提取指定页打印）

## 依赖

```bash
brew install pandoc typst
pip3 install pypdf   # 页数检查和提取
```

## 安装

### 方式一：Hermes skills 安装

```bash
hermes skills install <skill-id>
```

### 方式二：手动安装

```bash
# 克隆到 Hermes skills 目录
git clone https://github.com/Chrisn-gs/hermes-printing-skill.git \
  ~/.hermes/skills/productivity/printing
```

### 方式三：OneDrive 同步

把整个文件夹放到 OneDrive 同步的目录，在其他机器上复制到 `~/.hermes/skills/productivity/printing/`。

## 使用

### 打印 Markdown 文件

Hermes Agent 会自动识别打印意图。用户说"打印这个文件"时，skill 会：

1. 去掉 YAML frontmatter
2. Pandoc 转 Typst 格式
3. Typst 编译为 PDF（宋体五号，最小页边距）
4. 发送到打印机

### 流程

```
Markdown → Pandoc → Typst → PDF → lp 打印
```

### 字体

默认字体：**Songti SC**（宋体），中文五号字（10.5pt）。

系统已安装的中文字体：

| 字体 | Typst 名称 | 风格 |
|------|-----------|------|
| 宋体 | Songti SC | 衬线，正式 |
| 黑体 | PingFang SC | 无衬线，现代 |
| 楷体 | Kaiti SC | 手写风格 |
| 仿宋 | STFangsong | 仿宋，公文 |
| 等宽 | Menlo | 代码 |

安装自定义字体：

```bash
cp your-font.ttf ~/Library/Fonts/
# Typst 自动识别，无需额外配置
typst fonts | grep -i "你的字体名"
```

### 打印命令

```bash
# 默认打印机（HP115W）
lp -d HP115W output.pdf

# 查看队列
lpstat -o HP115W

# 取消任务
cancel HP115W-N

# 仅打印第1页
python3 -c "
from pypdf import PdfReader, PdfWriter
r = PdfReader('output.pdf')
w = PdfWriter()
w.add_page(r.pages[0])
w.write('page1.pdf')
"
lp -d HP115W page1.pdf
```

### 添加网络打印机

```bash
# 1. 测试连通
ping -c 3 <printer-ip>

# 2. 探测端口（631=IPP, 9100=JetDirect, 515=LPD）
nc -z -w2 <printer-ip> 631 && echo "IPP 可用"

# 3. 查询打印机能力
ipptool -tv ipp://<printer-ip>:631/ipp/print get-printer-attributes.test

# 4. 添加（macOS 不需要 sudo）
lpadmin -p MyPrinter -E \
  -v "ipp://<printer-ip>:631/ipp/print" \
  -m everywhere \
  -D "我的打印机"

# 5. 设为默认
lpoptions -d MyPrinter
```

## Typst 模板参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| font | Songti SC | 字体 |
| size | 10.5pt | 字号（中文五号） |
| stroke | 0.02pt | 笔画加粗 |
| margin | 0.5cm | 页边距（打印机物理限制） |
| numbering | "1" | 页码，右下角 |
| leading | 0.65em | 行间距 |
| justify | true | 两端对齐 |

## 目录结构

```
printing/
├── SKILL.md                    # Hermes skill 定义
├── README.md                   # 本文件
└── references/
    ├── compact-css.md          # 旧方案参考（Chrome headless，已废弃）
    ├── hp115w-printer.md       # HP115W 打印机配置参考
    └── typst-fonts.md          # macOS 可用字体列表
```

## Pitfalls

1. **不要打原始 .md 文件** — `#`、`|`、`*` 会原样输出，必须渲染为 PDF
2. **macOS lpadmin 不需要 sudo** — 跟 Linux 不一样
3. **HP115W 卡纸后先取消旧任务再重发** — 否则会重复打印
4. **WSD 地址在 macOS 无用** — 只用 IP，端口用 631（IPP）
5. **`-m everywhere` 是现代默认** — 不需要找型号 PPD 文件

## License

MIT
