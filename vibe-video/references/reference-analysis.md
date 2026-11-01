# 参考片风格分析

目标：把"为什么好看"变成可迁移的设计语法，沉淀进 `references/<片名>/ANALYSIS.md` + `TIMELINE.md`。工具全走 hypit 本地 media 能力（ffmpeg），不需要凭据。

## 命令

```powershell
hypit media probe references/<片名>/source.mp4          # 时长/分辨率/帧率/音轨
hypit media boundaries references/<片名>/source.mp4     # 视觉切换点（判断快剪还是连续动画）
hypit media tiles references/<片名>/source.mp4 --start 0 --end 129 --every 2 `
  --columns 4 --rows 4 --cell 480 --to references/<片名>/evidence/overview
```

注意：`--end` 不要取到时长整值（130.0 会报 "no video frame exists at or after 130 s"），用 129。

然后**用 read_image 逐张读网格**，边读边记录。

## ANALYSIS.md 要回答

1. 这部片子**是什么**（形式：纯 MG / 实拍 / 混合；有无真人；署名与参考文献）
2. **为什么成立**：贯穿的问题、结构骨架（如编年史章节）、单场景信息密度、核心视觉修辞（如"同一文物连续形变穿越时代"）、美术克制点（配色/字体/装饰系统）
3. **持久系统**：跨章节常驻元素（章节铭牌、文献卡、字幕系统、背景、水印）各自的内容与行为
4. **声音**：BGM、有无旁白（字幕句式推断）
5. **可迁移资产**：哪些能 1:1 复用到新主题

## TIMELINE.md 要回答

按源时间分段（如 `## 18–27s · 1494 帕乔利`），每段记：铭牌内容、文物（视觉主角）及其入场/行为/退场、字幕原文与金色关键词、MG 数据动画（计数器/进度条/印章）的节奏。写到"能照着实现"的程度。

## 产出

- ANALYSIS.md / TIMELINE.md 写入 `references/<片名>/`
- 概览网格留在 `evidence/`，正文引用其路径
- 转写（可选）：`hypit transcribe <file> --language zh --to transcript.json`，需要 whisperx.local 服务在线
