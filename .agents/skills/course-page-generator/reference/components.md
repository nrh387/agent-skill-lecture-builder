# 组件映射参考

Markdown 语法 → HTML 组件的完整对照。生成 HTML 时按此规则转换。

## 1. Section（主章节）

**Markdown：**
```markdown
# 新项目：用 SDD 让 AI 根据规格建立项目
> 规格驱动开发（Spec-Driven Development）— 让 AI 不只写程序，还帮你建立完善的规格文档
```

**HTML：**
```html
<hr class="divider">
<section class="section" id="new-project">
  <div class="reveal">
    <span class="section-label"><span class="num">1</span> 新项目</span>
    <h2>用 SDD 让 AI 根据规格建立项目</h2>
    <p class="lead">
      规格驱动开发（Spec-Driven Development）— 让 AI 不只写程序，还帮你建立完善的规格文档
    </p>
  </div>
```

规则：
- `#` 标题用冒号分成 label 和 title（`# LABEL：TITLE`），冒号前为 section-label，冒号后为 h2
- 如果没有冒号，整句作为 h2，section-label 使用简短版
- 紧接着的 `>` blockquote 作为 `.lead` 段落
- 自动递增编号（第一个主章节 = 1）
- `id` 由标题转为 kebab-case 英文（需要你根据语义判断）
- 第一个 section 前加 `<hr class="divider">`

## 2. Sub-section（子章节）

**Markdown：**
```markdown
## OpenSpec 初始化
```

**HTML：**
```html
<div class="reveal" id="sub-openspec-init">
  <div class="sub-title"><span class="bar"></span>OpenSpec 初始化</div>
```

规则：
- `##` 标题转为 `.sub-title`
- 自动生成 `id`（加 `sub-` 前缀）
- 出现在 TOC 的子项目中

## 3. Card（卡片）

**Markdown：**
```markdown
### 🔧 为什么需要 OpenSpec？
- AI 写程序越来越快，但项目越改越乱
- 关键人物离职，没有文档，系统知识直接断层
- 解法：白话对话 → AI 自动建立规格文档
```

**HTML：**
```html
<div class="card">
  <h3><span class="icon">🔧</span> 为什么需要 OpenSpec？</h3>
  <ul>
    <li>AI 写程序越来越快，但项目越改越乱</li>
    <li>关键人物离职，没有文档，系统知识直接断层</li>
    <li>解法：白话对话 → AI 自动建立规格文档</li>
  </ul>
</div>
```

规则：
- `###` 标题带 emoji → card 的 `h3`，emoji 放在 `.icon` span 中
- 下方的 bullet list → card 内的 `<ul>`
- 下方的段落 → card 内的 `<p>`
- 整个 card 包在 `<div class="reveal">` 中

## 4. Prompt Block（终端提示块）

**Markdown：**
~~~markdown
```prompt [label="安装指令"]
npm install -g @fission-ai/openspec@latest
openspec init
```
~~~

**HTML：**
```html
<div class="prompt-block">
  <div class="prompt-header">
    <div class="dots"><span></span><span></span><span></span></div>
    Terminal
    <span class="label">安装指令</span>
  </div>
  <div class="prompt-body">npm install -g @fission-ai/openspec@latest
openspec init</div>
</div>
```

规则：
- 用 `prompt` 作为 fenced code block 的语言标记
- `[label="..."]` 可选，显示在右上角
- header 中间文字默认是 "Prompt"，如果内容是 shell 指令则用 "Terminal"
- body 内容保持原样（`white-space: pre-wrap`）

## 5. Insight Box（洞察框）

**Markdown：**
```markdown
> **AI 正在改变企业决策**
> 过去 Dashboard 这类系统，企业通常找厂商购买、支付年费维护。
> 但 Vibe Coding 的出现正让企业做出不同的选择。
```

