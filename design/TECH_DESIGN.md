# TECH_DESIGN — EasySlot V0.1

> 文档版本：v0.2 ｜ 状态：评审中，关键选型已确认 ｜ 最后更新：2026-10-10
>
> 上位文档：`PRODUCT_VISION.md`、`PRD.md`、`CONTENT_DESIGN.md`、`UX_DESIGN.md`；视觉实现见 `.agents/skills/vibeslot-ui/`。
> 本文只回答**代码怎么组织、数据怎么存、后端怎么做、怎么部署**，不重复功能规则和视觉规范；行为以 PRD 为准，本文与 PRD 冲突时以 PRD 为准并修正本文。
>
> 标注约定沿用 Vision：【已确认】【建议】【待决策】【未来】。未标注的条目视为【建议】。

---

## 1. 总体架构

```
┌──────────────────────────── 浏览器（移动端优先） ────────────────────────────┐
│  React SPA                                                                   │
│  ├─ 页面：首页 / 老虎机 / 编辑器 / 导入预览                                    │
│  ├─ 引擎（纯函数）：可用项计算、加权随机、不连续重复、句式渲染                   │
│  ├─ 状态：会话（useReducer，不持久化） + 模板 / 偏好 / 应用状态（Zustand）       │
│  ├─ 持久化：localStorage（带版本号与迁移）                                     │
│  └─ 统计：Umami（可选加载，失败静默）                                          │
└───────────────┬──────────────────────────────────────────────────────────────┘
                │ 仅“分享模板 / 打开短链接”时
┌───────────────▼────────────── Cloudflare（同一域名） ─────────────────────────┐
│  Pages：静态资源 + SPA 回退                                                   │
│  Pages Functions：POST /api/share、GET /api/share/:id                         │
│  D1（SQLite）：shares 表                                                      │
└──────────────────────────────────────────────────────────────────────────────┘
```

原则：

- **除了短链接，一切在浏览器内完成**【已确认 D21】。后端不可用时，只有“生成短链接 / 打开短链接”受影响（PRD 非功能需求）。
- **引擎与界面分离**：随机、可用项、句式等逻辑是无副作用的纯函数，单元测试覆盖；组件只负责展示和派发动作。
- **前后端共享一份数据校验**：`shared/` 目录同时被前端和 Pages Functions 引用，保证导入校验与服务端校验一致。

---

## 2. 技术栈【建议】

| 类别 | 选择 | 理由 |
|---|---|---|
| 语言 | TypeScript（`strict: true`） | 【已确认 D14】 |
| 框架 | React 18 | 【已确认 D14】 |
| 构建 | Vite 5 | 【已确认 D14】 |
| 路由 | React Router 6（`createBrowserRouter`） | 生态成熟，Agent 熟悉 |
| 全局状态 | Zustand 4 | 体积约 1KB，API 简单；不用 Redux |
| 会话状态 | React `useReducer` | 状态机清晰、离开页面即丢弃（PRD R11） |
| 数据校验 | Zod 3 | 前后端共用一份 schema；类型可由 schema 推导 |
| 样式 | CSS Modules + 全局 CSS 变量（`tokens.css`，取值见 skill `visual-system.md`） | 与 skill 的 token 体系一致；不引入 UI 组件库 |
| 动画 | Web Animations API（`element.animate`）+ CSS | 走合成层，60fps；无需动画库 |
| 长链接压缩 | lz-string | 约 3KB；`compressToEncodedURIComponent` 输出可直接放进 URL |
| ID 生成 | nanoid（`customAlphabet`） | 体积小 |
| 单元测试 | Vitest + @testing-library/react | 与 Vite 同配置 |
| 端到端测试 | Playwright【P1】 | 有时间再做 |
| 托管 / 后端 | Cloudflare Pages + Pages Functions + D1 | 见 §10，【已确认】 |
| 统计 | Umami | 无 Cookie、支持自定义事件；见 §11，【待决策 T2】 |

**禁止引入**（除非先修改本文）：UI 组件库（MUI、Ant Design、Chakra 等）、Tailwind 等原子化 CSS 框架、动画库（Framer Motion、GSAP 等）、彩纸库、日期库、Lodash、任何 AI SDK、任何音频库（V0.1 无音效）。

**体积预算**（PRD：首屏 JS gzip 后 < 200KB）：React + ReactDOM ≈ 45KB、React Router ≈ 15KB、Zod ≈ 14KB、Zustand + nanoid + lz-string ≈ 5KB，业务代码目标 < 60KB。编辑器页与导入页按路由懒加载。以构建产物实测为准，CI 中输出体积。

---

## 3. 目录结构

