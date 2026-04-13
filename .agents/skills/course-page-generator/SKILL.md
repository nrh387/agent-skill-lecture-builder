---
name: course-page-generator
description: 将讲稿或非结构化笔记转换为约定的 Markdown 格式，再通过 build script 生成单一 HTML 课程页面与 OG 缩略图。Skill 的主要任务是 Markdown 格式转换，build 与 OG 图片生成是最后的必要步骤。
---

# 课程页面生成器

原始讲稿 → 结构化 Markdown → `node .agents/skills/course-page-generator/scripts/build.mjs <dir>` → `index.html` → `node .agents/skills/course-page-generator/scripts/generate-og.mjs <dir>` → `assets/og-*.jpg`

## 项目结构

```
.agents/skills/course-page-generator/
├── scripts/
│   ├── build.mjs          # 课程页面 build script
│   └── generate-og.mjs    # 为课程页面产出 1200x630 OG 缩略图（依赖 Puppeteer）
└── reference/             # 格式示例与 HTML 模板（包含支持 ?og=1 的 base.html）

<root>/                      # 任意根目录（例如 course/、lectures/、docs/）
├── config/
│   ├── global.yaml          # 全局设置（讲师、社群、页脚）
│   └── assets/              # 共用图片（avatar 等）
├── <course-dir>/
│   ├── config.yaml          # 课程专属设置（覆盖 global）
│   ├── content.md           # 结构化 Markdown 讲稿
│   ├── index.html           # 课程页面（build 生成）
│   └── assets/
│       └── og-*.jpg         # OG 缩略图（generate-og.mjs 产出）
```

## 工作流程

### Step 0：识别输入类型

在进入转换流程之前，先判断用户提供的是哪种输入：

| 情境 | 判断依据 | 动作 |
|------|----------|------|
| **只有主题** | 只给了一句话主题／标题，没有对应文件夹或 Markdown | → 执行“主题生成流程”（见下方） |
| **有讲稿内容** | 提供了讲稿文字、大纲，或已有 content.md | → 直接进入 Step 1 |
| **有现有目录** | 指定了已存在的课程文件夹 | → 读取后进入 Step 1 |

#### 主题生成流程

当用户只提供主题（例如“Python 异步编程”）：

1. **决定课程文件夹位置**：将主题转为 kebab-case 英文（例如 `python-async`）作为文件夹名称。
   - 如果用户指定了路径（例如 `lectures/python-async`），直接使用。
   - 否则，**用 Glob 工具扫描 repo 根目录**，观察现有课程文件夹放在哪一层（例如是否有 `course/`、`lectures/` 等约定目录），沿用同层。
   - 如果没有任何约定可循，直接建在 **repo 根目录**下（`<course-dir>/`），不要假设子目录。

2. **建立文件夹结构**：
   ```
   <root>/<course-dir>/
   ├── config.yaml      # 从主题推导课程设置
   ├── content.md       # 根据主题生成骨架
   └── assets/          # 空文件夹（预留给图片）
   ```

3. **生成 `config.yaml`**：根据主题填入基础字段（`page.title`、`page.hero_title`、`seo.title`、`seo.description`、`quotes.opening`、`quotes.closing`）；`seo.image` 与 `seo.url` 按照 Step 2-0 检测到的 GitHub Pages 前缀填入；如果检测失败则留空。

4. **生成 `content.md` 骨架**：
   - 推导 3–5 个主要章节（`#`），每章下 1–2 个子章节（`##`）与 2–3 张卡片（`### Emoji Title`）
   - 每张卡片留 2–4 个条列式占位点（重点提示，不是最终内容）
   - 最后加入 `[summary]` 区块，列出各章节预期的学习成果
   - 所有占位内容用 `<!-- TODO: ... -->` 或简短提示标记，让用户知道哪里需要补充
   - 骨架本身应具备足够结构，让 build 可以成功执行

5. **确认全局配置**：检查是否已有 `config/global.yaml`，如果没有，在回复中提示用户参考 Step 2 创建。

6. **告知用户**：列出已创建的文件清单，并说明下一步（补充内容或直接 build）。

7. **完成后继续 Step 2 以下流程**（确认 config → build → **OG 缩略图**）。Build 成功后**必须**立即执行 `generate-og.mjs`。

---

### Step 1：将讲稿转换为结构化 Markdown

这是 Skill 的核心任务。用户提供的内容可能是：
- 非结构化的演讲笔记或大纲
- 已部分格式化的 Markdown
- 课程投影片的文字内容

