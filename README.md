# Course Page Generator

提供主题或 Markdown 讲稿，通过 Agent Skills 生成单一 HTML 课程页面。

## 快速开始

```bash
# 安装依赖（生成 OG 缩略图需要 puppeteer）
npm install
```

在 AI 对话窗口输入“帮我生成课程页面”这类指令，AI 会自动触发 `course-page-generator` Skill 完成所有步骤。

### 情境一：给主题，从零生成

只提供主题，AI 自动生成完整课程内容：

```
扮演一位擅长用实际案例讲解的资安专家，设计“生成式 AI 信息安全”的讲义并生成网页，完成后直接启动
```

AI 会依序执行：生成 `content.md` + `config.yaml` → build `index.html` → 生成 OG 缩略图 → 启动本地预览

### 情境二：给讲稿，辅助转换

提供现有讲稿、笔记或大纲，AI 会将其转换为结构化课程页面：

```
帮我把这份讲稿转成课程页面（贴上讲稿内容，或指定文件路径）
```

AI 会依序执行：提炼重点并转换为结构化 `content.md` → 建立 `config.yaml` → build `index.html` → 生成 OG 缩略图

## 项目结构

```
agent-skill-lecture-builder/
├── .agents/
│   └── skills/
│       └── course-page-generator/
│           ├── scripts/
│           │   ├── build.mjs        # 构建课程页面
│           │   ├── dev.mjs          # 本地预览
│           │   └── generate-og.mjs  # 生成 1200x630 OG 图
│           └── reference/
│               ├── base.html
│               ├── components.md
│               └── config-example.yaml
├── config/
│   ├── global.yaml          # 全局设置（讲师、社群、页脚）
│   └── assets/              # 共用图片（avatar 等）
├── example/                 # 课程目录示意（建立后会包含下列文件）
│   ├── config.yaml          # 课程专属设置（覆盖 global）
│   ├── content.md           # 结构化 Markdown 讲稿
│   ├── index.html           # build 产出
│   └── assets/
│       └── og-*.jpg         # generate-og.mjs 产出
├── package.json
└── README.md
```

## 新增一门课程

```bash
mkdir -p my-course/assets
```

1. **建立 `my-course/content.md`** — 用约定的 Markdown 语法编写讲稿（或把原始笔记交给 AI，触发 Skill 自动转换）
2. **建立 `my-course/config.yaml`** — 只需写要覆盖全局设置的字段
3. **Build**

```bash
node .agents/skills/course-page-generator/scripts/build.mjs my-course
```

产出 `my-course/index.html`。

如果只想单纯启动本地预览：

```bash
node .agents/skills/course-page-generator/scripts/dev.mjs my-course
```

也可以指定 port：

```bash
node .agents/skills/course-page-generator/scripts/dev.mjs my-course --port 8080
```

`dev.mjs` 会：

- 启动本地服务器
- 先自动执行一次 build
- 监听 `content.md`、`config.yaml`、`config/global.yaml`、`reference/base.html`
- 保存后自动重建并刷新浏览器

如果只想产出静态文件，不需要启动 server：

```bash
node .agents/skills/course-page-generator/scripts/build.mjs my-course
```

如果要产出 OG 缩略图：

```bash
node .agents/skills/course-page-generator/scripts/generate-og.mjs my-course
```

## Config 机制

两层设置，deep merge：

| 层级 | 文件 | 内容 |
|------|------|------|
| 全局 | `config/global.yaml` | 讲师信息、社群链接、页脚 |
| 课程 | `<dir>/config.yaml` | 页面标题、badge、hero、引言、导航按钮 |

课程 config 只需写要覆盖的字段，其余继承全局。数组字段（如 `socials`）会整体替换。

`nav`（Hero 导航按钮）默认从 `content.md` 的 `#` 章节自动生成，不需要在 config 维护。

`config/global.yaml` 不必放在固定位置。Build 会从课程目录向上搜索最多 4 层父目录，找到第一个 `config/global.yaml` 就使用。

### Global config 示例