```text
/
├── AGENTS.md
├── README.md
├── design/                       # 产品与设计文档（本文所在）
├── .agents/skills/vibeslot-ui/   # 视觉 skill
├── package.json
├── tsconfig.json
├── vite.config.ts
├── wrangler.toml                 # Pages + D1 绑定
├── index.html
├── public/
│   ├── _headers                  # 安全头、缓存头
│   ├── favicon.svg
│   └── og-image.png              # 分享预览图（无敏感内容）
├── shared/                       # 前端与 Functions 共用
│   ├── schema.ts                 # Zod schema（模板、分享载荷）
│   ├── limits.ts                 # 所有数量与长度上限
│   └── types.ts                  # 由 schema 推导的类型
├── functions/
│   └── api/share/
│       ├── index.ts              # POST /api/share
│       └── [id].ts               # GET  /api/share/:id
├── migrations/
│   └── 0001_create_shares.sql    # D1 表结构
├── scripts/
│   ├── delete-share.mjs          # 开发者按 ID 删除分享
│   └── sample-content.ts         # 内容人工抽查用：随机生成组合句子
└── src/
    ├── main.tsx
    ├── router.tsx
    ├── copy.ts                   # 界面固定文案（按钮、toast、错误提示）
    ├── styles/
    │   ├── tokens.css            # skill 中的 CSS 变量
    │   └── global.css
    ├── content/
    │   ├── presets/              # cook.ts / date.ts / party.ts / mix.ts / couple.ts
    │   ├── index.ts              # PRESETS 列表与按 ID 查找
    │   └── content.test.ts       # CONTENT_DESIGN §9.1 自动检查
    ├── engine/
    │   ├── random.ts             # 随机源
    │   ├── availability.ts       # 可用项、排除状态计算
    │   ├── pick.ts               # 加权随机 + 不连续重复
    │   ├── spin.ts               # 计算一次转动的目标结果
    │   ├── sentence.ts           # 句式解析与渲染
    │   └── *.test.ts
    ├── state/
    │   ├── session.ts            # 老虎机页会话 reducer（状态机）
    │   ├── templatesStore.ts     # 自定义模板
    │   ├── prefsStore.ts         # 每个模板的偏好
    │   ├── appStore.ts           # 一次性提示、最近使用、回访
    │   └── *.test.ts
    ├── storage/
    │   ├── storage.ts            # 安全读写 localStorage
    │   └── migrate.ts            # 各 key 的版本迁移
    ├── share/
    │   ├── resultText.ts         # 结果分享文本
    │   ├── longLink.ts           # 长链接编解码
    │   ├── shortLink.ts          # 短链接接口客户端
    │   └── shareSheet.ts         # 系统分享 / 剪贴板 / 兜底面板
    ├── analytics/track.ts
    ├── hooks/
    │   ├── useSpinAnimation.ts
    │   ├── useReducedMotion.ts
    │   └── useToast.ts
    ├── components/               # 组件划分见 skill pages-and-components.md §3
    │   ├── SlotCabinet/ ReelColumn/ ReelWindow/ LockButton/
    │   ├── ResultTicket/ PrimaryAction/ SceneControls/ SessionStatus/
    │   ├── BottomSheet/ ConfirmDialog/ NoticeDialog/ Toast/ GuideBubble/
    │   ├── SegmentedControl/ Chip/ Switch/ Confetti/ Stamp/
    │   └── stickers/             # SVG 贴纸组件，全部 aria-hidden
    └── pages/
        ├── HomePage/ PlayPage/ EditorPage/ ImportPage/ ErrorPage/
        └── PlayPage/ReelSheet.tsx、SettingsSheet.tsx
```

---

## 4. 数据模型

### 4.1 模板（`shared/schema.ts`）

以下为类型示意，实际以 Zod schema 为唯一定义，类型用 `z.infer` 推导。

```ts
export type SceneTheme = 'cook' | 'date' | 'party' | 'mix' | 'couple' | 'custom';
export type Mode = 'decision' | 'game';
export type Intensity = 1 | 2 | 3;

export interface Option {
  id: string;              // 模板内唯一；预设格式 `cook.main.01`，自定义为 nanoid(8)
  label: string;           // 1–8 字，唯一的展示文字（无 detail 字段，D23）
  weight?: number;         // 默认 1，> 0；仅预设使用
  intensity?: Intensity;   // 默认 1；仅带尺度的模板使用
  tags?: string[];         // 内容分组标签，必须在 tagGroups 中定义
}

export interface Reel {
  id: string;              // 模板内唯一，仅 [a-z0-9_]；句式占位 `{id}` 引用它
  name: string;            // 列名，1–8 字
  options: Option[];       // 2–30 项
  optional?: {             // 存在即为可选列（F10）
    defaultOn: boolean;
    sentenceSuffix?: string; // 开启时追加到句式末尾，可含本列占位
  };
}

export interface TagGroup {
  tag: string;             // 例如 'touch'
  name: string;            // 例如 '身体接触'
  defaultExcluded: boolean;// 例如 alcohol 为 true
}

export interface Template {
  schemaVersion: 1;
  id: string;              // 预设：'cook' 等；自定义：'c_' + nanoid(10)
  kind: 'preset' | 'custom';
  mode: Mode;
  theme: SceneTheme;       // 新建自定义模板为 'custom'；复制 / 导入的继承来源主题色（D30）
  name: string;            // 1–20 字
  subtitle?: string;       // 首页卡片副标题（预设）
  reels: Reel[];           // 1–4 列（含可选列）
  sentence?: string;       // 结果句式
  note?: string;           // 结果卡片底部提示，≤ 40 字（D25）
  notice?: string;         // 首次进入须知，≤ 60 字（D25）
  hint?: string;           // 仅预设：额外提示（cook 的“AI 推荐菜谱即将上线”），不复制、不分享
  intensityLabels?: [string, string, string]; // 存在即启用尺度筛选（F09）
  defaultIntensity?: Intensity;
  tagGroups?: TagGroup[];  // 内容分组（F22）
  createdAt?: number;      // 自定义模板
  updatedAt?: number;
}
```

**约束（`shared/limits.ts`，全部集中定义，前后端共用）：**

```ts
export const LIMITS = {
  templateName: 20, reelName: 8, optionLabel: 8, optionLabelWarn: 6,
  reelsMin: 1, reelsMax: 4, optionsMin: 2, optionsMax: 30,
  sentenceVisible: 60, note: 40, notice: 60,
  customTemplatesMax: 50,
  sharePayloadBytes: 8 * 1024,
  shareUrlMax: 200,
} as const;
```

**跨字段校验**（`superRefine`）：列 ID 唯一；同列 `label` 去首尾空格后唯一；句式占位只能引用存在的列，且非可选列不能出现在 `sentenceSuffix` 中；`tags` 必须在 `tagGroups` 中定义；所有字符串不得包含控制字符。

**句式可见长度**：占位符按对应列名的长度计算（编辑器中显示为列名标记），可见长度 ≤ 60。

### 4.2 预设数据

