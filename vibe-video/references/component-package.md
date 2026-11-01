# 组件包 @vibe/scenes（htmlVisual 逐帧 MG）

用项目 Author Package 表达每个章节的"文物"。核心 API：`@hypit/html-program` 的 `htmlVisual({html, css, data, setup})` —— setup 是 `(root, data) => render(localFrame)` 函数体字符串，**逐帧调用、必须同步、确定性**（随机数用种子 PRNG，多 worker 才一致）。

## 包结构

```
packages/vibe-scenes/
├── package.json        # "hypit": {"activation": "./dist/activation.js"}，依赖 @hypit/* 域包 + @hypit/hypit
├── tsconfig.json       # NodeNext, strict, exactOptionalPropertyTypes, verbatimModuleSyntax
└── src/
    ├── kit.ts          # buildScene(def, args)：htmlVisual + text 子元素 + sealVisualTrack
    ├── scenes.ts       # 全部场景定义（SceneDef[]）
    ├── activation.ts   # manifest + producer + markup facets
    └── index.ts
```

改任何 TS 后必须 `npx tsc -p tsconfig.json` 重编译（activation 从 dist 加载）。

## kit.ts 关键实现

```ts
buildScene(def, args) {
  // data 注入 {w, h, frames, fps, params}；frames = window.span 帧数 → 编舞全部窗口相对
  // 根元素：kind:"program"，style 定位到 within.xPx/yPx/widthPx/heightPx
  // 文本子元素：kind:"text"，parent: 根id，fonts: args.font.faces（FontStackRef.faces）
}
```

**铁律**：
- text 元素带 exact fonts 时 style 禁止 font-weight/font-family/font-style（TYPE_REFINEMENT_REJECTED）
- 一个 Present 恰好一个根元素，其余元素全部 `parent: 根id`
- producer 里 args.id 用**实例 id**（从 options.sceneId 传），否则同 Surface 多实例 Track id 冲突
- htmlVisual 的 setup/data 只在非 undefined 时传入（exactOptionalPropertyTypes）

## 场景编舞模式（本包 9 个 Surface）

- **Stage**：星尘 canvas（130 颗种子星，缓慢漂移+闪烁）+ 暗角 + 水印，during=program.window
- **Tangle / FogList / Football / Routes / Typing**：SVG path 或 canvas 粒子；进度用 `P(a,b)=clamp((frame/T-a)/(b-a))` + easeOutCubic，各元素错峰（start 偏移）
- **TitleCard / LawCard / Plate**：字体排印为主；LawCard 打字机用 `el.textContent = full.slice(0, n)`
- **Typing**：光标闪烁 `(frame % 30) < 16`；雾粒子透明度绑定打字进度

CSS 注意：program 的 CSS 是 `@scope` 根样式（`:scope{...}`）；**禁止 CSS animation/transition**（捕获按帧步进，时间基动画不同步），一切运动在 setup 里按 frame 计算。

## activation.ts 骨架

- 单 manifest：producers = 每 Scene 一个 `render<Tag>`，inputs = timeline/within/window/(font)/options，output = compositionTypes.visualTrack
- `decodeFor(def, optionsType)`：assertAttributes → resolveTemporalContext/WindowReference → 字面量参数收进 options record（含 `sceneId: id`）→ sealGraphFragment（单 operation，export root = `{kind:"fragment-operation", operation: opId}`）
- declaration 的 `outputs` 必须包含 **visualTrack + optionsType**（漏了报 MARKUP_SURFACE_OUTPUT）
- facets：createProducerPackageFacet + createAdmissionPackageFacet + 每 Surface 一个 createMarkupSurfaceFacet
- hypitPackage：`{format:"hypit.package@1", modules:[{manifest}], facets}`（modules 要包 `{manifest}`）

## 在 main.svml 里用

```svml
<import as="vibe" from="@vibe/scenes@1"/>
<vibe:Plate id="plate-law" timeline={program.timeline} within={full} during={program.w-law}
  font={font-black} year="20世纪" place="底特律 · 通用汽车" src1="管理顾问" src2="查尔斯·吉德林"/>
```

`during` 支持全部时间窗形式；font 只在 needsFont 的 Surface 上传（否则 "does not accept font"）。
