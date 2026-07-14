# AI 互动短剧 · 「长按稳住气场」互动 Demo

> 一个基于**竖屏短剧播放页**的可玩互动原型：视频播放到关键剧情点自动暂停，弹出「长按蓄力」互动组件；用户长按填满半圆进度环即可「帮男主稳住气场」，触发粒子庆祝动效，随后展示蓄力成功文案与短剧推荐卡；蓄力中途松手则判定失败并给出提示。

纯 `HTML + CSS + 原生 JavaScript` 实现，无任何框架依赖，单文件 `index.html` 即可运行。

---

## 一、体验设计（Experience Design）

### 1.1 设计目标
把传统「被动观看」的短剧播放，改造成一个**有参与感的轻互动**：在剧情高潮（男主即将登台演唱）时，邀请用户「长按帮他稳住气场」，用体感化的**蓄力**动作代替单纯点击，成功后给予**粒子庆祝 + 剧集推荐**的正反馈，自然引导用户「看后续剧情」。

### 1.2 完整体验流程

```
┌─────────────────────────────────────────────────────────────┐
│  ① 短剧正常播放（竖屏 / 进度条纯净轨道）                      │
│                     │ 视频播放到 19s（军中绿花剧情）           │
│                     ▼                                          │
│  ② 视频暂停 → 进入互动态（300ms）                              │
│     · 底部文案、右侧点赞/评论/转发栏 淡出                       │
│     · 顶部弹出互动标题横幅 + 中央长按组件（缩放淡入）           │
│     · 中心钮常驻动效：背景呼吸缩放+旋转、星光闪烁               │
│                     │                                          │
│         ┌───────────┴───────────┐                             │
│         ▼                        ▼                             │
│  ③A 长按蓄力成功            ③B 蓄力失败（中途松手）            │
│    进度环填满(0.8s)          文案变红「蓄力失败，请再次尝试～」 │
│    core/底衬/文字同步缩       持续1s后淡出恢复原文案            │
│         │                     （可重新长按重试）               │
│         ▼                                                      │
│  ④ 粒子炸裂庆祝(400ms)                                         │
│         ▼                                                      │
│  ⑤ 视频恢复播放 + 蓄力成功文案淡入                             │
│         │ 间隔 300ms                                           │
│         ▼                                                      │
│  ⑥ 短剧推荐卡向上位移+淡入(300ms)                              │
│     · 封面 / 标题 / 简介 / 标签 / “看后续剧情”按钮 / 关闭       │
│         │ 点击关闭短剧卡                                        │
│         ▼                                                      │
│  ⑦ 标题+卡片淡出(300ms) → 恢复底部文案/右侧操作区(300ms)       │
└─────────────────────────────────────────────────────────────┘
```

> 关闭按钮 `.interact-close` 可随时手动点击，跳过互动、恢复播放。

### 1.3 关键体验决策
| 决策 | 说明 |
|------|------|
| 用「长按蓄力」而非点击 | 增加参与的仪式感与情绪投入，契合「稳住气场」的语义 |
| 半圆进度环 + 中心钮 | 直观呈现蓄力进度，边按边看进度增长，形成即时反馈 |
| 中心钮常驻动效 | 背景暖晕呼吸缩放 + 中心旋转、星光闪烁，让按钮「有生命感」，引导用户去按 |
| 进度块跟随进度头 | 一个圆角小滑块随进度沿半圆环移动、长边始终指向圆心，强化进度可读性 |
| 按压同步缩放 | 长按时内芯、底衬圆、文字三者同步 `scale(0.96)`，实体按压手感 |
| 松手回落机制 | 蓄力未满松手会缓慢回落，鼓励用户「一鼓作气」按住 |
| 失败可重试 | 失败仅提示、不惩罚，1s 后自动恢复，允许无限次重试 |
| 成功后隐藏原生 UI | 结果态隐藏底部文案与操作栏，聚焦成功反馈与剧集推荐 |
| 关闭短剧卡后回归原态 | 卡片与成功标题一起淡出，随后底部文案 / 右侧操作区淡入恢复 |
| 粒子只在上半圈 | 庆祝粒子从进度环上方炸出，避免遮挡中心钮与文案，视觉聚焦 |

---

## 二、页面结构与视觉基准