- 每个预设一个文件，导出 `Template` 常量；内容**逐字**来自 CONTENT_DESIGN §4–§8，不得改写。
- 列 ID 与 CONTENT 一致：`cook`（main/side/flavor）、`date`（activity/food/bonus）、`party`（who/task/rule）、`mix`（base/mixer/extra）、`couple`（who/place/action/rule）。
- 选项 ID 按 CONTENT §3.1 的规则：`{模板ID}.{列ID}.{两位序号}`，发布后永不修改、永不复用。
- `content.test.ts` 实现 CONTENT §9.1 的全部自动检查（标签长度、唯一性、列间不重叠、句式占位、尺度覆盖、分组覆盖、黑名单词等），作为 CI 必过项。
- `scripts/sample-content.ts`：`npm run sample -- cook 50` 输出 50 个随机组合整句，供 CONTENT §9.2 人工抽查。

### 4.3 偏好（每个模板一份）

```ts
export interface TemplatePrefs {
  excluded: string[];      // 用户主动排除的选项 ID
  restored: string[];      // 属于“默认排除分组”、但用户手动恢复的选项 ID
  intensity?: Intensity;   // 不存在时用模板 defaultIntensity
  optionalOn?: Record<string, boolean>; // 可选列开关；不存在时用 defaultOn
  respinLimit?: number | null;          // null = 不限；仅 decision
}
```

**排除状态的统一算法**（预设与自定义共用）：

```ts
isExcluded(option) =
  prefs.excluded.includes(option.id)
  || (hasDefaultExcludedTag(option) && !prefs.restored.includes(option.id))
```

这样做的好处：

- 首次进入 `party` 时无需写入任何数据，饮酒类自然处于排除状态（PRD F22-7）。
- 预设以后新增带 `alcohol` 标签的选项，会自动处于排除状态；新增普通选项自动可用（PRD F13-2）。
- 偏好中引用的已删除选项 ID 不影响计算，读取时顺带清理。

> **预设与自定义的排除统一存放在偏好中**【已确认，PRD F08-8】，模板本身只描述内容，引擎和界面只有一条代码路径。必须同时处理偏好的操作：
> - 复制模板：新模板的偏好 = 来源模板偏好的副本（排除、尺度、可选列开关、重转次数）。
> - 分享模板：载荷中去掉已排除选项（§9.2），偏好不进入载荷。
> - 删除模板：同时删除 `vs.prefs.byTemplate[id]`。
> - 导入模板：新模板没有偏好，按模板默认值生效。

### 4.4 应用状态

```ts
export interface AppState {
  seenNotices: string[];     // 已看过须知的模板 ID
  seenLockGuide: boolean;    // 锁定引导气泡（PRD F17）
  lastUsedAt: Record<string, number>; // 模板 ID → 时间戳，用于首页排序
  lastVisitDate?: string;    // 'YYYY-MM-DD'，用于 return_visit 事件
  storageWarned: boolean;    // “无法保存”提示是否已显示（仅内存，不持久化）
}
```

---

## 5. 本地持久化

### 5.1 存储键

| key | 内容 | 说明 |
|---|---|---|
| `vs.custom` | `{ schemaVersion: 1, templates: Template[] }` | 最多 50 个，估算上限约 250KB |
| `vs.prefs` | `{ schemaVersion: 1, byTemplate: Record<string, TemplatePrefs> }` | 预设与自定义共用 |
| `vs.app` | `{ schemaVersion: 1, ...AppState }` | |
| `vs.sharedDraft`（sessionStorage） | `{ payload, source }` | 临时模式数据，刷新 `/play/shared` 时使用；关闭标签页即清除 |

键名使用项目代号前缀 `vs.`（技术标识，D28）。

### 5.2 读写规则（`storage/storage.ts`）

```ts
read<T>(key, schema, migrate): T | null
  1. try localStorage.getItem；异常 → 标记 unavailable，返回 null
  2. JSON.parse 失败 → console.warn，返回 null（不删除原数据，便于排查）
  3. schemaVersion > 当前 → 返回 null（PRD F13-1：版本较新则忽略）
  4. schemaVersion < 当前 → migrate()
  5. schema.safeParse；整体失败时，对数组逐条校验，丢弃非法条目，保留合法条目

write(key, value): 'ok' | 'unavailable' | 'quota'
  - unavailable → 首次时 toast “当前浏览器无法保存，关闭页面后设置会丢失”（PRD F13-4）
  - QuotaExceededError → toast “保存失败，存储空间不足”（PRD F13-5），内存中的数据保留
```

- Store 变更后**同步写入**（数据量小，无需节流）；写入失败不回滚内存状态，保证当前会话可继续使用。
- localStorage 不可用时，各 store 以内存模式运行（PRD E09）。
- 多标签页：不监听 `storage` 事件，后保存者覆盖（PRD E15）。

### 5.3 版本迁移

- 每个 key 有独立的 `schemaVersion` 和迁移函数链 `migrations[n]: (v_n) => v_{n+1}`。
- V0.1 只有版本 1；任何字段的结构性变更都必须提升版本并补迁移函数和测试。

---

## 6. 随机引擎（`src/engine/`）

### 6.1 随机源

```ts
// 返回 [0, 1)
export function random(): number {
  if (globalThis.crypto?.getRandomValues) {
    const a = new Uint32Array(1);
    crypto.getRandomValues(a);
    return a[0] / 2 ** 32;
  }
  return Math.random();
}
```

所有引擎函数接受可注入的 `rng` 参数，测试中使用固定种子的伪随机数。

### 6.2 可用项

```ts
availableOptions(reel, template, prefs): Option[]
  = reel.options.filter(o =>
      !isExcluded(o, template, prefs) &&
      (!template.intensityLabels || (o.intensity ?? 1) <= currentIntensity(template, prefs)))
```

