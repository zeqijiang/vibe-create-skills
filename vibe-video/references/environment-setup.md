# 环境准备（Windows · 国内网络）

首次搭建或换机时按序执行。所有命令在视频项目根目录运行。

## 1. 基础工具

```powershell
node --version   # 需 24.x
npm --version
ffmpeg -version  # winget install Gyan.FFmpeg
uv --version     # pip install uv（hypit 托管 Python 程序依赖）
```

## 2. hypit CLI（npm 用户级 prefix）

npm 全局 prefix 若指向无写权限的目录（如 `D:\node-global` 只给 Users 读执行），会 EPERM。改为用户目录：

```powershell
npm config set prefix "C:\Users\admin\npm-global"
npm install --global @hypit/hypit
& "C:\Users\admin\npm-global\hypit.cmd" version
# 期望输出: @hypit/hypit 0.3.1 + Distribution/Launcher 路径
```

注意：npm 的 allow-scripts 会拦截 koffi/esbuild 的安装脚本，需 `npm approve-scripts koffi` / `esbuild` 后 `npm rebuild`。

## 3. 渲染浏览器（绕 Chrome for Testing 官方源被墙）

`hypit programs prepare --endpoint html.local` 默认从 googleapis 下载 Chrome for Testing——国内 0 字节卡死。**绕法**：把 Profile 的 `chromePath` 指向本机已有的 ms-playwright headless shell（`chrome.exe --version` 在 Windows 不回显会导致探测失败，**headless-shell 会正常回显**）：

```json
"html.local": {
  "use": "@hypit/provider-html-local",
  "config": {
    "chromePath": "C:/Users/admin/AppData/Local/ms-playwright/chromium_headless_shell-1234/chrome-headless-shell-win64/chrome-headless-shell.exe"
  }
}
```

验证：`hypit programs status --endpoint html.local` → ready。

## 4. WhisperX 本地对齐（绕 HuggingFace 被墙）

Profile 加端点 + 绑定：

```json
"whisperx.local": {
  "use": "@hypit/provider-whisperx-local",
  "config": { "alignmentLanguages": ["zh"] }
},
"bindings": {
  "@hypit/whisperx@1#whisperx-alignment": "whisperx.local"
}
```

准备顺序与坑：

```powershell
# ① uv 环境装依赖（frozen）；NLTK 句子数据若因系统代理被 SSRF 防护拒绝：
$env:NLTK_ALLOW_PROXIED_URLOPEN = "1"   # 仅对准备命令使用
# ② zh 对齐模型：huggingface.co 直连超时；hf-mirror.com 可达但 huggingface_hub 0.36
#    的元数据校验会拒镜像响应（FileMetadataError）→ 手动灌缓存：
```

手动灌缓存（wav2vec2 zh 对齐模型，sha 从 `curl -I https://hf-mirror.com/<repo>/resolve/main/preprocessor_config.json` 的 X-Repo-Commit 拿）：

```powershell
$sha = "<commit-sha>"
$base = "$env:USERPROFILE\.cache\huggingface\hub\models--jonatasgrosman--wav2vec2-large-xlsr-53-chinese-zh-cn"
New-Item -ItemType Directory -Force -Path "$base\snapshots\$sha", "$base\refs" | Out-Null
Set-Content "$base\refs\main" -Value $sha -NoNewline
foreach ($f in @("config.json","preprocessor_config.json","vocab.json","special_tokens_map.json","pytorch_model.bin")) {
  curl.exe -sS -L --retry 3 -C - -o "$base\snapshots\$sha\$f" "https://hf-mirror.com/jonatasgrosman/wav2vec2-large-xlsr-53-chinese-zh-cn/resolve/main/$f"
}
```

然后离线准备并启动服务：

```powershell
$env:HF_HUB_OFFLINE = "1"
hypit programs prepare --endpoint whisperx.local
hypit programs up --endpoint whisperx.local   # 127.0.0.1:8765
hypit programs status --endpoint whisperx.local  # → ready
```

ASR 模型（Systran/faster-whisper-small）通常已在默认 HF 缓存，无需下载。

## 5. 验证

```powershell
hypit doctor        # 0 diagnostics
hypit programs status  # html.local / media.local / image.opencv.local / whisperx.local 全 ready 或服务在线
```