- **设计基准**：`750 × 1624`（竖屏），通过 `fitViewport()` 等比缩放居中，适配任意屏幕。
- **舞台层级**（`#stage` 内，从底到顶）：
  1. 背景层：循环视频 `ai-drama.mp4` + 顶/底遮罩
  2. 蓄力失败红色蒙层（挂在文案横幅上，非全屏）
  3. 进度条（纯净轨道）/ 状态栏 / 导航栏
  4. 右侧操作栏 `.actions`、底部文案 `.caption`
  5. **互动层**：互动标题 `.interact-title`、长按组件 `.longpress`、粒子层 `.burst`、关闭按钮 `.interact-close`
  6. **结果层**：底部遮罩 `.result-mask`、成功文案 `.success-title`、短剧卡 `.drama-card`

- **长按组件 `.longpress` 内部层级**（从底到顶）：
  1. `.lp-decor`：沿半圆外缘的发光弧线装饰（`decor-arc.svg`）
  2. `.lp-track`：半圆进度环底轨
  3. `.lp-halo1`：底色圆晕（Figma 背景 PNG `halo-bg.png`）
  4. `.lp-progress`：半圆进度前景（JS 驱动）
  5. `.lp-slider`：跟随进度头的进度块
  6. `.lp-btn`：中心钮（见 3.3）

### 状态机（通过 `#stage` 的 class 切换）
| 状态 | class | 表现 |
|------|-------|------|
| 播放态 | （无） | 正常播放，互动元素隐藏 |
| 互动态 | `.interact-layer` | 暂停，显示互动组件，隐藏底部文案/操作栏 |
| 结果态 | `.interact-layer .result` | 隐藏互动组件，保持底部文案/操作栏隐藏，显示结果层 |

> 关键点：成功后**不移除** `interact-layer`（否则底部文案/操作栏会重新出现），而是**追加** `result` 类，仅隐藏互动组件。关闭短剧卡时才一次性移除 `result` 与 `interact-layer`，让原界面淡入恢复。

---

## 三、核心组件与技术实现

### 3.1 半圆进度环（纯 CSS，可交互）
进度环用 **`conic-gradient` + `mask` 挖空**实现，避免使用图片，方便 JS 实时驱动进度。

- `.lp-track`：底轨道，半圆（180°），浅橙 30%
- `.lp-progress`：进度前景，Angular 渐变（`#FFF3DC → #FFE895 → #FF8A14`）
- 通过 CSS 变量控制：
  - `--start: -90deg`（起点 9 点方向）
  - `--arc: 180deg`（9 点 → 12 点 → 3 点上半圈）
  - `--progress: 0~1`（sweep 进度，JS 驱动）
- 环形通过 `mask: radial-gradient(...)` 挖空中心得到 31px 宽的环。

```css
background: conic-gradient(from var(--start),
    #FFF3DC 0deg,
    #FFE895 calc(var(--arc) * var(--progress) * 0.66),
    #FF8A14 calc(var(--arc) * var(--progress)),
    transparent calc(var(--arc) * var(--progress)));
```

### 3.2 蓄力驱动（requestAnimationFrame）
用 rAF 逐帧根据「是否按住」增减进度，保证蓄力与回落都平滑：

```js
function tick(ts) {
  var dt = ts - lastTs; lastTs = ts;
  setProgress(progress + (holding ? dt / HOLD_MS : -dt / DECAY_MS));
  if (progress >= 1 && !done) { complete(); return; }   // 填满 → 成功
  if (holding || progress > 0) rafId = requestAnimationFrame(tick);
  else { rafId = null; lastTs = 0; }
}
```

| 参数 | 值 | 含义 |
|------|-----|------|
| `HOLD_MS` | `800` | 按住填满所需时长（0.8s） |
| `DECAY_MS` | `1200` | 松手后回落到 0 的时长 |
| 失败阈值 | `0.05 < progress < 1` | 有明显蓄力但未满 → 判定失败 |

`setProgress()` 每帧同步驱动进度前景与进度块位置。

### 3.3 中心钮（Figma PNG 素材分层重建）
中心钮容器 `.lp-btn` 为 294×294，内部**热区 194×194 居中**，按 Figma 图层顺序分层：