AI 需要根据以下语法规则，将内容转换为 `content.md`。

#### Markdown 语法约定

| 语法 | 用途 | 示例 |
|------|------|------|
| `# LABEL：TITLE` | 主章节 | `# 新项目：用 SDD 让 AI 根据规格建立项目` |
| `> lead text` | 章节引言（紧接 `#` 后） | `> 规格驱动开发（Spec-Driven Development）` |
| `## Title` | 子章节 | `## OpenSpec 初始化` |
| `### Emoji Title` | 卡片标题 | `### 🔧 为什么需要 OpenSpec？` |
| `` ```prompt [label="..."] `` | 终端机/Prompt 区块 | 见下方 |
| `> **Bold Title**` | 洞察框（Insight） | `> **AI 正在改变企业决策**` |
| `[flow]...[/flow]` | 流程步骤 | 见下方 |
| `[tags]...[/tags]` | 标签（必须用此区块包裹） | `- [green] 正面` |
| `[summary]...[/summary]` | 总结卡片 | `- 🏗️ **标题** \| 描述` |
| `- [x] item` | 勾选清单（仅用于已验证/已完成的事项） | `- [x] 已完成项目` |
| `![alt](src)` | 独立图片 | `![架构图](images/arch.png)` |
| `[image-text]...[/image-text]` | 图文并排 | 见下方 |
| `[youtube id="..." title="..."]` | YouTube 视频嵌入 | 见下方 |
| `---` | 章节分隔线 | 放在 `#` 章节之间 |

#### 详细语法

**Prompt Block：**
~~~markdown
```prompt [label="安装指令"]
npm install -g @fission-ai/openspec@latest
```
~~~

- Shell 指令开头（`npm`, `git`, `docker` 等）→ header 显示 "Terminal"
- 其他内容 → header 显示 "Prompt"

**Flow Steps：**
```markdown
[flow]
1. proposal.md — 确认目标与范围
2. design.md — 技术选型与风险评估
[/flow]
```

**Tags：**
```markdown
[tags]
- [green] 正面标签
- [orange] 警告标签
- [purple] 中性标签
- [blue] 信息标签
[/tags]
```
⚠️ `- [color] text` 必须放在 `[tags]...[/tags]` 内，独立使用不会套用颜色。

**Summary Grid：**
```markdown
[summary]
- 🏗️ **标题** | 描述文字
- ⚙️ **标题** | 描述文字
[/summary]
```

**Insight Box：**
```markdown
> **洞察标题**
> 第一段内容。
>
> 第二段内容（空行分隔）。
```

**独立图片：**
```markdown
![架构示意图](images/architecture.png)
```
- 独立一行 → 居中显示，alt 文字自动成为图注
- 在段落或列表中使用 → 行内图片

**图文并排（Image-Text）：**
```markdown
[image-text position="left" width="50"]
![产品截图](images/screenshot.png)
这是产品的主要界面，提供了 **直觉式操作** 体验。
- 支持拖放操作
- 实时预览结果
[/image-text]
```
- `position="left"`（默认）：图片在左、文字在右
- `position="right"`：图片在右、文字在左
- `width="N"` 设置图片占比百分比（默认 40），例如 `width="30"` 或 `width="60"`
- 文字区域支持段落、粗体、代码、链接、列表
- 响应式：平板（≤ 900px）及手机自动改为上下排列

**YouTube 视频嵌入：**

单行：
```markdown
[youtube id="dQw4w9WgXcQ" title="Demo 视频"]
```

区块（含说明文字）：
```markdown
[youtube id="dQw4w9WgXcQ"]
这是一段演示视频的说明
[/youtube]
```
- `id` 为 YouTube 视频 ID（网址中 `v=` 后面的值）
- `title` 为选填标题，显示在视频下方
- 视频以 16:9 比例响应式嵌入
- 打印模式下显示 YouTube 链接替代 iframe

完整组件对照请参考：[components.md](reference/components.md)
Markdown 示例请参考：[content-example.md](reference/content-example.md)

### Step 2：确认或建立课程 Config

#### 2-0：检测 GitHub Pages 前缀（必须先执行）

在编写任何 config 之前，先执行以下指令获取 GitHub Pages base URL：

```bash
git remote get-url origin
```

解析规则：
- SSH 格式 `git@github.com:user/repo.git` → `https://user.github.io/repo`
- HTTPS 格式 `https://github.com/user/repo.git` → `https://user.github.io/repo`

