# 新项目：用 SDD 让 AI 根据规格建立项目
> 规格驱动开发（Spec-Driven Development）— 让 AI 不只写程序，还帮你建立完善的规格文档

## OpenSpec 初始化

### 🔧 为什么需要 OpenSpec？
- AI 写程序越来越快，但项目越改越乱，甚至越改越坏
- 关键人物离职，没有文档，系统知识直接断层
- 解法：白话对话 → AI 自动建立规格文档 → 根据规格驱动开发

### 📦 安装与初始化

```prompt [label="安装指令"]
npm install -g @fission-ai/openspec@latest
openspec init
```

- 选择使用的 AI 工具（Claude / Cursor）
- 生成 `.claude` / `.cursor` 下的 Agent Skills

### ⚡ Skills 与 Commands
- **Skills** — AI 在对话过程中自动触发的技能包，不需要背命令
- **Commands** — 用 `/opsx` 前缀强制驱动：apply / archive / explore / propose
- 可通过 `openspec config profile` 扩充更多 workflows

```prompt [label="查看 Skill"]
我想知道 openspec 目前安装的 skill 用途
请使用表格呈现，用白话简短描述
```

## 从零建立项目

### 🎯 Prompt 设计三要素

[flow]
1. 项目目标 — 大方向描述需求，AI 会厘清细节
2. 使用技术 — 指定技术栈，便于团队接手
3. 细节讨论 — 提醒 AI 主动提问，厘清模糊需求
[/flow]

```prompt [label="建立 Dashboard（Plan Mode）"]
请设计一个公司内部 Dashboard 系统，包含以下功能：
- 登录页面（账号密码验证，区分管理员与一般用户）
- 首页仪表板（显示关键数据卡片：营收、订单数、活跃用户数、转化率）
- 数据图表页（折线图、柱状图，支持日期筛选）
- 员工管理页（管理员可查看、新增、编辑、删除员工资料）

前端使用 React + TypeScript，使用 Mock API 模拟后端响应
这是初步需求，我们可以通过讨论厘清细节后，参考 openspec 的 skill 执行
以最小可行方案来规划
```

### 📋 OpenSpec 自动建立规格文档

[flow]
1. proposal.md — 确认目标与范围
2. design.md — 技术选型与风险评估
3. specs/ — 按功能分类的详细规格
4. task.md — 任务清单，完成后自动打勾
[/flow]

```prompt [label="开始实作"]
开始实作
```

> **AI 正在改变企业决策**
> 过去 Dashboard 这类系统，企业通常找厂商购买、支付年费维护。但 Vibe Coding 的出现正让企业做出不同的选择。
>
> 有些企业导入 Vibe Coding 的目标不是取代工程师，而是让熟悉业务的人有能力设计出符合使用需求的产品原型，再交给工程师做优化与维护。
>
> 用 OpenSpec 建立规格文档 —— 就是让这个交接过程有据可循，而不是丢过去一团没有文档的程序代码。

```prompt [label="归档"]
功能符合预期，进行归档
```

## 建立项目规则

### 📐 建立项目规则

```prompt [label="初始化规则"]
/init
```

```prompt [label="OpenSpec 设置"]
Please read openspec/config.yaml and help me fill it out
with details about my project, tech stack, and conventions
```

**CLAUDE.md** 是给“做事”用的，**config.yaml** 是给“规划”用的

---

# 旧项目：根据情境设计 Skills，让 AI 有执行依据
> 最难的不是 0 到 1，而是 1 到 100；通过 Skills 设计，让 AI 在迭代功能、版本控制、Code Review 时都有规范可循

## OpenSpec 迭代

### ⚠️ 版本控制的必要性
- 反面案例：一个 PR 塞了 18 个文件、近千行变更、只有一个 commit
- 结果：无法追踪设计逻辑、Code Review 形同虚设
- AI 加速开发后，这个问题被成倍放大

```prompt [label="新增功能"]
帮我设计 Dashboard 的深色/浅色主题切换功能
上方导航栏新增切换按钮，用户偏好存在 localStorage
使用 OpenSpec
```

```prompt [label="确认后实作"]
开始实作
```

> **为什么 1 到 100 比 0 到 1 更难？**
> 如果没有规格文档，下次改功能时 AI 不知道之前的设计逻辑，可能把同一个功能重复写好几次，或者改 A 坏 B。
>
> 用 OpenSpec 每次迭代都会在 Source Control 留下规格变更，AI 和人类都有文档可以参考。关键人物离职最痛的不是少了一个人，而是系统知识直接断层。

```prompt [label="归档变更"]
帮我归档
```

⭐ 保持好习惯：每做完一件事就 commit，不要把多个功能混在一起

## 设置 Commit Skill

### 📝 为什么需要 Commit Skill？

[tags]
- [orange] 手动输入：耗时且风格不一致
- [purple] AI 自动生成：长短随机、中英混杂
- [green] 解法：git-smart-commit Skill
[/tags]

- 分析变更的文件 → 判断应拆成几个 commit → 分段提交
- 不同功能的修改分开 commit，让逻辑可被追踪

```prompt [label="拆分 Commit"]
新增 commit
```