`activeReels(template, prefs)` = 非可选列 + 已开启的可选列，保持模板中的顺序。

### 6.3 加权随机 + 不连续重复（PRD F11）

```ts
pick(options, avoidId, rng): Option
  candidates = options.length >= 2 && avoidId
             ? options.filter(o => o.id !== avoidId)
             : options
  total = Σ (o.weight ?? 1)
  r = rng() * total
  for o of candidates: r -= (o.weight ?? 1); if r < 0 return o
  return candidates[candidates.length - 1]   // 浮点兜底
```

- 只有 1 个可用项时直接返回（PRD F11 验收）。
- `avoidId` 是该列**上一次转动的结果**；`idle` 预览不算“上一次”（PRD F11-3）。

### 6.4 一次转动

```ts
planSpin(template, prefs, session, rng):
  | { ok: true; targets: Record<reelId, optionId>; participating: reelId[] }
  | { ok: false; reason: 'allLocked' | 'noOptions'; reelId?: string }

  participating = activeReels.filter(r => !session.locked.has(r.id))
  if participating.length === 0 → allLocked（PRD E01）
  for r of participating:
    opts = availableOptions(r)
    if opts.length === 0 → noOptions(r.id)（PRD E05）
    targets[r.id] = pick(opts, session.lastSpun[r.id], rng).id
```

**结果在这里一次性确定**，之后交给动画展示（PRD R02）。

### 6.5 句式渲染

```ts
type Segment = { type: 'text'; text: string } | { type: 'value'; reelId: string; text: string; excluded: boolean };

renderResult(template, prefs, values): { labels: Segment[]; sentence: Segment[] | null }
```

- 解析 `sentence`：按 `/\{([a-z0-9_]+)\}/g` 切分为文本段与取值段。
- 已开启、且已有取值的可选列，追加其 `sentenceSuffix`；开启但尚为“?”占位时不追加（PRD F10-4）。
- 没有 `sentence` 时，`sentence` 返回 `null`，界面用 `labels`（“A × B × C”）作为大字显示（UX §5.8）。
- 组件按段渲染：取值段用场景色高亮，已排除的取值加删除线。**禁止拼接 HTML 字符串**。

---

## 7. 老虎机会话状态机（`src/state/session.ts`）

### 7.1 状态

```ts
interface SessionState {
  status: 'idle' | 'spinning' | 'result' | 'accepted';
  values: Record<string, string | null>; // 每列当前显示的选项 ID；null = “?”占位
  lastSpun: Record<string, string>;      // 每列上一次转出的结果，用于不连续重复
  locked: string[];                      // 已锁定列 ID
  plan: { targets: Record<string, string>; participating: string[]; startedAt: number } | null;
  respinsUsed: number;                   // 决策类：第一次之后的转动次数
  spinsSinceAccept: number;              // 防疲劳提示计数
  totalSpins: number;                    // 统计 nth / spins
  round: number;                         // 游戏类，从 1 开始
  fatigueShown: boolean;
}
```

### 7.2 动作与转移

| 动作 | 允许的状态 | 效果 |
|---|---|---|
| `INIT(previewValues)` | 任意 | 回到 `idle`，填入预览值，清空锁定与计数（进入页面、“重新开始”） |
| `SPIN_START(plan)` | `idle` / `result` / `accepted` | → `spinning`；非首次时 `respinsUsed += 1`（仅 decision）；`totalSpins += 1`；`spinsSinceAccept += 1` |
| `SPIN_END` | `spinning` | `values` ← `targets`；`lastSpun` 更新；→ `result` |
| `ACCEPT` | `result` | → `accepted`；`spinsSinceAccept = 0`；游戏类 `round += 1` |
| `TOGGLE_LOCK(reelId)` | `result` / `accepted` | 切换锁定；在 `accepted` 时 → `result`（决策类，PRD F05） |
| `AUTO_UNLOCK(reelIds)` | 任意非 `spinning` | 移除锁定（当前值被排除或超出尺度，PRD F04-6/7） |
| `VALUE_EXCLUDED(reelId)` | `result` / `accepted` | 自动解锁该列；`accepted` → `result`（PRD F08-9） |
| `OPTIONAL_TOGGLED(reelId, on)` | 非 `spinning` | 开：`values[reelId] = null`（“?”）；关：删除该列的值与锁定 |
| `FATIGUE_SHOWN` | — | `fatigueShown = true` |

- 主按钮的文案、可用性、剩余次数全部由 `SessionState + Template + Prefs` **派生**（`selectors.ts`），不单独存储。
- 剩余次数 = `limit == null ? ∞ : max(0, limit − respinsUsed)`，上限修改时立即重新计算（PRD F07-5）。
- 防疲劳：决策类、`spinsSinceAccept === 8 && !fatigueShown` 时显示 toast（PRD F05）。
- reducer 是纯函数，`session.test.ts` 覆盖 PRD F02/F04/F05/F06/F07 的全部验收条目。

### 7.3 全局 store（Zustand）

| store | 职责 | 持久化 |
|---|---|---|
| `templatesStore` | 自定义模板的增删改查、上限检查、复制 | `vs.custom` |
| `prefsStore` | 偏好读写；`toggleExclude`、`setTagGroup`、`restoreAll`、`setIntensity`、`setOptional`、`setRespinLimit` | `vs.prefs` |
| `appStore` | 须知 / 引导是否已看、最近使用、回访日期 | `vs.app` |

`prefsStore` 中的规则：

