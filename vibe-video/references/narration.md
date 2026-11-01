# 旁白（TTS）与文案

## 1. 写分段文案 script.json

按 Treatment 章节拆段，每段一个 id。中文约 4.2–4.6 字/秒，全片语音时长 ≈ 字数/4.5；加章节间隙 1.3–1.7s/处 + 片头 1.5s + 尾部 4s，估算总时长。

```json
{
  "voice": "zh-CN-XiaoxiaoNeural",
  "rate": "-8%",
  "segments": [
    { "id": "hook", "text": "遇到难题，你脑子里是什么样子？大多数时候，它不是难题，它是一团乱麻。" }
  ]
}
```

女声选型：`XiaoxiaoNeural`（温暖圆润，纪录片感）；备选 `XiaoyiNeural`（明快）。男声 `YunyangNeural`（正式沉稳）。

## 2. 生成

```powershell
$cfg = Get-Content script.json -Raw | ConvertFrom-Json
foreach ($seg in $cfg.segments) {
  edge-tts --voice $cfg.voice "--rate=$($cfg.rate)" --text $seg.text --write-media "$($seg.id).mp3"
  ffprobe -v error -show_entries format=duration -of csv=p=0 "$($seg.id).mp3"   # 记录每段时长
}
```

坑：`--rate -8%` 会被 argparse 当旗标，必须写 `--rate=-8%`。

## 3. Script 文案必须与音频朗读一致（词级对齐的根）

WhisperX 是**强制对齐**：把 Script 的词对到音频上。所以 Script 的 spoken 侧必须等于 TTS 实际读出的音：

- **数字**：TTS 把「1980年代」读成"一九八零年代"，对齐模型对数字 token 无力 → 用 Dual Text：`<1980年代|一九八零年代>的英国`
- **字母**：`<AI|A I>`（TTS 读字母名）
- **标点**：随前字，不产生独立 token
- 口语化改写时**先改 script.json 重新生成音频**，再改 SVML，两边永远同步

## 4. SVML Script 语法速记

- Segment：`<hook> ... </hook>`；Cue 断句用 `||`（中文一句 ≤18 字）
- Moment：`@{name!}`（下一词起点）；Selection：`@{name} ... @{/name}`
- 关键词属性：`word{keyword}`（caption family 解释）
- 本片只用 Moment 给语义钩子，MG 编舞走窗口相对时间，不逐词绑
