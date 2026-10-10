# EasySlot

> 不知道干什么？转一下。

EasySlot 是一个场景化的随机决策器：选一个生活场景，转动多列老虎机，得到一个有趣、可执行的主意。不满意就锁住喜欢的列，再转其他的。

V0.1 包含 5 个预设场景——今天做什么菜、约会去哪、聚会挑战、便利店调酒、情侣互动——以及自定义老虎机。移动端 Web 优先，无需登录，数据保存在浏览器本地；唯一的后端是模板分享用的短链接服务。

> 仓库名 `VibeSlot` 是项目代号；对外产品名为 **EasySlot**。

---

## 当前状态

**设计阶段完成，尚未开始开发。** 全部文档已就绪，可以交给 Coding Agent 按开发计划逐个任务实施。

仍待决定（不阻塞开发起步）：

| 事项 | 何时需要 | 位置 |
|---|---|---|
| 统计服务（Umami Cloud 或自部署） | 任务 T12 | TECH_DESIGN §19 T2 |
| 自有域名 | 任务 T13 | TECH_DESIGN §19 T3 |
| 限流规则不可用时的备选方案 | 任务 T11 | TECH_DESIGN §19 T4 |
| 反馈问卷平台 | 任务 T12 | TECH_DESIGN §19 T5 |
| 首页贴纸文案 | 任务 T07 前后 | UX_DESIGN §15 U5 |

---

## 文档

所有产品与设计文档在 [`design/`](design/) 下，建议按以下顺序阅读：

| 顺序 | 文档 | 回答的问题 |
|---|---|---|
| 1 | [`PRODUCT_VISION.md`](design/PRODUCT_VISION.md) | 为什么做、给谁用、产品原则、全部决策记录 |
| 2 | [`PRD.md`](design/PRD.md) | 做哪些功能、规则与验收标准 |
| 3 | [`CONTENT_DESIGN.md`](design/CONTENT_DESIGN.md) | 每台老虎机里的全部选项、句式与安全边界 |
| 4 | [`UX_DESIGN.md`](design/UX_DESIGN.md) | 页面结构、交互状态、动效节奏 |
| 5 | [`TECH_DESIGN.md`](design/TECH_DESIGN.md) | 代码组织、数据结构、存储、后端、部署 |
| 6 | [`IMPLEMENTATION_PLAN.md`](design/IMPLEMENTATION_PLAN.md) | 开发任务拆解、依赖与验收 |

视觉规范：[`.agents/skills/vibeslot-ui/`](.agents/skills/vibeslot-ui/SKILL.md)（Neo-Brutalism × 手绘贴纸 × 玩具机台），概念效果图在 `assets/ui-references/`。

**文档优先级**：Vision > PRD > UX；预设内容与文案以 CONTENT 为准；视觉数值以 skill 为准；实现方式以 TECH 为准。

---

## 交给 Coding Agent

1. Agent 会自动读取根目录的 [`AGENTS.md`](AGENTS.md)，其中要求界面相关工作加载视觉 skill。
2. 按 [`IMPLEMENTATION_PLAN.md`](design/IMPLEMENTATION_PLAN.md) 的顺序，**一次交一个任务**（T01 → T13），使用该文档 §7.1 的提示词模板。
3. 每个任务完成后，按其验收标准抽查，并确认 CI 通过，再进入下一个任务。
4. Agent 报告文档冲突或缺口时，先更新对应文档，再继续开发。

里程碑：M1 能转起来（T05）→ M2 完整的预设体验（T08）→ M3 自定义与分享（T11）→ M4 上线（T13）。

---

## 技术栈

React 18 · TypeScript · Vite · React Router · Zustand · Zod · CSS Modules · Web Animations API · Cloudflare Pages + Pages Functions + D1 · Umami（统计）

开发命令（项目初始化后可用，详见 TECH_DESIGN §16）：

```bash
npm install
npm run dev        # 前端
npm run dev:api    # 本地短链接接口（wrangler）
npm run check      # 类型检查 + lint + 内容检查
npm test
npm run build
```
