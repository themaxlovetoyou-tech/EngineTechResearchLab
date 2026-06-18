# AI 产业框架地图 · 工程规格 (SPEC)

> 这份文档是为长上下文对话中的 AI 协作者准备的"项目说明书"。每次接手前先读它。
> 文档与代码同步：改了代码必须同步修文档；改了文档但没动代码，要标记 `[PENDING]`。
>
> **最近一次修订：2026.06.07**

---

## 0. 一句话定位

一张可持续维护的 AI 产业「活地图」：把产业拆成 12 层 × 4 大族群，每周由自动任务采集动态、刷新数据，让读者从"骨架 + 经络"两个角度同时理解产业当前的形状与正在变化的张力。

---

## 1. 文件结构

| 文件 | 角色 | 是否同步 |
|---|---|---|
| `AI产业框架地图.html` | 主页面，单文件交付 | 主产物 |
| `AI产业框架地图_SPEC.md` | 本文件，工程规格 | 跟主页面同步 |
| (调度) `mnt/uploads/SKILL.md` 中的 `ai-stack-weekly-pulse` | 每周自动维护任务定义 | 由调度系统读取 |

主页面是**单文件 HTML**——所有 CSS 与 JS 内嵌，便于拷贝/分享/离线打开。
不要拆分成多文件，除非项目从"可分享的研究读物"演化为"可部署的 Web App"。

---

## 2. 顶层信息架构

页面分 6 段，按出现顺序：

| 段 | 锚点 | 角色 | 是否动态 |
|---|---|---|---|
| 顶部 Header | `#top` | 标题 + 元信息（编制 / 最近更新 / 层数 / 玩家数） | 元信息按周更新 |
| §00 产业动态 Pulse | `#pulse` | 动态流，每周追加一批 | **每周自动** |
| §01 全栈一览 | `#stack` | 12 层骨架 + 缩略 SVG | 静态 |
| §02 层级关系 | `#relations` | 价值流 / 动态 / 张力 / 压缩链 | 静态 |
| §详情 12 层 | `#details` `#l1..#l12` | 12 个长卡片（含玩家 / 规模 / 机会窗） | 数字与玩家季更 |
| §03 缺失维度 | `#dimensions` `#dim-a..c` | 安全/评估/法律、行业/地理、资本结构 | 半年级 |
| Footer | — | 使用说明、阅读路径、数据来源 | 静态 |

固定顶部导航 `.topnav` + 右侧抽屉式目录 `.toc-overlay` 横跨所有段。

---

## 3. 数据约定（最重要的一节）

### 3.1 族群与层级

四大族群，颜色与 CSS 类名固定：

| 族群 | CSS 类 | 颜色变量 | 包含层级 | 中文名 |
|---|---|---|---|---|
| g1 物理基础 | `g1` | `--g1` 赤陶 | L1–L3 | 能源 / 芯片 / 基础设施 |
| g2 模型与能力 | `g2` | `--g2` 深海 | L4–L7 | 大型语言模型 / 上下文 / 长期存储 / 工具集成 |
| g3 智能体 | `g3` | `--g3` 森林 | L8–L9 | 智能体框架 / 智能体 |
| g4 经济组织 | `g4` | `--g4` 陈金 | L10–L12 | AI native 部门 / 公司 / 经济生态 |

**约定**：任何引用层级的地方，标号必须是 `L<数字>`（大写 L），可通过正则 `/L\s*(\d+)/g` 解析。
JS 中已有的层级映射函数 `groupOfNum(n)` 是单一事实来源。

### 3.2 动态流 (Pulse) 条目结构

```html
<article class="pulse-entry g<N>">             <!-- N = 1..4 族群编号 -->
  <div class="pulse-entry-layer">L?<br>层名</div>  <!-- 可多层："L8 · L9" -->
  <div>
    <h3 class="pulse-headline">标题</h3>
    <p class="pulse-text">说明（1–2 句）</p>
    <a class="pulse-src" href="..." target="_blank" rel="noopener noreferrer">来源名 ↗</a>
  </div>
</article>
```

批次结构：