- `toggleExclude(templateId, optionId)`：若会让该列一个未排除项都不剩，返回 `{ ok: false, reason: 'lastOne' }`（PRD F08-6）。
- `setTagGroup(templateId, tag, on)`：关闭 → 该组全部选项设为排除；打开 → 只恢复**不属于其他已关闭分组**的选项（PRD F22-4）；会让某列无选项时整体不执行（F22-5）。“分组是否关闭” = 该组全部选项均已排除。
- `restoreAll(templateId)`：清空 `excluded`，并把所有带默认排除标签的选项加入 `restored`，即**全部选项恢复可用，包括饮酒类**【已确认 D29，PRD F08-10】。确认框文案见 UX §8。
- `restoreReel(templateId, reelId)`：同上，范围限定为一列（PRD F08-11）。
- “已排除 N 项”= 该范围内 `isExcluded` 为真的选项数（包括默认排除分组中的选项）。

---

## 8. 转动动画（`src/hooks/useSpinAnimation.ts`）

### 8.1 方案

使用 **Web Animations API** 对每列的“滚动条带”容器做 `transform: translateY` 动画：

```
点击 → planSpin() 确定 targets
     → 为每个参与列生成条带：[当前值, 随机填充 × n_i, 目标值]
     → element.animate(keyframes, { duration: T_i + 120, fill: 'forwards' })
     → 每列 finished → onReelStop(i)
     → 全部 finished → dispatch(SPIN_END)，onAllStopped()
     → 条带替换为静态三行（上一项 / 结果 / 下一项），transform 归零
```

- **条带长度**：`n_i = 14 + 4 × i`（i 为参与列中的序号），填充项从可用项中随机取、相邻不重复；只有 1–2 个可用项时允许重复。
- **时长**：第 i 个参与列在 \(T_i = 1100 + 300 \times i\) ms 停止，另加 120ms 回弹（UX §6.1）；4 列最长约 2.12s，满足 ≤ 2.5s。
- **关键帧**（距离 D = 条带长度 × 行高）：

```ts
[
  { transform: 'translateY(0)',              easing: 'cubic-bezier(0.5, 0, 1, 1)' },   // 加速
  { transform: `translateY(${-0.12 * D}px)`, offset: 0.12, easing: 'linear' },          // 匀速
  { transform: `translateY(${-0.55 * D}px)`, offset: 0.45, easing: 'cubic-bezier(0.15, 0.85, 0.3, 1)' }, // 制动
  { transform: `translateY(${-(D + 7)}px)`,  offset: 0.95, easing: 'ease-out' },        // 越过 7px
  { transform: `translateY(${-D}px)` },                                                 // 回弹落定
]
```

关键帧比例为起始值，真机调参后更新本节与 UX §6.1。

- **锁定列**不生成条带、不创建动画，DOM 不变（PRD F03-3、skill 要求“像素级静止”）。
- **只改 `transform` / `opacity`**；条带项预先渲染，动画过程中不触发 React 重渲染。

### 8.2 事件钩子（为 V0.2 音效预留，PRD F18）

```ts
interface SpinEvents {
  onSpinStart?(participating: string[]): void;
  onReelStop?(reelId: string, index: number): void;
  onAllStopped?(): void;
  onAccept?(): void;
}
```

V0.1 中这些钩子只用于停止时的视觉反馈和统计，不接任何音频。

### 8.3 边界情况

| 情况 | 处理 |
|---|---|
| 切到后台再回来（PRD E03） | WAAPI 基于文档时间线，回到前台时动画已结束并触发 `finished`；另设安全定时器 `maxDuration + 500ms` 强制 `finish()` 并派发 `SPIN_END` |
| 转动中离开页面 | 组件卸载时 `cancel()` 全部动画，清理定时器，不派发任何动作 |
| 屏幕旋转 / 尺寸变化（PRD E04） | 监听 `resize`，若在转动中，直接 `finish()` 跳到结果 |
| 减少动态效果（PRD F03-8） | 不生成条带；每列文字做淡出淡入切换，第 i 列在 `300 + 100 × i` ms 完成；无回弹、彩纸、印章动画 |
| P1 急停（F19） | 对仍在运行的动画按顺序间隔 100ms 调用 `finish()`，结果不变 |
| `Element.animate` 不可用 | 退化为减少动态效果的方案（目标浏览器均支持，仅作保险） |

### 8.4 彩纸与印章

- `Confetti` 组件：约 40 个（游戏类 20 个）绝对定位的小方块 / 圆点 / 彩条，每个用 WAAPI 做抛物线 + 旋转，1.2s 后移除 DOM；颜色取场景主色 + 2 个辅助色；`pointer-events: none`、`aria-hidden`。
- 减少动态效果时不渲染。

---

## 9. 分享

### 9.1 结果分享（PRD F14）

- `resultText.ts` 生成纯文本：`{句子或 A × B × C}\n—— 来自 EasySlot，不知道干什么？转一下 {链接}`。
- 链接：预设 → `${origin}/play/${id}`；自定义 / 临时模式 → `${origin}/`。
- `shareSheet.ts`：`navigator.share` → `navigator.clipboard.writeText` → 兜底面板（文本框 + 复制按钮）。用户取消系统分享（`AbortError`）不提示错误。

### 9.2 模板分享载荷

```ts
interface SharePayload {
  v: 1;
  template: Omit<Template, 'id' | 'kind' | 'hint' | 'createdAt' | 'updatedAt'>;
}
```

- 生成时：从自定义模板中**去掉已排除的选项**（PRD F15-4）；删除选项后少于 2 项的列 → 提示“『{列名}』可分享的选项不足 2 个”，不生成链接。
- 偏好（尺度档位、重转次数、可选列开关）不进入载荷。
- 接收方导入后，模板的 `tagGroups` 默认排除规则照常生效（例如饮酒分组对接收方默认关闭）——安全默认值，属预期行为。
- 大小：`new TextEncoder().encode(JSON.stringify(payload)).length ≤ 8KB`，超出时提示“模板太大，分享不了。试着删掉一些选项”。

### 9.3 短链接流程

