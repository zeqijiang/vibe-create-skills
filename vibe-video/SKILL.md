---
name: vibe-video
description: 制作抖音「Vibe知识大赏」风格的 AI 知识视频（暗金 MG 纪录片风），走本机 hypit 零付费管线：参考片下载与风格分析 → 旁白 TTS → 组件化动画 → WhisperX 词级对齐 → 本地渲染成片。Use when the user asks to create a knowledge video (知识视频/vibe知识大赏/抖音知识号), adapt a reference video's style to a new topic, or run the local hypit video pipeline on Windows.
---

# Vibe Video — 知识大赏风格视频生产

你是这条片子的导演兼制片人。目标：以**零付费全本地管线**产出一条 1080p 横屏、约 2 分钟、带女声旁白 + 词级对齐字幕 + MG 动画的知识视频，风格对标「Vibe知识大赏」系列。

## 前提

本 skill 假设（首次运行按 [environment-setup](references/environment-setup.md) 准备）：

- hypit CLI 0.3.1：`C:\Users\admin\npm-global\hypit.cmd`（npm 用户级 prefix）
- 渲染浏览器：ms-playwright headless shell（经 chromePath 注入，Chrome for Testing 官方源被墙）
- WhisperX 本地对齐服务（127.0.0.1:8765，zh 对齐模型已入 HF 缓存）
- ffmpeg/ffprobe/uv/node 24 在 PATH

## 流程总览

```
① 参考与立项 → ② 环境就绪 → ③ 旁白与文案 → ④ 组件与装配 → ⑤ 构建 → ⑥ 验片交付
```

| 步骤 | 做什么 | 读 |
| --- | --- | --- |
| ① 拿到参考片、风格分析、写 Brief/Treatment | 下载抖音参考片、抽帧分析、沉淀 ANALYSIS/TIMELINE | [reference-download](references/reference-download.md) · [reference-analysis](references/reference-analysis.md) |
| ② hypit 项目骨架、4 端点 Profile、服务启动 | runtime init、programs prepare/up、国内网络三处绕墙 | [environment-setup](references/environment-setup.md) · [project-setup](references/project-setup.md) |
| ③ edge-tts 分段旁白 + Script 文案（Dual Text 处理数字） | script.json → mp3 → 全片时长核算 | [narration](references/narration.md) |
| ④ 组件包 @vibe/scenes（htmlVisual 逐帧 MG）+ 主装配 main.svml | 9 场景模式、SVML 接线、全部类型坑 | [component-package](references/component-package.md) · [authoring](references/authoring.md) |
| ⑤ check → plan → build --follow | 零付费确认、卡死自救、大堆参数 | [build-and-qa](references/build-and-qa.md) |
| ⑥ ffprobe + 抽帧网格 + 音量检测验片 | 逐帧核对、字幕交接检查、交付 | [build-and-qa](references/build-and-qa.md) |

## 硬性规则（踩过的坑，违反必炸）

1. **先 `hypit check` 再 build**；`hypit plan` 必须显示 `Local requests 33/33, no Provider charge` 才允许提交——出现任何付费请求先停下问用户。
2. **Build 卡在 "Saving Result"**：33/33 完成后归档可能挂起（大值 JSON）。成片 mp4 一定完整落在 `results/<date>/<build>/files/file-0009.mp4`，直接取出，不要反复重试 finish。
3. **exact fonts 的 text 元素禁止 font-weight/font-family 样式**（字重由 Face 决定）；**一个 Present 只允许一个根元素**，子元素必须带 `parent`；**多个同 Surface 实例的 Track id 必须唯一**（用实例 id 不是 Surface 名）。
4. **字幕防叠画**：caption recipe `lead-frames: 0; tail-frames: 0`——outgoing 拖尾 + incoming 提前入场会在每个 Cue 交接叠画。
5. **文档头要对版本**：run 文件 `<?svml using="@hypit/markup/run@1"?>`，配方 `@hypit/recipe@1`（不是 svs）；tag 前缀跟 import 别名走。
6. **Done means watched**：交付前必须 ffprobe + 抽帧看图 + volumedetect 听混音，交接段抽 10fps 网格查叠画。

## 项目记忆

项目文件就是记忆：`BRIEF.md`（用户目标/费用授权）、`TREATMENT.md`（导演答案）、`PROGRESS.md`（当前状态/下一步）、`references/<片名>/ANALYSIS.md`+`TIMELINE.md`（参考理解）。每次会话先读这些再动手；结论变化就重写对应文件。