取得 `GH_BASE` 后，`seo.image` 和 `seo.url` 的值即为：
- `seo.url`：`{GH_BASE}/{course-dir}/`
- `seo.image`：`{GH_BASE}/{course-dir}/assets/og-image.jpg`（固定文件名，由 generate-og.mjs 输出）

如果指令失败（非 git repo、没有 remote、不是 GitHub），直接在 config 中留空这两个字段，并告知用户需要手动填入。

Config 分为两层：

| 文件 | 用途 | 必要性 |
|------|------|--------|
| `config/global.yaml` | 全局设置（讲师信息、社群链接、页脚） | 首次使用时建立一次 |
| `<course-dir>/config.yaml` | 课程专属设置（覆盖 global） | 每门课程各一份 |

> `config/global.yaml` 不需要放在固定位置，build 会从课程目录向上搜索最多 4 层父目录。只需确保它存在于课程目录的某个祖先层即可。

#### 首次设置：创建 Global Config

如果还没有 `config/global.yaml`，需要先建立全局设置。可参考 [config-example.yaml](reference/config-example.yaml) 作为模板：

1. 在课程目录的任意祖先层创建 `config/global.yaml`，填入讲师信息、社群链接、页脚默认值
2. 在同层创建 `config/assets/` 文件夹，放入讲师头像（文件名为 `author`，扩展名可省略，build 会自动检测 jpg/jpeg/png/webp/gif/svg）

全局设置的关键字段：

```yaml
instructor:
  name: "讲师姓名"
  tagline: "一句话简介"
  bio: "讲师介绍（支持 <br> 换行）"
  avatar: "config/assets/author"    # 可省略扩展名
  stats:                             # text 支持 **粗体** 等 inline markdown
    - text: "📚 出版 **7** 本专业书籍"
      url: "https://..."
  socials:                           # 支持 Medium/Facebook/Threads/YouTube/GitHub/LinkedIn/Email
    - platform: "YouTube"
      url: "https://..."

footer:
  cta: "行动呼吁文字"
  copyright: "© 你的名字"
  show_socials: true

seo:
  site_name: "网站名称"
```

#### 课程 Config

每个课程目录的 `config.yaml` 只需写要覆盖全局设置的字段：

```yaml
page:
  title: "课程标题"
  badge: "BADGE 文字"
  hero_title: "Hero 大标题<br>支持换行"
  subtitle: "副标题"

seo:
  title: "SEO 标题"
  description: "页面描述"
  image: "https://username.github.io/repo/<course-dir>/assets/og-image.jpg"  # 由 Step 2-0 检测填入
  url: "https://username.github.io/repo/<course-dir>/"                        # 由 Step 2-0 检测填入

quotes:
  opening:
    text: "开场引言"
  closing:
    text: >
      结尾引言
```

⚠️ **`seo.image` 必须使用绝对 URL**（`https://...`），社交平台无法解析相对路径，会导致 OG 预览图无法显示。这两个字段的值应在 Step 2-0 执行 `git remote get-url origin` 后填入。

`nav`（Hero 导航按钮）默认从 `content.md` 的 `#` 章节自动生成，不需要手动维护。
如需自定义按钮文案，可在 config.yaml 中覆盖：

```yaml
nav:
  - text: "自定义文字"
    href: "#section-id"
```

YAML 完整示例：[config-example.yaml](reference/config-example.yaml)

### Step 3 + 4：执行 Build 并生成 OG 缩略图（必须连续执行）

> ⚠️ **Step 3 与 Step 4 是绑定的：只要执行了 build，就必须接着生成 OG 缩略图。不能只做 build 而跳过 OG。**

所有指令都从 **repo 根目录** 执行：

```bash
# Step 3: Build 课程页面
node .agents/skills/course-page-generator/scripts/build.mjs <course-dir>

# Step 4: 生成 OG 缩略图（build 成功后立即执行）
node .agents/skills/course-page-generator/scripts/generate-og.mjs <course-dir>
```

示例：

```bash
node .agents/skills/course-page-generator/scripts/build.mjs course/cake
node .agents/skills/course-page-generator/scripts/generate-og.mjs course/cake
```

**Step 3 — Build 自动流程：**
1. 读取 `config/global.yaml`（base config）
2. 读取 `<course-dir>/config.yaml`（deep merge 覆盖）
3. 解析 `<course-dir>/content.md`
4. 套用 HTML 模板 → 填入 TOC / Scroll Spy
5. 输出 `<course-dir>/index.html`

**Step 4 — OG 缩略图（build 完成后自动接续）：**