```
分享模板
  → POST /api/share  body: SharePayload   （3 秒超时，AbortController）
     ├─ 201 { id } → 链接 `${origin}/s/${id}`（约 30 字符，≤ 200 ✓）
     ├─ 429        → toast “分享太频繁了，稍后再试”（PRD E18），不生成链接
     ├─ 400        → 视为前端 bug：上报 console.error，改用长链接
     └─ 超时 / 网络错误 / 5xx → 改用长链接 + toast “短链接生成失败，已改用长链接”（PRD E16）
  → shareSheet（同 9.1）

打开 /s/:id
  → GET /api/share/:id
     ├─ 200 → schema 校验 → 导入预览页
     ├─ 404 → “链接已损坏”（PRD E07）
     └─ 网络错误 / 5xx → “加载失败” + [重试]（PRD E17）
```

### 9.4 长链接兜底（`share/longLink.ts`）

- 格式：`${origin}/import#t=${LZString.compressToEncodedURIComponent(JSON.stringify(payload))}`。
- 数据放在 hash 中，不发送到服务器，不受 200 字符限制（PRD F15-2）。
- 解码失败、校验失败 → “链接已损坏”；`v` 高于当前版本 → “这个老虎机来自新版本，刷新一下试试”（PRD E08）。

### 9.5 导入与临时模式

- 导入预览页拿到载荷后，用 `shared/schema.ts` 按 PRD F12 规则重新校验。
- **直接转一下**：把 `{ payload, source }` 写入 sessionStorage `vs.sharedDraft`，跳转 `/play/shared`；该页从 draft 构造一个 `id: 'shared'` 的临时模板，偏好只存在内存中。刷新时从 draft 恢复；draft 不存在 → 回到首页。
- **保存**：生成新 ID（`c_` + nanoid(10)）、同名时追加“（2）”“（3）”……（PRD F15）、`kind: 'custom'`，`theme` 沿用载荷中的值，写入 `templatesStore`；然后 `navigate('/play/' + id, { replace: true })`。
- 达到 50 个上限时“保存”禁用，“直接转一下”可用（PRD F15-5）。

---

## 10. 后端：短链接服务

### 10.1 选型【已确认：Cloudflare Pages + Pages Functions + D1】

| 方案 | 结论 |
|---|---|
| **Cloudflare Pages + Pages Functions + D1（推荐）** | 静态站与接口同域部署，无 CORS；D1 是 SQLite，写入后读取立即可见；免费额度足够验证期 |
| Cloudflare Pages + KV | **不推荐**：KV 是最终一致性存储，写入后其他地区可能需要数十秒才能读到。用户刚分享、朋友马上打开，可能会看到“链接已损坏” |
| Vercel + 第三方 Redis | 需要两家服务，配置更多 |


D1 建库时指定靠近用户的位置提示（`--location apac`，以官方文档支持的取值为准）。免费额度以 Cloudflare 官方文档为准。

### 10.2 表结构（`migrations/0001_create_shares.sql`）

```sql
CREATE TABLE IF NOT EXISTS shares (
  id         TEXT PRIMARY KEY,      -- 8 位 [0-9A-Za-z]
  payload    TEXT NOT NULL,         -- SharePayload JSON
  created_at INTEGER NOT NULL       -- Unix 秒，用于排查；不记录 IP 等个人信息
);
```

### 10.3 接口

**`POST /api/share`**（`functions/api/share/index.ts`）

| 步骤 | 规则 |
|---|---|
| 1. 方法与头 | 只接受 `POST`；`Content-Type` 必须为 `application/json`；`Origin` 必须等于站点自身来源（不符 → 403） |
| 2. 大小 | 读取原始请求体，字节数 > 8KB → 413 |
| 3. 校验 | `JSON.parse` + `SharePayloadSchema.safeParse`（与前端同一份 schema），失败 → 400 `{ error: 'invalid' }` |
| 4. 生成 ID | `customAlphabet('0-9A-Za-z', 8)`（约 2.18 × 10¹⁴ 种组合） |
| 5. 写入 | `INSERT INTO shares (id, payload, created_at) VALUES (?, ?, ?)`；主键冲突时重新生成，最多 3 次，仍失败 → 500 |
| 6. 返回 | `201 { id }` |

**`GET /api/share/:id`**（`functions/api/share/[id].ts`）

| 步骤 | 规则 |
|---|---|
| 1. 参数 | `id` 必须匹配 `^[0-9A-Za-z]{8}$`，否则 404 |
| 2. 查询 | `SELECT payload FROM shares WHERE id = ?`；无结果 → 404 |
| 3. 返回 | `200`，原样返回 payload；`Cache-Control: public, max-age=300` |

- 所有响应带 `X-Content-Type-Options: nosniff`；错误响应不包含堆栈。
- 不提供列表、搜索、修改接口（PRD F15：只能按 ID 读取）。

### 10.4 限流

- **首选**：在 Cloudflare 控制台为 `POST /api/share` 配置 WAF 速率限制规则：同一来源每分钟最多 10 次，超出返回 429。代码中不处理 IP，符合“不存储 IP”的要求。
- 如果所用套餐无法配置该规则：【待决策】改为在函数中做计数（需要对来源 IP 做带每日轮换盐的哈希并在 1 分钟后删除）。这会短暂处理哈希后的 IP，需负责人确认可以接受。

### 10.5 管理脚本

```bash
npm run share:delete -- <id>
# 等价于：wrangler d1 execute easyslot --remote --command "DELETE FROM shares WHERE id = '<id>'"
```

- 脚本先校验 ID 格式再执行，防止误删。
- 由于 GET 响应缓存 5 分钟，删除后最长 5 分钟内仍可能被读到；需要立即生效时，在 Cloudflare 控制台按 URL 清除缓存。

---

## 11. 统计（PRD F16、§10）

### 11.1 选型【待决策 T2】

推荐 **Umami Cloud**（免费套餐，额度以官网为准），无需自行运维；自部署作为备选。

### 11.2 接入方式