```html
<div class="pulse-batch">
  <div class="pulse-batch-head">
    <span class="pulse-batch-date">YYYY.MM.DD</span>
    <span class="pulse-batch-label">第 N 批</span>
    <span class="pulse-batch-count">X 条</span>
    <!-- pulse-batch-summary 由 JS 注入，不要手写 -->
  </div>
  <article class="pulse-entry ...">...</article>
  ...
</div>
```

**标记锚点**（自动任务依赖）：

- `<!-- PULSE-FEED:START -->` / `<!-- PULSE-FEED:END -->`
- `id="headerLastUpdate"` / `id="pulseLastUpdate"` / `id="pulseChangeCount"` / `id="pulseBatchCount"`

绝对不要删除这 4 个 id 或两个 HTML 注释标记，否则每周任务会断。

### 3.3 12 层卡片

每个层卡片是 `<article class="layer g<N>" id="l<num>">`，结构：

```
.layer
  .layer-marker (大数字 + 竖线)
  .layer-content
    .layer-head (标题 + 标签)
    [由 JS 注入] .layer-recent  ← 自动收集相关 pulse
    .layer-desc (一句话)
    .layer-body (描述 + 玩家 + 规模)
    [由 JS 注入] .layer-scorecard ← 6 维评分
    .opportunity (创业切入口)
    [由 JS 注入] .layer-mininav  ← 上一层 / 顶部 / 下一层
```

### 3.4 6 维层级评分

在 JS 中 `scores` 表是单一事实来源，键名固定：

| 维度 | 范围 | 含义 |
|---|---|---|
| moat | 0–5 | 护城河深度 |
| commod | 0–5 | 商品化压力（越高越被压扁） |
| risk | 0–5 | 5 年风险 |
| cycle | 0–5 | 客户决策周期（越长越高） |
| capital | 0–5 | 资本密度 |
| growth | 0–5 | 当前增速 |

更新规则：当 pulse 流连续 3-4 期影响同一维度（例如开源模型连续突破压低 L4 的 moat），就调整 1 档；不要频繁微调。

---

## 4. CSS 架构与已知技术债

### 4.1 颜色 / 字体 token

`:root` 已定义：
- 颜色: `--bg --bg-deep --ink --ink-soft --ink-mute --line --line-soft --accent --accent-deep --paper`
- 族群: `--g1..--g4` + `--g1-bg..--g4-bg`
- 字体: `--display --body --mono --cn-display`

`@media (prefers-color-scheme: dark)` 内已重新映射颜色 token，**保留族群语义**。
新增组件应只用变量，不要硬编码颜色。

### 4.2 已知债务（按优先级）

| # | 项 | 状态 | 描述 |
|---|---|---|---|
| 1 | 尺寸 token 化 | **PENDING** | 媒体查询里大量 `!important`，需引入 `--fs-* --pad-* --gap-*` 一系列尺寸 token，断点里只改变量。是未来"密度切换 / 紧凑模式"的前置条件 |
| 2 | 时间维度 | PENDING | 缺少 2017→2026 时间带，无法可视化"每层重心如何移动" |
| 3 | §04 失败档案 | PENDING | 全篇是赢家叙事，缺少"被压扁的层 / 被并购消失的玩家" |
| 4 | 3 个新横向维度 | PENDING | 人才流 / 开源份额 / 能耗。陈列在 §03，与现有 3 维并列 |
| 5 | 资料源「层 × 来源」矩阵 | PENDING | 每个 `scale-num` 应有 footnote 编号 |
| 6 | 玩家三层结构化 | PENDING | 平铺玩家芯片 → 头部 / 挑战者 / 悬疑 三栏 |
| 7 | 暗色模式手动切换 | PENDING | 目前只跟随 OS，没有按钮 |
| 8 | OG 卡片图 | PENDING | meta 已加，但没真的图片资源 |

---

## 5. JS 行为

主脚本是页面尾部单一 IIFE。注入顺序（重要）：