| 层 | 元素 | 说明 |
|----|------|------|
| 底衬 | `.lp-btn-plate` | 208×208 `rgba(255,255,255,0.20)` 半透明白圆（比内芯大一圈） |
| 内芯容器 | `.lp-btn-core` | 194×194 圆，`overflow:hidden` 作为**遮罩区域**，裁切内部所有动画 |
| ├ 底色 | `.lp-btn-ellipse` | `radial-gradient(#FCF4EC 70% → #FDEAD9 100%)` |
| ├ 背景 | `.lp-btn-bg`(wrap+img) | Figma `btn-bg.png`，**常驻缩放呼吸 + 中心旋转** |
| ├ 内阴影 | `.lp-btn-shadow` | Figma `btn-inner-shadow.svg`，圆内暗边 |
| └ 星光 | `.lp-sparkles`(wrap+img) | Figma `btn-sparkle.png`，**常驻缩放/透明度呼吸**，`brightness(1.5)` 提亮 |
| 文案 | `.lp-btn-text` | `点击/长按`，字体 `MF DianHei` 36px、字距 7px，flex 居中于内芯 |

**常驻动效**（进入互动态即一直运行，与蓄力状态无关）：
```css
/* 背景：外层 wrap 呼吸缩放，内层 img 中心旋转，两者叠加 */
.lp-btn-bg-wrap { animation: lp-btn-bg-breathe 2.4s ease-in-out infinite; }
@keyframes lp-btn-bg-breathe { 0%,100% { transform: scale(1); } 50% { transform: scale(2); } }
.lp-btn-bg-img  { animation: lp-btn-bg-spin 8s linear infinite; }
@keyframes lp-btn-bg-spin { to { transform: rotate(360deg); } }
/* 星光：缩放 + 透明度呼吸 */
.lp-sparkles-wrap { animation: lp-sparkles-breathe 2.8s ease-in-out infinite; }
@keyframes lp-sparkles-breathe { 0%,100% { transform: scale(0.85); opacity: 0.45; } 50% { transform: scale(1.1); opacity: 1; } }
```

**按压反馈**：长按时内芯、底衬圆、文案三者同步 `scale(0.96)`（`transition: transform 120ms`）：
```css
.lp-btn.pressing .lp-btn-core,
.lp-btn.pressing .lp-btn-plate,
.lp-btn.pressing .lp-btn-text { transform: scale(0.96); }
```

> 因内芯 `overflow:hidden`，背景放大到 2 倍时仅圆内可见，呼吸感明显且不溢出；`transform-box:fill-box` 等细节保证圆内裁切正确。

### 3.4 进度块跟随进度头（`.lp-slider`）
一个 36×8 的圆角小滑块（`#FCECDD` + 外发光阴影，样式对齐 Figma），随进度沿半圆环从 **9 点 → 12 点 → 3 点** 移动，**长边始终指向圆心（径向朝向）**。

- 与进度严格同步：在 `setProgress()` 中每帧调用 `updateSlider(progress)`
- 圆环中心 `(170, 157)`、环中心半径 `127px`
- 角度 `ang = -90° + 180° × progress`（-90°=9 点，0°=12 点，90°=3 点）
- 位置 `x = CX + R·sin(ang)`，`y = CY − R·cos(ang)`；朝向 `rotate(ang + 90°)`
- `progress > 0` 时淡入显示，回落到 0 时淡出隐藏

### 3.5 粒子炸裂（JS 动态生成）
成功时从进度环中心炸出 9 个粒子（星星 + 圆点簇），位置严格参照 Figma 布局，集中在**上半圈**，被约束在文案下方标注区域内。

- 锚点：圆环中心 `(375, 1224)`；粒子容器 `.burst` 为 0 尺寸锚点，粒子相对其定位
- 每个粒子通过 CSS 变量控制运动：
  - `--px/--py`：峰值位置（透明度 100% 时，收缩系数 `SHRINK = 0.75`）
  - `--tx/--ty`：最终位置（沿同方向继续扩散 `K = 1.45` 倍，可超出标注区域）
- 动画 `burst-fly`（**400ms**）：中心缩放淡入 → 峰值全显 → 外扩淡出

```css
@keyframes burst-fly {
  0%   { transform: translate(-50%,-50%) scale(0.2); opacity: 0; }
  30%  { transform: translate(calc(-50% + var(--px)), calc(-50% + var(--py))) scale(1.1); opacity: 1; }
  100% { transform: translate(calc(-50% + var(--tx)), calc(-50% + var(--ty))) scale(1.5); opacity: 0; }
}
```