- `index.html` 中仅在配置了 `VITE_UMAMI_WEBSITE_ID` 时插入脚本：`<script defer src="..." data-website-id="..." data-auto-track="false">`。
- **关闭自动页面统计**，由路由变化时手动上报**归一化路径**：`/`、`/play/cook`…、`/play/custom`、`/play/shared`、`/edit`、`/s`、`/import`。自定义模板 ID 与短链 ID 不进入统计。
- `analytics/track.ts`：

```ts
export function track(name: EventName, props?: Record<string, string | number | boolean>) {
  try { (window as any).umami?.track?.(name, props); } catch { /* 静默 */ }
}
```

- 事件名与属性**仅限** PRD §10 的 7 个事件；`EventName` 用联合类型限定，属性值只能是枚举或数值，编译期即可阻止上报用户文本。

| 事件 | 触发点 |
|---|---|
| `scene_open` | 老虎机页挂载（`type`：预设 ID / `custom` / `shared`） |
| `spin` | `SPIN_START` 时（`type`、`locked`、`nth` = min(totalSpins, 20)） |
| `accept` | `ACCEPT` 时（`type`、`spins`） |
| `share` | 分享成功调用系统分享或复制时（`type`、`kind`） |
| `import` | 导入预览页 `view` / 点击直接转一下 `try` / 保存成功 `save` |
| `custom_save` | 编辑器保存成功（`source`、`is_new`） |
| `return_visit` | 应用启动时，`lastVisitDate` 存在且不等于今天 → 上报一次并更新 |

---

## 12. 路由（`src/router.tsx`）

| 路径 | 页面 | 加载方式 | 说明 |
|---|---|---|---|
| `/` | HomePage | 首屏 | |
| `/play/shared` | PlayPage（临时模式） | 首屏 | 从 `vs.sharedDraft` 读取 |
| `/play/:templateId` | PlayPage | 首屏 | 预设 ID 或 `c_…`；找不到 → ErrorPage（PRD E06） |
| `/edit/new` | EditorPage | 懒加载 | |
| `/edit/:templateId` | EditorPage | 懒加载 | 仅自定义；预设 → 重定向到 `/edit/copy/:id` |
| `/edit/copy/:templateId` | EditorPage | 懒加载 | |
| `/s/:shareId` | ImportPage | 懒加载 | 短链接 |
| `/import` | ImportPage | 懒加载 | 长链接，读取 `location.hash` |
| `*` | ErrorPage | 首屏 | |

- Cloudflare Pages 在没有 `404.html` 时会把未匹配路径回退到 `index.html`，SPA 路由可直接工作；`/api/*` 由 Functions 处理。
- 编辑器保存后 `navigate(..., { replace: true })`，返回时回到首页（UX §2.2）。
- 编辑器的“放弃修改？”使用 React Router 的 `useBlocker`；浏览器刷新 / 关闭用 `beforeunload`。

---

## 13. 界面实现约定

视觉规范完全遵循 skill，这里只列与代码结构相关的约定。

- **设计 token**：`styles/tokens.css` 原样采用 skill `visual-system.md` §2 的变量；页面根节点设置 `data-scene={theme}`，组件只使用 `var(--vs-scene)` 等变量，不写死颜色。
- **组件**：按 skill `pages-and-components.md` §3 的结构拆分；每个组件一个目录（`index.tsx` + `*.module.css`）。
- **底部弹层**：用原生 `<dialog>` + `showModal()` 实现焦点约束与 Esc 关闭；打开时锁定背景滚动。
- **底部主按钮**：`position: sticky; bottom: 0` + `env(safe-area-inset-bottom)`；页面高度使用 `100dvh`，并提供 `100vh` 回退（skill `motion-responsive.md` §4）。
- **文案**：界面固定文案集中在 `src/copy.ts`（取自 UX §12、PRD 各处提示）；预设内容文案只在 `content/presets/`。组件内不出现硬编码中文。
- **用户内容**：一律作为 React 文本节点渲染；禁止 `dangerouslySetInnerHTML`、`innerHTML`、`eval`（PRD R14）。
- **无障碍**：结果区域 `aria-live="polite"`，仅在 `SPIN_END` 后更新文本；锁定按钮 `aria-pressed` + 完整 `aria-label`；装饰 SVG `aria-hidden="true"`。

---

## 14. 安全与隐私

| 项 | 措施 |
|---|---|
| 用户内容渲染 | 纯文本渲染（§13）；schema 拒绝控制字符 |
| 导入数据 | 前端与服务端同一份 schema 校验；大小上限 8KB |
| 接口滥用 | 同源检查、内容类型检查、大小限制、WAF 限流、ID 格式校验 |
| 违规内容 | 无公开浏览入口；导入预览默认折叠选项；开发者脚本按 ID 删除 |
| 安全响应头（`public/_headers`） | `Content-Security-Policy: default-src 'self'; script-src 'self' {统计域名}; connect-src 'self' {统计域名}; img-src 'self' data:; style-src 'self'; frame-ancestors 'none'`、`X-Content-Type-Options: nosniff`、`Referrer-Policy: strict-origin-when-cross-origin` |
| 隐私 | 无账号、无 Cookie；统计不含用户内容与模板 / 短链 ID；数据库不存 IP |
| 密钥 | 前端不含任何密钥；Cloudflare 凭据只存在本机 `wrangler` 登录态与 Pages 控制台 |

---

## 15. 测试策略