**HTML：**
```html
<div class="insight">
  <div class="insight-title">⏳ AI 正在改变企业决策</div>
  <p>过去 Dashboard 这类系统，企业通常找厂商购买、支付年费维护。</p>
  <p style="margin-top:.5rem">但 Vibe Coding 的出现正让企业做出不同的选择。</p>
</div>
```

规则：
- blockquote 第一行以 `**粗体**` 开头 → insight box
- 粗体文字成为 `.insight-title`（自动加 ⏳ emoji 前缀）
- 后续段落（以空行分隔）分别成为 `<p>` 元素
- 如果 blockquote 不是以粗体开头，则判断为普通引言或 lead 文字

## 6. Flow Steps（流程步骤）

**Markdown：**
```markdown
[flow]
1. proposal.md — 确认目标与范围
2. design.md — 技术选型与风险评估
3. specs/ — 按功能分类的详细规格
4. task.md — 任务清单，完成后自动打勾
[/flow]
```

**HTML：**
```html
<div class="flow">
  <div class="flow-step">
    <div class="step-num">1</div>
    <div class="step-title">proposal.md</div>
    <div class="step-desc">确认目标与范围</div>
  </div>
  <div class="flow-step">
    <div class="step-num">2</div>
    <div class="step-title">design.md</div>
    <div class="step-desc">技术选型与风险评估</div>
  </div>
  <!-- ... -->
</div>
```

规则：
- 使用 `[flow]...[/flow]` 包裹 ordered list
- 每个项目用 `—` 或 ` - ` 分隔标题和描述
- 如果没有分隔符，整行作为 step-title，step-desc 留空
- 也可以不用 `[flow]` 标记，在 `###` 卡片标题包含“流程”“步骤”等关键词时自动转换

## 7. Tags（标签）

**Markdown：**
```markdown
[tags]
- [orange] 手动输入：耗时且风格不一致
- [purple] AI 自动生成：长短随机、中英混杂
- [green] 解法：git-smart-commit Skill
[/tags]
```

**HTML：**
```html
<div class="tags">
  <span class="tag orange">手动输入：耗时且风格不一致</span>
  <span class="tag purple">AI 自动生成：长短随机、中英混杂</span>
  <span class="tag green">解法：git-smart-commit Skill</span>
</div>
```

可用颜色：`green`, `orange`, `purple`, `blue`

⚠️ 带颜色的 `- [color] text` 项目**必须**放在 `[tags]...[/tags]` 区块内才会生效。单独使用时会被当成普通文本，不会套用颜色样式。

## 8. Checklist（勾选清单）

**Markdown：**
```markdown
- [x] 电子邮件格式错误 → 前端拦住，不调用 API
- [x] 密码不符合规则 → 显示对应错误信息
- [x] 格式正确 → 调用 Mock API → 成功取得 Token
```

**HTML：**
```html
<ul class="checklist">
  <li>电子邮件格式错误 → 前端拦住，不调用 API</li>
  <li>密码不符合规则 → 显示对应错误信息</li>
  <li>格式正确 → 调用 Mock API → 成功取得 Token</li>
</ul>
```

规则：
- `- [x]` 项目 → checklist（CSS 自带 ✓ 图标）

适用场景：表达“已验证”“已完成”“已踩过的坑”等**带有完成语义**的事项清单。
- ✅ 适合：测试情境清单、踩坑记录、已确认的检查项
- ❌ 不适合：核心观点、重点摘要、一般条列（这些应使用卡片 `###` + 普通 list `- item`）

## 9. Summary Grid（总结卡片）

**Markdown：**
```markdown
# 总结
[summary]
- 🏗️ **新项目 — SDD** | OpenSpec + Spec-Driven Development，让 AI 根据规格建立 Dashboard
- ⚙️ **旧项目 — Skills** | 设计 Commit / PR / Worktree Skills，让 AI 有规范可循
- 🧪 **导入测试 — CI/CD** | 用 Workflow 驱动 AI 编写测试，搭配 GitHub Action 守住质量
[/summary]
```