### 3.6 装饰弧线（`.lp-decor`）
长按组件顶部沿半圆外缘的一条**发光弧线装饰**（Figma 素材 `decor-arc.svg`，314×157），作为组件最底装饰层，随组件一起淡入/淡出，`pointer-events:none` 不影响交互。

### 3.7 提示横幅（互动标题 / 蓄力成功）
- 底色：`#333333` 80% 不透明的横向渐变（两端透明，中段 `rgba(51,51,51,0.80)`）。
- `.interact-title`（互动标题）与 `.success-title`（蓄力成功）共用同色底衬，保持互动/结果两态一致。
- **蓄力失败**：给 `.interact-title` 加 `.fail` 类，在原黑色底（70%）之上叠加红色渐变（40%），文案切换为「蓄力失败，请再次尝试～」；持续 **1s** 后带 **300ms** 过渡恢复。用 `failTimer` 管理，重新长按 / 成功时清除。

### 3.8 短剧推荐卡与关闭恢复
按 Figma 1:1 还原：卡片背景图 + 半透明暗色叠加、封面、标题、简介（2 行截断）、标签、「看后续剧情」按钮、关闭按钮。
- 出现动效：`opacity: 0→1` + `transform: translateY(48px→0)`，**300ms**，向上浮现。
- **关闭恢复**：点击关闭时，短剧卡、成功标题 `.success-title`、底部遮罩 `.result-mask` 一起淡出（**300ms**）；**300ms 后**移除 `result` 与 `interact-layer`，底部文案与右侧操作区淡入恢复（**300ms**）。

### 3.9 播放进度条（纯净轨道）
底部播放进度条为一条 702×3 的 `white/0.1` 圆角轨道（`assets/base/progress-track.svg`），**不显示已播放高亮块**，保持画面简洁。

---

## 四、交互事件（Interaction Events）

| 事件 | 绑定对象 | 处理 |
|------|---------|------|
| `video.timeupdate` | 背景视频 | `currentTime >= 19` 时暂停并进入互动态（仅触发一次） |
| `pointerdown` | `.longpress` | `begin()`：开始蓄力，内芯/底衬/文字同步按压 `scale(0.96)`；清除上次失败提示 |
| `pointerup` | `window` | `release()`：停止蓄力；未满则判定失败；进度开始回落 |
| `pointercancel` | `.longpress` | `release()`：同上（触摸中断兜底） |
| `click` | 关闭按钮 `#interactClose` | `doClose()`：退出互动态、恢复播放（跳过互动） |
| `click` | 短剧卡关闭 `#dramaCardClose` | 卡片+标题+遮罩淡出，随后恢复底部文案/右侧操作区 |
| `resize` | `window` | `fitViewport()` 重新等比缩放 |

> 采用 **Pointer Events** 统一处理鼠标与触摸；`pointerup` 绑定在 `window` 上，保证手指移出组件外松开也能正确结算。

---

## 五、动画一览（Animations）

| 动画 | 时长 | 缓动 | 说明 |
|------|------|------|------|
| 进入互动态（组件淡入/缩放） | 300ms | ease | 长按组件 `scale(0.86→1)` + 淡入；底部文案/操作栏淡出 |
| 中心钮背景呼吸缩放 | 2.4s | ease-in-out | `scale(1↔2)`，圆内裁切 |
| 中心钮背景旋转 | 8s | linear | 背景图中心匀速旋转 |
| 中心钮星光呼吸 | 2.8s | ease-in-out | `scale(0.85↔1.1)` + 透明度 `0.45↔1` |
| 蓄力进度增长 | 约 800ms | 线性(rAF) | 半圆进度环 sweep `0→1` |
| 蓄力回落 | 约 1200ms | 线性(rAF) | 松手后进度回落 `→0` |
| 按压同步缩放 | 120ms | ease | 内芯/底衬/文字 `scale(0.96)` |
| 粒子炸裂 | 400ms | ease-out | 中心 → 峰值全显 → 外扩淡出 |
| 蓄力成功文案淡入 | 300ms | ease | 粒子结束后出现 |
| 短剧卡出现 | 300ms | ease | 向上位移 + 淡入 |
| 关闭短剧卡恢复 | 300ms + 300ms | ease | 卡片/标题淡出后，原界面淡入 |
| 失败文案/红底切换 | 300ms | ease | 显示即时，恢复时淡出→淡入 |