| 层级 | 工具 | 必须覆盖 |
|---|---|---|
| 内容检查 | Vitest（`content.test.ts`） | CONTENT §9.1 全部检查项 |
| 引擎 | Vitest | PRD F11 全部验收条目（分布、不连续重复、单项、锁定、排除与尺度） |
| 偏好与排除 | Vitest | PRD F08、F09、F22 的规则（含多分组交叉、最后一项保护、默认排除与恢复） |
| 会话 reducer | Vitest | PRD F02、F04–F07 的状态转移与派生文案 |
| 持久化 | Vitest（模拟 localStorage） | 非法 JSON、版本较新、逐条丢弃非法项、不可用、写满 |
| 分享编解码 | Vitest | 长链接往返一致、截断链接报错、大小超限、排除项不进入载荷 |
| 接口 | Vitest（直接调用 Functions handler，D1 用内存 SQLite 模拟或 Miniflare） | 合法写入与读取、400 / 403 / 404 / 413 |
| 组件 | @testing-library/react | 主按钮在各状态的文案与可用性、锁定按钮的无障碍属性 |
| 端到端【P1】 | Playwright | 首页 → 转动 → 锁定 → 就它了 → 分享；导入短链接 |
| 视觉 / 真机 | 手动 | skill `review-checklist.md`；iOS Safari、Android Chrome、微信内置浏览器 |

---

## 16. 开发与部署

### 16.1 本地开发

```bash
npm install
npm run dev            # Vite，http://localhost:5173
npm run dev:api        # wrangler pages dev（Functions + 本地 D1），http://localhost:8788
                       # vite.config.ts 中把 /api 代理到 8788
npm run db:migrate:local
npm test               # Vitest
npm run check          # tsc --noEmit + eslint + 内容检查
npm run sample -- cook 50
npm run build          # 输出 dist/，并打印各 chunk 的 gzip 体积
```

### 16.2 环境变量

| 变量 | 用途 | 必需 |
|---|---|---|
| `VITE_UMAMI_WEBSITE_ID` | 统计站点 ID；不配置则不加载统计 | 否 |
| `VITE_UMAMI_SRC` | 统计脚本地址 | 否 |
| `VITE_FEEDBACK_URL` | 首页“意见反馈”链接；不配置则隐藏入口 | 否 |
| D1 绑定 `DB` | 在 `wrangler.toml` 与 Pages 控制台中配置 | 是（生产） |

### 16.3 部署【域名待定 T3】

1. 在 Cloudflare 创建 D1 数据库 `easyslot`，执行 `migrations/0001_create_shares.sql`。
2. Cloudflare Pages 连接 GitHub 仓库 `Vinn0427/VibeSlot`：构建命令 `npm run build`，输出目录 `dist`，Node 版本 20；绑定 D1 为 `DB`；配置环境变量。
3. 绑定自有域名（Vision P4）；配置 WAF 速率限制规则（§10.4）。
4. `main` 分支自动部署到生产；其他分支自动生成预览地址。
5. 上线检查：PRD §1.2 发布标准 + skill 检查清单。

> 风险提示（与 Vision D2 一致）：`*.pages.dev` 默认域名在国内访问不稳定，**必须使用自有域名**；国内访问速度仍可能一般，有起色后按 D2 备案迁移。

### 16.4 持续集成（GitHub Actions）

- 每次推送与 PR：`npm ci` → `npm run check` → `npm test` → `npm run build`（输出体积）。
- 任一步失败则阻止合并。

---

## 17. 给 Coding Agent 的约定

1. **先读文档再动手**：功能看 PRD，内容看 CONTENT，布局与交互看 UX，视觉看 skill，实现方式看本文。文档之间冲突时，按 Vision > PRD > UX 的层级处理，并在报告中指出，不要自行改变产品决策。
2. **不得发明内容**：预设选项、句式、`note`、`notice` 逐字来自 CONTENT；界面文案来自 UX §12 与 PRD。
3. **不得超出范围**：PRD §12“明确不做”的功能一律不实现；不添加 §2 禁止的依赖。
4. **逻辑与界面分离**：新增业务规则先写在 `engine/` 或 `state/` 的纯函数中并配测试，再接到组件。
5. **TypeScript 严格模式**：不使用 `any`（第三方全局对象除外，需局部注释）；不使用 `@ts-ignore`。
6. **数据结构变更**：修改 `shared/schema.ts` 或存储结构时，必须提升 `schemaVersion`、补迁移与测试，并同步更新本文 §4–§5。
7. **完成标准**：`npm run check`、`npm test`、`npm run build` 全部通过；涉及界面时按 skill 检查清单自查；报告中如实说明哪些在浏览器或真机上验证过、哪些没有。

---

## 18. 与其他文档的同步项

| 项 | 同步位置 | 状态 |
|---|---|---|
| 排除状态统一存放在偏好中（预设与自定义一致） | PRD F08-8、F13 存储内容表 | 已同步 |
| 短链接存储定为 D1 | Vision D21；PRD Q7、Vision P6 已关闭 | 已同步 |
| “恢复全部排除”包括默认排除分组（如饮酒），确认框点明 | PRD F08-10/11、UX §7.2 / §8、Vision D29 | 已同步 |
| 新建模板为玩具蓝，复制 / 导入的继承来源主题色，显示“自制”小标签 | PRD F12.1、UX §5.2 / §11、Vision D30 | 已同步 |
| 模板中新增 `hint` 字段（仅预设，cook 的 AI 预告） | CONTENT §3.2 / §4.2 | 已同步 |

---

## 19. 待决策问题

| # | 问题 | 推荐默认值 |
|---|---|---|
| T2 | 统计服务 | Umami Cloud |
| T3 | 自有域名 | 上线前准备（Vision P4） |
| T4 | WAF 限流不可用时的备选方案 | 函数内对哈希 IP 短时计数（需确认可接受） |
| T5 | 反馈问卷平台 | 腾讯问卷 / 金数据（PRD Q8） |

---

## 20. 变更记录

| 日期 | 版本 | 变更 |
|---|---|---|
| 2026-10-10 | v0.1 | 初稿 |
| 2026-10-10 | v0.2 | 确认 D1、偏好统一存放、`hint` 字段；“恢复全部”包括默认排除分组；复制 / 导入继承主题色 |
