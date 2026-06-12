# 省纸模式参考（旧方案：Chrome headless）

> ⚠ 此方案已废弃，标准方案见 SKILL.md "Rendering Markdown Before Printing"。
> Typst 方案排版质量更高、PDF 更小、不依赖浏览器。
> 保留此文件仅供参考。

## CSS 值（Chrome headless 专用）

```css
@page {
  size: A4;
  margin: 0.6cm 0.8cm 1.0cm 0.8cm;
}
body {
  font-family: "PingFang SC", "Microsoft YaHei", "Noto Sans CJK SC", sans-serif;
  margin: 0; padding: 0;
  line-height: 1.4; font-size: 11px; color: #111;
}
p { margin: 0 0 0.35em 0; text-align: justify; }
```

## 用户偏好

- 宋体五号（10.5pt），不要默认 sans-serif
- 页边距尽量小（0.5cm），页码右下角
- 段落间距 0.35em，不要空太多
- 不要页眉页脚
- 飞蚊症，偏好纸质阅读，纸要省