### 成功流程时序（`complete()`）
```
t=0     进度填满 → clearFail() → spawnBurst()（粒子 400ms）
t=500   进入结果态(add .result) + 视频 play() + 成功文案淡入 + 底部遮罩淡入
t=800   短剧卡向上位移 + 淡入（300ms）
```
（`500 = 400ms 粒子动画 + 100ms 余量`；`800 = 500 + 300ms 间隔`）

---

## 六、目录结构

```
ai互动短剧/
├── index.html                 # 全部结构 / 样式 / 逻辑（单文件）
├── README.md                  # 本文档
└── assets/
    ├── ai-drama.mp4           # 背景短剧视频
    ├── base/                  # 播放页基础 UI（状态栏/导航/操作栏/进度条轨道等）
    ├── longpress/             # 长按组件素材：
    │   ├── halo-bg.png            # 底色圆晕
    │   ├── decor-arc.svg          # 顶部发光弧装饰
    │   ├── btn-bg.png             # 中心钮背景暖晕
    │   ├── btn-inner-shadow.svg   # 中心钮内阴影
    │   ├── btn-sparkle.png        # 中心钮星光
    │   └── close-ring.svg / close-cross.svg  # 关闭按钮
    ├── particles/             # 成功炸裂粒子素材（星星 / 圆点簇）
    ├── drama-card/            # 短剧卡素材（封面 / 背景 / 图标 / 遮罩）
    └── CodeBuddyAssets/       # 从 Figma 导出的选区素材（源文件）
```

---

## 七、本地运行

由于使用了本地视频与图片资源，需通过本地 HTTP 服务运行（不能直接双击打开 `file://`，否则部分资源可能受限）：

```bash
cd ai互动短剧
python3 -m http.server 8899
# 浏览器打开 http://localhost:8899/index.html
```

### 快速验证互动
1. 打开页面，等待视频播放到 **19s**（男主敬礼剧情）自动暂停并弹出长按组件；此时中心钮背景在呼吸缩放+旋转、星光闪烁；
2. **按住**中心「点击 / 长按」钮约 **0.8 秒**，观察：内芯/底衬/文字同步按压缩放 → 进度块随进度环移动 → 进度环填满 → 粒子炸裂 → 成功文案 → 短剧卡上浮；
3. 试试蓄力中途**松手**，观察「蓄力失败」红色提示，1 秒后自动恢复，可再次尝试；
4. 点击短剧卡右上角**关闭**，观察卡片与标题淡出、底部文案与右侧操作区淡入恢复。

> 调试提示：想快速验证互动，可临时把 `index.html` 中的 `var TRIGGER = 19;` 改小（如 `2`）。

---

## 八、可配置参数速查

| 参数 | 位置 | 默认 | 作用 |
|------|------|------|------|
| `TRIGGER` | 19s 触发 IIFE | `19` | 触发互动的视频时间点（秒） |
| `HOLD_MS` | 长按 IIFE | `800` | 蓄力填满时长 |
| `DECAY_MS` | 长按 IIFE | `1200` | 松手回落时长 |
| 失败阈值 | `release()` | `0.05` | 判定失败的最小蓄力量 |
| 失败持续 | `showFail()` | `1000` | 失败提示停留时长 |
| 背景呼吸 | `lp-btn-bg-breathe` | `scale 1↔2` / `2.4s` | 中心钮背景缩放呼吸 |
| 背景旋转 | `lp-btn-bg-spin` | `8s` | 中心钮背景旋转周期 |
| 星光呼吸 | `lp-sparkles-breathe` | `2.8s` | 星光缩放/透明度呼吸 |
| 进度块几何 | `updateSlider()` | 中心 `(170,157)`、`R=127` | 进度块沿半圆环的运动参数 |
| 粒子数/位置 | `spawnBurst()` | 9 个 | 星星+圆点簇布局 |
| `SHRINK` | `spawnBurst()` | `0.75` | 粒子峰值收拢系数 |
| `K` | `spawnBurst()` | `1.45` | 粒子外扩倍数 |
| 粒子时长 | `.burst-p` 动画 | `400ms` | 炸裂动画时长 |
| 短剧卡间隔 | `complete()` | `300ms` | 成功文案后到短剧卡出现的间隔 |