**HTML：**
```html
<div class="summary-grid">
  <div class="summary-card">
    <div class="sc-icon">🏗️</div>
    <h4>新项目 — SDD</h4>
    <p>OpenSpec + Spec-Driven Development，让 AI 根据规格建立 Dashboard</p>
  </div>
  <!-- ... -->
</div>
```

规则：
- `[summary]...[/summary]` 包裹的 list
- 每项格式：`emoji **标题** | 描述`
- emoji → `.sc-icon`，粗体 → `<h4>`，`|` 后面 → `<p>`

## 10. Image（独立图片）

**Markdown：**
```markdown
![架构示意图](images/architecture.png)
```

**HTML：**
```html
<figure class="content-image">
  <img src="images/architecture.png" alt="架构示意图" loading="lazy">
  <figcaption>架构示意图</figcaption>
</figure>
```

规则：
- 独立一行的 `![alt](src)` → 图片区块，居中显示
- `alt` 文字自动成为 `<figcaption>` 图片说明
- 如果 `alt` 为空则不生成 figcaption
- 图片包在 `<div class="reveal">` 中
- 行内图片（在文字段落或列表中）则以 `<img class="inline-image">` 渲染

## 11. Image-Text（图文并排）

**Markdown：**
```markdown
[image-text position="left" width="50"]
![产品截图](images/screenshot.png)
这是产品的主要界面，提供了 **直觉式操作** 体验。
- 支持拖放操作
- 实时预览结果
[/image-text]
```

**HTML：**
```html
<div class="image-text" style="--img-width:50%">
  <div class="image-text__img">
    <img src="images/screenshot.png" alt="产品截图" loading="lazy">
  </div>
  <div class="image-text__body">
    <p>这是产品的主要界面，提供了 <strong>直觉式操作</strong> 体验。</p>
    <ul>
      <li>支持拖放操作</li>
      <li>实时预览结果</li>
    </ul>
  </div>
</div>
```

规则：
- `[image-text]...[/image-text]` 包裹图片与文字
- `position="left"`（默认）：图片在左、文字在右
- `position="right"`：图片在右、文字在左
- `width="N"` 设置图片占比百分比（默认 40），例如 `width="30"` 或 `width="60"`
- 文字区域支持段落、**粗体**、`程序代码`、链接、列表等行内格式
- 响应式：平板（≤ 900px）及手机自动改为上下排列（图片在上）
- 整个区块包在 `<div class="reveal">` 中

## 12. YouTube Embed（YouTube 视频嵌入）

**Markdown（单行）：**
```markdown
[youtube id="dQw4w9WgXcQ" title="Demo 视频"]
```

**Markdown（区块，含说明文字）：**
```markdown
[youtube id="dQw4w9WgXcQ"]
这是一段演示视频的说明
[/youtube]
```

**HTML：**
```html
<div class="youtube-embed">
  <div class="youtube-wrapper" data-id="dQw4w9WgXcQ">
    <iframe src="https://www.youtube.com/embed/dQw4w9WgXcQ" title="Demo 视频" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen loading="lazy"></iframe>
  </div>
  <p class="youtube-caption">Demo 视频</p>
</div>
```

规则：
- `id` 为 YouTube 视频 ID（网址中 `v=` 后面的值）
- `title` 为选填的视频标题/说明，会显示在视频下方
- 区块形式 `[youtube]...[/youtube]` 中间的文字作为 caption
- 视频以 16:9 比例响应式嵌入
- 打印模式下 iframe 隐藏，改为显示 YouTube 链接
- 整个区块包在 `<div class="reveal">` 中

## 13. Inline Elements（行内元素）

| Markdown | HTML | 说明 |
|---|---|---|
| `**bold**` | `<strong>` | 粗体 |
| `` `code` `` | `<code>` | 行内代码 |
| `[text](url)` | `<a href="url">text</a>` | 链接 |
| 一般段落 | `<p>` | card 或 section 内的段落 |