```yaml
page:
  lang: zh-CN

instructor:
  name: "讲师名称"
  tagline: "一句话介绍"
  bio: >
    这里可放讲师简介。<br>
    支持 HTML `<br>` 换行。
  avatar: "config/assets/author"  # 可省略扩展名
  stats:
    - text: "📚 代表作品或经历 **X** 项"
      url: "https://example.com/books"
  socials:
    - platform: "YouTube"
      url: "https://youtube.com/@your-channel"

quotes:
  opening:
    text: "课程开场金句"
  closing:
    text: >
      课程结尾金句

footer:
  cta: "页脚行动号召"
  copyright: "© 你的名字"
  show_socials: true

seo:
  title: "默认 SEO 标题"
  description: "默认 SEO 描述"
  image: "https://your-domain.example/course/example/assets/og-image.jpg"
  url: "https://your-domain.example/course/example/"
```

### 课程 config 示例

```yaml
page:
  title: "课程标题"
  badge: "BADGE 文字"
  hero_title: "Hero 大标题<br>支持换行"
  subtitle: "副标题"

quotes:
  opening:
    text: "开场引言"
  closing:
    text: >
      结尾引言

# nav 自动从 content.md 的 # 章节生成
# 需要自定义时才写：
# nav:
#   - text: "自定义文字"
#     href: "#section-id"
```

如果你需要完整字段，请直接参考 [config-example.yaml](./.agents/skills/course-page-generator/reference/config-example.yaml)。

## Markdown 语法

| 语法 | 用途 |
|------|------|
| `# LABEL：TITLE` | 主章节 |
| `> lead text`（紧接 `#`） | 章节引言 |
| `## Title` | 子章节 |
| `### 🔧 Title` | 卡片 |
| `` ```prompt [label="..."] `` | 终端机 / Prompt 区块 |
| `> **Bold Title**` | 洞察框 |
| `[flow]...[/flow]` | 流程步骤 |
| `[tags]...[/tags]` | 标签（`green / orange / purple / blue`，必须用此区块包裹） |
| `[summary]...[/summary]` | 总结卡片 |
| `[bonus title="..."]...[/bonus]` | Bonus 按钮 + 弹窗 |
| `- [x] item` | 勾选清单（仅用于已验证/已完成事项，不适合一般观点条列） |
| `![alt](src)` | 独立图片（居中、含说明文字） |
| `[image-text]...[/image-text]` | 图文并排（图片+文字左右排列，默认图片占 40%） |
| `[youtube id="..." title="..."]` | YouTube 视频嵌入（16:9 响应式） |
| `---` | 章节分隔线 |

详细语法与 HTML 对照见 [components.md](./.agents/skills/course-page-generator/reference/components.md)。

### 图片

**独立图片（居中显示）：**
```markdown
![架构示意图](images/architecture.png)
```
- `alt` 文字自动成为图片说明（figcaption）
- 行内使用时（段落或列表中）会以内联方式渲染

**图文并排：**
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
- 平板（≤ 900px）及手机自动改为上下排列

### YouTube 视频嵌入

**单行（带标题）：**
```markdown
[youtube id="dQw4w9WgXcQ" title="Demo 视频"]
```

**区块（带说明文字）：**
```markdown
[youtube id="dQw4w9WgXcQ"]
这是一段演示视频的说明
[/youtube]
```

- `id` 为 YouTube 视频 ID（网址中 `v=` 后面的值）
- `title` 为选填标题/说明，显示在视频下方
- 视频以 16:9 比例响应式嵌入
- 打印模式下显示 YouTube 链接替代 iframe

### Bonus 弹窗

在任意章节（通常放在总结最下方）加入 `[bonus]` 区块，build 后会产出一个按钮，点击打开 Modal 弹窗，弹窗内容支持 Markdown。

```markdown
[bonus title="🎁 幕后制作心得"]
这里是**弹窗内容**，支持基本 Markdown：

- 段落间用空行分隔
- 列表用 `- item`
- 粗体用 `**文字**`
- 行内代码用 `` `code` ``

连续非空行会自动以 `<br>` 合并成同一段落。
[/bonus]
```

弹窗交互：
- 点击遮罩或右上角 ✕ 可关闭
- 按 `Esc` 也可关闭
