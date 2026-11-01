# vibe-create-skills

自研 Agent 技能（skills）集合，每个子目录是一个可安装的 skill（含 `SKILL.md` + `references/`）。

## Skills

| Skill | 说明 |
| --- | --- |
| [`vibe-video`](./vibe-video/) | 制作抖音「Vibe知识大赏」风格的 AI 知识视频（暗金 MG 纪录片风），走本机 hypit **零付费**管线：参考片下载与风格分析 → 旁白 TTS → 组件化动画 → WhisperX 词级对齐 → 本地渲染成片。含国内网络全套绕墙方案（Chrome for Testing / HuggingFace / 抖音反爬）。 |

## 安装

把对应 skill 目录复制到你的 Agent 技能目录即可，例如 DeepSeek Harness：

```powershell
Copy-Item .\vibe-video "$env:USERPROFILE\.dsh\skills\vibe-video" -Recurse
```

（重启会话或等待技能目录刷新后生效。）

## 来源

`vibe-video` 由一次完整的生产实拍提炼：《运气的形状》风格对标 → 《问题的形状》（吉德林法则）成片，1080p 横屏 106.9s，全程零付费本地渲染（7 段 WhisperX 词级对齐 + 3206 帧浏览器光栅化 + 混音封装）。
