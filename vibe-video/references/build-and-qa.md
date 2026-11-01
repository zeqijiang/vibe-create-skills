# 构建、自救与验片

## 提交前

```powershell
hypit check productions/<主题>/runs/main.svrun     # 图合法（类型/导入/输出全查）
hypit plan productions/<主题>/runs/main.svrun      # 必须显示 Local requests N/N，无 Provider charge
```

plan 里应看到：whisperx-alignment ×段数（local, no Provider charge）、normalize-media、rasterize-visual、render-timeline-audio、mux-program-media。任何付费请求 → 停下与用户确认账号/预算。

确保服务在线：`hypit runtime up` + `hypit programs up --endpoint whisperx.local`。

## 构建

```powershell
hypit build productions/<主题>/runs/main.svrun --follow
```

节奏参考（106s 成片、3206 帧）：对齐 ~1 分钟 → 光栅化 ~1.5 分钟 → 编码+混音 ~1 分钟，全程 5 分钟上下。

## 归档卡死自救（高频）

33/33 完成后可能停在 "Saving Result"（worker 序列化 ~90MB 的 composition/TimelineVisual 值）：

1. **先找成片**：`results/<date>/<build>/files/file-0009.mp4`（编号可能变，取 `.mp4`）——执行阶段已完成编码，文件完整。
2. `ffprobe` 验证时长/分辨率/双流后再取用。
3. 簿记修复（可选）：`$env:NODE_OPTIONS="--max-old-space-size=12288"` 后 `hypit result finish <build-id> --runtime <profile>`；若报 "owned by another process"，`hypit runtime down` → `runtime up`（提示会变为 interrupted）→ 再 finish。**不要为了簿记反复重跑 build。**

## 验片（Done means watched）

```powershell
ffprobe -v error -show_entries format=duration -show_entries stream=codec_name,codec_type,width,height <mp4>
# 抽关键帧逐张看（钩子/片头/各章节/尾卡）
ffmpeg -y -ss <t> -i <mp4> -frames:v 1 f<t>.jpg
# 字幕交接检查：交接时刻 ±0.65s、10fps 拼网格，确认无叠画
ffmpeg -y -ss 49.4 -t 1.3 -i <mp4> -vf "fps=10,scale=640:-1,tile=7x2" -frames:v 1 boundary.jpg
# 混音检查：旁白段 / BGM 间隙 / 尾部 三处
ffmpeg -ss 20 -t 2 -i <mp4> -af volumedetect -f null - 2>&1 | Select-String "mean_volume|max_volume"
```

期望：旁白段 max ≈ -9dB，BGM 床 mean ≈ -24dB（低于旁白 4–6dB），尾部渐弱。

## 交付

复制成片到 `productions/<主题>/output/<片名>.mp4`，present 给用户；更新 PROGRESS.md（Build id、验片结论、遗留瑕疵）。

## 已知非阻塞瑕疵示例

- 大值 JSON 导致 result 簿记挂起（见上）
- 场景相位与旁白微错位（如航线图红弧早退）→ 改场景编舞的 P(a,b) 窗口重跑
- 竖屏版：Canvas 1080×1920 + 布局参数重排（组件已按 within Frame 定位，理论可复用）