> ⚠️ 此步骤为**必要步骤**，每次 build 完成后都**必须**执行，不可省略。需要 Puppeteer（`npm install --save-dev puppeteer`）。

`generate-og.mjs` 的行为概述：

1. 使用 Puppeteer 打开 `file://<course-dir>/index.html?og=1`
   - `?og=1` 会触发 `base.html` 中的 `og-mode`：
     - 隐藏 sidebar / footer / 普通内容 section
     - 只保留 hero 区块，适合作为社交缩略图
2. 将 viewport 设为接近手机宽度、较高 `deviceScaleFactor`，输出 1200×630 截图
3. 将结果存到 `<course-dir>/assets/og-*.jpg`，课程 config 的 `seo.image` 应指向该文件

## Config 合并规则

- **Deep merge**：课程 config 只需写要覆盖的字段
- **Arrays 整体替换**：例如 `nav` 或 `socials` 在课程 config 中有定义时，完整替换全局的
- **没有课程 config**：直接使用全局 config

## 转换指引

**⚠️ 核心原则：讲义是传递信息的载体，请“提炼重点”而不是“逐字转录”。**
请忽略口语化过场词（如：大家好、接下来我们看、老实说）、赘字与讲师自言自语，直接将讲稿“提炼”为结构化的条列重点、图表或卡片。

当用户提供原始讲稿时，AI 应该：

1. **提炼章节结构** — 移除口语过场，找出核心主题的切换点，对应到 `#` 主章节与 `##` 子章节。
2. **信息卡片化（极其重要）** — 将长篇解释提炼成精简的 `### Emoji Title` 卡片与条列列表，不要把讲稿段落直接复制粘贴。
   - **原则：所有内容都应该落在结构化组件内**（卡片、Insight、Flow、Tags 等），避免出现裸段落（`loose-text`）。
   - ❌ 错误示范：直接把讲稿贴成段落
     ```markdown
     一开始我把课程大纲交给 AI，第一版完成度很高。
     但 AI 会自己增减文字，想修改时得回去改 HTML。
     ```
   - ✅ 正确做法：提炼成卡片 + Insight
     ```markdown
     ### 💡 第一版很快，但改不动
     - 把课程大纲直接交给 AI，第一版完成度很高
     - 但 AI 会自行增减文字、改变强调方式
     - 想修改时，得回去改 HTML 原始码

     > **不能只停留在 AI 帮我生成第一版**
     > 讲义要维护的是内容，不是 HTML。
     ```
3. **标记指令与操作** — 代码、CLI 指令、Prompt 示例，用 `` ```prompt `` 包裹。
4. **整理流程步骤** — 讲到操作或逻辑步骤时，将其整理成清晰的 `[flow]...[/flow]`。
5. **提炼深度观点** — 将讲稿中的核心洞察、反思或重要结论，转为 `> **Title**` Insight Box 点出。
6. **生成总结** — 最后一个段落请直接用 `[summary]...[/summary]` 归纳本次课程精华。
7. **确认开场与结尾引言** — 检查课程 `config.yaml`（或 `global.yaml`）是否已设置 `quotes.opening` 和 `quotes.closing`。如果尚未设置，根据讲稿的核心精神各撰写一段引言，写入 `config.yaml`。开场引言出现在讲师介绍之后、第一章之前；结尾引言出现在所有章节之后、页脚之前，用于收束整场课程的信息。
9. **搜索并插入图片** — 当内容适合搭配图片时（架构图、截图、流程图等），主动用 Glob 工具搜索 `<course-dir>/assets/` 文件夹中的图片文件（`*.png`, `*.jpg`, `*.jpeg`, `*.gif`, `*.svg`, `*.webp`）。
   - **找到匹配图片**：根据文件名判断最合适的图片，插入对应的 `![alt](assets/filename)` 或 `[image-text]` 区块。
   - **找不到图片**：在该处插入 HTML 注释标记，格式为 `<!-- TODO: 建议在此加入图片：{图片描述}，请将图片放到 assets/ 文件夹 -->`，同时在回复中汇总所有缺图位置，提醒用户补充。
   - **assets 文件夹不存在**：提醒用户创建 `<course-dir>/assets/` 并放入相关图片。
10. **验证格式** — 对照 [components.md](reference/components.md) 确认语法正确。

## 参考文件

- 组件对照：[components.md](reference/components.md)
- YAML 示例：[config-example.yaml](reference/config-example.yaml)
- Markdown 示例：[content-example.md](reference/content-example.md)
- HTML 模板：[base.html](reference/base.html)