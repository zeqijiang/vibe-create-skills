# 主装配 main.svml

纯音频 A-roll + MG 画面的完整接线。文档头 `<?svml using="@hypit/markup@1"?>`。

## imports（tag 前缀 = 别名）

```svml
<import as="time" from="@hypit/timeline-author@1"/>
<import as="space" from="@hypit/spatial@1"/>
<import as="media" from="@hypit/media@1"/>            <!-- asset:Audio / FontStack -->
<import as="fonts" from="@hypit/fontsource@1"/>
<import as="mediaop" from="@hypit/media-operations@1"/>
<import as="whisperx" from="@hypit/whisperx@1"/>
<import as="semantic" from="@hypit/narrative-temporal@1"/>
<import as="ncap" from="@hypit/narrative-caption@1"/>
<import as="cap" from="@hypit/caption-fine@1"/>
<import as="audio" from="@hypit/audio-track@1"/>
<import as="vibe" from="@vibe/scenes@1"/>
<import as="film" from="@hypit/film@1"/>
<import as="html" from="@hypit/html-video@1"/>
<import from="@hypit/script@1"/>                      <!-- 别忘了，否则 <script> unknown -->
<import as="look" source="./look.svs"/>
```

## 接线链（顺序即依赖）

```svml
<script id="story"> 7 个 Segment，Dual Text 处理数字/字母，|| 断 Cue，Moment 埋语义点 </script>
<space:Canvas id="canvas" width="1920" height="1080"/>
<time:Clock id="clock" frame-rate="30"/>
<space:Frame id="full" within={canvas.bounds} left="0%" top="0%" right="100%" bottom="100%"/>
<fonts:Face id="serif-face" package="@fontsource-variable/noto-serif-sc" weight="600" style="normal"/>
<fonts:Face id="serif-black-face" ... weight="900" .../>
<media:FontStack id="font" primary={serif-face}/>  <media:FontStack id="font-black" .../>
<media:Audio id="narr-<seg>" src="../assets/narration/<seg>.mp3"/> ×7 + <media:Audio id="bgm" src="../../../assets/bgm.mp3"/>
<mediaop:Normalize id="<seg>-media" source={narr-<seg>} clock={clock} video="none" audio="default" span-authority="audio"/> ×8
<time:Timeline id="program" clock={clock} end="w-close.end+4.5s">
  <time:Window id="w-hook" from="start+1.4s" for={hook-media.extent}/>
  <time:Window id="w-law" from="w-hook.end+1.7s" for={law-media.extent}/>   <!-- 链式：前窗.end+间隙 -->
  ...
</time:Timeline>
<whisperx:Alignment id="<seg>-align" narrative={story} segment={story.segment.<seg>}
  media={<seg>-media.media} domain={<seg>-media.domain} language="zh"/> ×7
<semantic:Projection id="story-time" narrative={story} timeline={program.timeline}>
  <semantic:Map alignment={...} domain={...} window={program.w-...}/> ×7
</semantic:Projection>
<ncap:Timing id="captions-timing" document={story.caption} binding={story.caption-binding} projection={story-time}/>
<cap:Style id="caption-style" recipe={look.caption.spoken} font={font}/>
<cap:Caption id="captions" document={story.caption} timing={captions-timing} timeline={program.timeline} within={full}>
  <cap:Use style={caption-style}/>
</cap:Caption>
<audio:Track id="narration" timeline={program.timeline}> 7 × <audio:Clip source during={program.w-*}/> </audio:Track>
<audio:Track id="music" ...><audio:Clip source={bgm-media.media} during={program.window} gain="0.16" fade-in="30f" fade-out="120f"/></audio:Track>
<!-- 场景（视觉层，film 子顺序不决定遮挡，但按此序可读）-->
<vibe:Stage during={program.window} watermark="Vibe知识大赏"/>
<vibe:<场景> during={program.w-*} font={font-black} ...参数/>
<film:Film id="main" canvas={canvas.canvas} timeline={program.timeline} appearance={look.film.main}>
  <film:Track source={stage.visual}/> ... <film:Track source={captions.visual}/>
  <film:Track source={narration.audio}/> <film:Track source={music.audio}/>
</film:Film>
<html:Video id="final" composition={main.composition} timeline={program.timeline}/>
```

## 细分窗口（如片头字卡 vs 法则卡共用一段旁白）

语义 Instant 切窗（不能反向：Timeline 子窗不能引用后声明的 Projection）：

```svml
<semantic:Instant id="quote-i" projection={story-time} at={story.moment.quote}/>
<time:Window id="w-title" timeline={program.timeline} from={program.w-law.start} until={quote-i}/>
<time:Window id="w-lawcard" timeline={program.timeline} from={quote-i} until={program.w-law.end}/>
```

## look.svs（配方）

头 `<?svml using="@hypit/recipe@1"?>`；每属性以 `;` 结尾；数字就是数字（letter-spacing: 3 不是 "0.05em"）。

字幕防叠画（关键）：

```svsl
caption.spoken {
  stack-order: 70; x: 0.5; y: 0.93; width: 0.88; height: 0.2;
  anchor-x: center; anchor-y: bottom; align: center; block-align: end; inline-size: fixed;
  wrap: word; size: 52; line-height: 1.3; fill: #F2E8D5;
  letter-spacing: 3; karaoke: current; active-fill: #E8CF9A;
  cue-enter: spring; cue-enter-frames: 4; cue-enter-start-scale: 0.86;
  cue-exit: none; lead-frames: 0; tail-frames: 0; handoff: cut;
}
```

## runs/main.svrun

头 `<?svml using="@hypit/markup/run@1"?>`；`<author source="../authors/main.svml"/>`（相对 run 文件）；`<target output="final.video"/>`。
