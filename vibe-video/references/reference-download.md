# 下载抖音参考视频

yt-dlp 直连抖音会 403（"Fresh cookies are needed"）；浏览器 cookie 提取在 Windows 新版 Chrome/Edge 上因 app-bound 加密 / DB 锁定失败。**可靠路径：无头浏览器自己拿匿名 cookie + 截获页面 API 响应拿 CDN 直链。**

## 1. 解析短链拿到视频 id

`https://v.douyin.com/XXXX/` 跟随重定向得到 `https://www.douyin.com/video/<id>`。yt-dlp 报错信息里也会给出 id。

## 2. playwright-core 截获 API（scratch 目录做，勿入正式工程）

```powershell
mkdir scratch-cookie; cd scratch-cookie
npm init -y; npm install playwright-core
```

脚本要点（完整实现见项目 scratch-cookie/get-video.js）：

- `chromium.launch({ executablePath: <ms-playwright chromium>, headless: true })`
- context：Windows Chrome UA、`locale: "zh-CN"`、`timezoneId: "Asia/Shanghai"`
- `page.on("response")`：URL 含 `aweme/v1/web/aweme/detail` 就存响应体
- `page.goto(视频页)` 后 `waitForTimeout(12000)` 等接口返回
- 同时落盘：`ctx.cookies()`（Netscape 格式备用）+ API 响应 JSON

## 3. 从 API 响应提取直链

`aweme_detail.video.bit_rate[]` 按 `bit_rate` 降序取第一档（normal_1080_0），用其 `play_addr.url_list[0]`：

```powershell
curl.exe -sS -L -A "<Chrome UA>" -H "Referer: https://www.douyin.com/" -o source.mp4 "<CDN url>"
ffprobe -v error -show_entries format=duration,size -show_entries stream=codec_name,codec_type,width,height source.mp4
```

期望：H.264 + AAC，1080p 横屏（本系列参考片是横屏 1920×1080，不是竖屏）。

## 4. 失败回退

- 无头被风控 → `headless: false` 有头跑一次（桌面会话在）
- CDN 链接过期（签名有时效）→ 重新跑截获脚本拿新链接
- `hypit download <url>` 可用 yt-dlp 的场景优先用它（`hypit download prepare` 需 uv + ffmpeg）
