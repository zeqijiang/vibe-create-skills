# hypit 项目骨架

## 目录布局

```
<vibe-video-project>/
├── package.json                  # {"hypit": {"project": true}, "type": "module"}
├── hypit.runtime.json            # 4 端点 + whisperx 绑定
├── assets/                       # 跨片共用（BGM 等）
├── references/<参考片>/          # source.mp4 + ANALYSIS + TIMELINE + evidence/
├── productions/<主题>/
│   ├── BRIEF.md  TREATMENT.md  PROGRESS.md
│   ├── authors/main.svml  authors/look.svs
│   ├── runs/main.svrun
│   ├── assets/narration/         # script.json + 分段 mp3
│   └── output/                   # 交付成片
├── packages/vibe-scenes/         # 组件包（见 component-package.md）
└── scratch-cookie/               # 下载/校验脚本（不进交付）
```

## package.json（项目根）

```json
{
  "name": "vibe-video-project", "version": "0.0.0", "private": true, "type": "module",
  "hypit": { "project": true },
  "dependencies": {
    "@fontsource-variable/noto-serif-sc": "^5.3.1",
    "@hypit/composition": "^0.1.0", "@hypit/html-program": "^0.1.0", "@hypit/hypit": "^0.3.1",
    "@hypit/media": "^0.1.0", "@hypit/spatial": "^0.1.0", "@hypit/temporal": "^0.1.0", "@hypit/timeline": "^0.1.0",
    "@vibe/scenes": "file:packages/vibe-scenes"
  },
  "devDependencies": { "typescript": "5.9.3" }
}
```

注意：中文字体用 **@fontsource-variable/noto-serif-sc**（静态包缺 wght.css 会报错）。本地包必须 `npm install` 成 file: 依赖才会被 loader 发现。

## hypit.runtime.json

```json
{
  "format": "hypit.runtime-local@1",
  "dataRoot": ".hypit/runtimes/local",
  "credentials": { "local": { "use": "@hypit/credential-store-local" } },
  "endpoints": {
    "hypihub.default": { "use": "@hypit/provider-hypihub", "config": { "baseUrl": "https://hypit.ai", "apiKey": { "store": "local", "key": "hypihub.oauth" } } },
    "media.local": { "use": "@hypit/media-local" },
    "html.local": { "use": "@hypit/provider-html-local", "config": { "chromePath": "<headless-shell 路径>" } },
    "image.opencv.local": { "use": "@hypit/provider-image-opencv-local" },
    "whisperx.local": { "use": "@hypit/provider-whisperx-local", "config": { "alignmentLanguages": ["zh"] } }
  },
  "bindings": { "@hypit/whisperx@1#whisperx-alignment": "whisperx.local" }
}
```

## 立项文档

- `BRIEF.md`：用户目标、参考链接、明确要求（BGM/主题/动画）、规格（对齐参考片：横屏 1080p、时长）、**费用授权**（默认零付费；任何付费生成前须确认）
- `TREATMENT.md`：章节表（对应参考片时间轴节奏：铭牌/文物/收束字幕）、视觉语法、声音方案
- `PROGRESS.md`：当前状态、Build id、下一步、坑

## 启动服务

```powershell
hypit runtime up                                # Build Worker
hypit programs up --endpoint whisperx.local     # 对齐服务（build 前确保 ready）
```
