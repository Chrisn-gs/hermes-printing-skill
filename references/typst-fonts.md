# Typst 字体管理

## 安装自定义字体

```bash
# 复制到系统字体目录
cp /path/to/font.ttf ~/Library/Fonts/

# 验证 Typst 能识别
typst fonts | grep "字体关键词"
```

Typst 会自动扫描 `~/Library/Fonts/` 下的 .ttf/.otf/.ttc 文件，无需重启或刷新。

## 使用方式

在 Typst 模板中：
```typst
#set text(font: "字体在typst fonts中显示的名字")
```

**注意**：字体名必须和 `typst fonts` 输出完全一致，不是文件名。例如文件名 `HYGuoTuChuangXinShanHaiJing-75U.ttf`，Typst 中名为 `HYGuoTuChuangXinShanHaiJing`（去掉后缀）。

## macOS 系统中文字体映射

| 中文名 | Typst 名 | 风格 |
|--------|---------|------|
| 宋体 | Songti SC / STSong | 衬线，正文首选 |
| 黑体 | PingFang SC / Heiti SC | 无衬线，标题 |
| 楷体 | Kaiti SC / STKaiti | 手写 |
| 仿宋 | STFangsong | 公文 |
| 苹方 | PingFang SC | macOS 默认 UI 字体 |

## 排查

- `typst fonts` 输出为空？→ 检查 macOS 字体册是否正常
- 字体名有空格？→ Typst 中用引号包裹 `"Font Name"`
- 中文显示方块？→ 字体可能不包含中文字形，换一个确认支持中文的字体