## 14. Wrapping Rules（包裹规则）

- 每个独立组件都包在 `<div class="reveal">` 中以启用滚动动画
- 连续的 card + prompt-block 可以放在同一个 reveal wrapper 中
- insight box 通常单独一个 reveal wrapper

## 社交链接 SVG 图标

用于 instructor section 和 footer 的社交链接 icon：

| 平台 | SVG |
|---|---|
| Medium | `<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M13.54 12a6.8 6.8 0 01-6.77 6.82A6.8 6.8 0 010 12a6.8 6.8 0 016.77-6.82A6.8 6.8 0 0113.54 12zm7.42 0c0 3.54-1.51 6.42-3.38 6.42S14.2 15.54 14.2 12s1.51-6.42 3.38-6.42 3.38 2.88 3.38 6.42zm2.94 0c0 3.17-.53 5.75-1.19 5.75-.66 0-1.19-2.58-1.19-5.75s.53-5.75 1.19-5.75c.66 0 1.19 2.58 1.19 5.75z"/></svg>` |
| Facebook | `<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z"/></svg>` |
| Threads | `<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12.186 24h-.007c-3.581-.024-6.334-1.205-8.184-3.509C2.35 18.44 1.5 15.586 1.472 12.01v-.017c.03-3.579.879-6.43 2.525-8.482C5.845 1.205 8.6.024 12.18 0h.014c2.746.02 5.043.725 6.826 2.098 1.677 1.29 2.858 3.13 3.509 5.467l-2.04.569c-1.104-3.96-3.898-5.984-8.304-6.015-2.91.022-5.11.936-6.54 2.717C4.307 6.504 3.616 8.914 3.589 12c.027 3.086.718 5.496 2.057 7.164 1.43 1.783 3.631 2.698 6.54 2.717 2.623-.02 4.358-.631 5.8-2.045 1.647-1.613 1.618-3.593 1.09-4.798-.31-.71-.873-1.3-1.634-1.75-.192 1.352-.622 2.446-1.287 3.263-.809.993-1.927 1.587-3.324 1.768-1.138.147-2.258-.058-3.15-.579-1.005-.586-1.65-1.524-1.818-2.638-.322-2.15.946-3.71 2.476-4.407.967-.44 2.164-.685 3.553-.731-.21-1.118-.658-1.905-1.348-2.365-.823-.548-1.943-.685-3.125-.61l-.145-2.118c1.508-.098 2.995.097 4.165.86 1.024.668 1.73 1.69 2.102 3.058.8-.065 1.559-.033 2.24.128 2.346.555 3.844 2.086 4.33 4.13.612 2.573-.134 5.46-2.392 7.35-1.895 1.588-4.258 2.392-7.028 2.392z"/></svg>` |
| YouTube | `<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M23.498 6.186a3.016 3.016 0 00-2.122-2.136C19.505 3.545 12 3.545 12 3.545s-7.505 0-9.377.505A3.017 3.017 0 00.502 6.186C0 8.07 0 12 0 12s0 3.93.502 5.814a3.016 3.016 0 002.122 2.136c1.871.505 9.376.505 9.376.505s7.505 0 9.377-.505a3.015 3.015 0 002.122-2.136C24 15.93 24 12 24 12s0-3.93-.502-5.814zM9.545 15.568V8.432L15.818 12l-6.273 3.568z"/></svg>` |
| GitHub | `<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.374 0 0 5.373 0 12c0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23A11.509 11.509 0 0112 5.803c1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576C20.566 21.797 24 17.3 24 12c0-6.627-5.373-12-12-12z"/></svg>` |
| LinkedIn | `<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433a2.062 2.062 0 01-2.063-2.065 2.064 2.064 0 112.063 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/></svg>` |
| Email | `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="2" y="4" width="20" height="16" rx="2"/><path d="M22 7l-10 7L2 7"/></svg>` |