> **AI 生成的代码很多，但这不是你不看的理由**
> 导入 AI 后 Code Review 的负担大幅增加，有些公司认为一天要完成 3 倍工作量才算达标。结果大家牺牲深度思考，懒得 Code Review。
>
> 让 AI 根据 Skill 拆分 commit —— 就是在降低认知负担。每个 commit 只包含一件事，审核的人可以一步步理解设计逻辑。AI 生成的代码很多，但我们该想的是怎么更高效地审核，而不是放弃审核。

## 设置 PR Skill

### 🔀 git-pr-description Skill
- 比对当前分支与目标分支的差异
- 读取 commit 信息与变更文件
- 参考 `pr-template` 生成 Title 与 Description

```prompt [label="生成 PR"]
撰写 PR
```

## Git Worktree 并行开发

### 🌳 多 Agent 并行开发
- 不同功能使用不同 feature branch，搭配 Git Worktree 建立独立工作区
- 每个 Worktree 可同时运行不同 dev server，让多个 AI Agent 并行开发
- 设计 `git-worktree-design` Skill：一个指令拆分任务、建立 Worktree、安装套件、新增 SPEC

```prompt [label="Worktree 并行开发"]
采用 Worktree，新增通知中心弹窗、数据导出 CSV 功能、常见 QA 问答区
```

> **人，才是 AI 的瓶颈**
> Code Review 的速度已经跟不上 AI 写程序的速度。当人成为 AI 的瓶颈时，要思考的是如何降低门槛，而不是放弃审核。
>
> 每个功能单独验证，Code Review 时只需专注一件事。虽然每一步都是 AI 在执行，但如果没有业界经验，其实不知道怎么把这些工具串起来。**真正值钱的不是工具本身，而是知道什么时候用、怎么组合。**

### ⚠️ Worktree 注意事项
- 合并时可能有冲突：各分支独立开发，不知道彼此变更
- 建议共用功能优先开发、主分支变更时其他 Worktree 先同步
- 设计好流程才能提升效率

---

# 导入测试：让维护与扩展更有底气
> 市场不会为烂产品买单；加入自动化测试，是 Vibe Coding 从玩具走向产品的关键

### 🛡️ 为什么 Vibe Coding 一定要测试？

[flow]
1. 稳定性 — 请 AI 修 bug，结果旧功能坏掉
2. 复杂度 — 功能越多，人工测试越不可能覆盖全部
3. 扩展性 — 功能之间有依赖性，修改可能引发连锁影响
[/flow]

不写测试才浪费时间 —— 测试让你敢大胆修改，出错时能快速定位

## gen-test-cases

### 🔄 测试编写流程

[flow]
1. 建立文件夹 — 存放测试清单
2. AI 编写清单 — 类型、说明、输入、期待输出
3. 人类 Review — 确认情境有无遗漏
4. AI 编写测试 — 描述与文档一致
5. 自主验证 — 最多尝试 5 次
[/flow]

```prompt [label="生成测试案例"]
/gen-test-cases
（拖入 src/pages/LoginPage.tsx）
```

> **从玩具到产品，差的就是测试**
> 很多时候 AI 只是修好了眼前的错误，但过程中改坏了过去的逻辑。千万不要嫌写测试浪费时间 —— 测试其实是在帮你加速开发。
>
> 现在有 AI 了，你不用自己写测试程序，只要审核 AI 给出的测试情境是否有遗漏就好。

### ✅ 登录页测试情境示例

- [x] 电子邮件格式错误 → 前端拦住，不调用 API
- [x] 密码不符合规则 → 显示对应错误信息
- [x] 格式正确 → 调用 Mock API → 成功取得 Token
- [x] Mock API 返回密码错误 / 账号不存在 → 显示错误
- [x] 管理员登录 → 显示员工管理页入口；一般用户 → 不显示
- [x] 没有 Token 时直接访问 Dashboard → 导向登录页

### 💡 实务建议
- 不要一口气生成所有测试，先放一个文件确认结果符合预期
- 每个页面/模块有独立的测试程序，方便定位问题
- 测试案例会随着规格变更而调整，不可能一次到位

## GitHub Action 自动化

```prompt [label="自动化测试"]
我希望在 GitHub Action 中加入自动化测试流程
每个分支将更新推送到 GitHub 时都会触发一次自动化测试
测试完成后，要生成覆盖率报告让我下载
```

> **企业内训的真实反馈**
> 讲师设计的教材是最佳体验路径，但换到自己的项目情境一定会遇到不同问题。回去后不要只是复制指令，而是把流程搬到自己的项目里实际跑一遍。
>
> **学习 AI 不是搜集指令后复制粘贴，更重要的是理解应用情境后，通过实践调整为适合自己的工作流。**

### 🔁 自动化测试流程
- 每次推送到 GitHub 都触发测试
- 测试完成后生成覆盖率报告
- 设置 Branch Protection Rule：测试通过才能合并到主分支

测试覆盖率不需要追求 100%，重要逻辑都要测到。有了测试，规格书上的功能才能真正被验证。

---

# 总结

[summary]
- 🏗️ **新项目 — SDD** | OpenSpec + Spec-Driven Development，让 AI 根据规格建立 Dashboard，同时产出完善文档
- ⚙️ **旧项目 — Skills** | 设计 Commit / PR / Worktree Skills，让 AI 在大型项目中有规范可循
- 🧪 **导入测试 — CI/CD** | 用 Workflow 驱动 AI 编写测试，搭配 GitHub Action 守住质量底线
[/summary]