1. `buildObserver()` — 滚动监听 + 响应式 rootMargin
2. 全局 click 拦截器（冒泡阶段、不 stopPropagation）
3. TOC 抽屉控件
4. `onScroll` 与滚动节流
5. `setupPulseCollapse()` — **新加**: 折叠旧批次、注入工具条、持久化 localStorage 偏好
6. `injectScorecards()` — 6 维评分卡片
7. `wirePulseToLayers()` — 给 pulse 打 `data-layers`，给层卡注入「近期动态」
8. `injectLayerMiniNav()` — 层卡间迷你导航

注入函数都是幂等假定：**只调一次**，不要在用户交互后重复调用。

### 5.1 localStorage 键名

| 键 | 值 | 含义 |
|---|---|---|
| `pulseFeedMode` | `'compact'` \| `'expanded'` | 动态流默认显示模式 |

---

## 6. 自动维护任务约定

`ai-stack-weekly-pulse` 任务每周运行，逻辑：

1. 用 Glob 找到 `AI产业框架地图.html`
2. 用 WebSearch 搜过去 7 天 AI 产业变化（6–10 次搜索）
3. 在 `PULSE-FEED:START` 后插入新批次（5–9 条），每条带源 URL
4. 更新 4 个状态 ID：`headerLastUpdate / pulseLastUpdate / pulseChangeCount / pulseBatchCount`
5. 谨慎校准 12 层正文里过时的具体数字（宁可不动）
6. 控制长度：超过 10 个批次时删除最旧的

**任务行为不变量**：

- 不动 CSS、JS、页面结构
- 不动 `<!-- PULSE-FEED:START/END -->` 标记
- 每条动态必有真实来源 URL，数字模糊时如实表述
- 全程无需用户介入（autonomous）

---

## 7. 当前内容快照

- 编制日期: 2026.05
- 主页面最近更新: **2026.06.01** (第 2 批)
- 动态流批次数: 2
- 动态流总条数: 16
- 12 层卡片: 全部到位
- 玩家芯片: 约 300+
- 已实施速测: 12 层全部
- 已实施层间小导航: 12 层全部
- 已实施 pulse↔layer 联动: 12 层全部

---

## 8. 设计语言（不要破坏）

- 视觉风格："杂志式产业研究"，不是仪表盘。版面密度高、衬线斜体强调词、纸面颗粒底纹。
- 字体: 中文 Noto Serif SC + 英文 Fraunces (display) / Inter (body) / JetBrains Mono (mono)。
- 留白宁宽不窄。所有新增组件应保留同样的边距与字号节奏。
- 不要加 emoji。所有"强调"应通过 `.accent` 类或 `em` 斜体，不通过表情或颜色噪点。
- 任何用户可见文字，中文为主、英文小副标在斜体。

---

## 9. 验证清单（每次大改后必跑）

- [ ] Python `HTMLParser` 跑 0 错误
- [ ] `PULSE-FEED:START` / `PULSE-FEED:END` 标记仍在
- [ ] 4 个状态 ID 仍在且值正确
- [ ] `id="l1"..id="l12"` 12 个层 ID 都在
- [ ] `</body></html>` 结束、IIFE 闭合
- [ ] 浏览器无 console error
- [ ] 700px 与桌面端断点视觉无明显回退
- [ ] 暗色模式（OS 切换）下所有族群仍可读

---

## 10. 已决问题（不要再讨论）

- 不引入构建工具（Webpack/Vite），保持单文件可分享
- 不引入框架（React/Vue），原生 HTML+CSS+JS
- 不加 cookie / 追踪脚本
- 不引入外部图片资源（OG 卡片例外，但尚未做）
- 字体只用 Google Fonts CDN
- localStorage 仅用于偏好持久化，不保存任何用户数据

---

## 11. 下一步候选（按建议优先级）

1. 玩家信息三层化（头部 / 挑战者 / 悬疑）—— **内容价值最高**
2. CSS 尺寸 token 化 —— **基建价值最高**，为后续切换打基础
3. 2017→2026 时间带
4. 3 个新横向维度（人才 / 开源份额 / 能耗）
5. §04 失败档案
6. 资料源矩阵

每完成一项，请同步更新本文档 §4.2 与 §11。
