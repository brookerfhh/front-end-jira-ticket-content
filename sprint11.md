# Front-end Jira Tickets — Sprint 11

> Sprint：`MD 2026 Sprint 11`，周期 **2026-05-18 → 2026-06-01**

| Key | 摘要 (Summary) | 类型 |
|-----|----------------|------|
| [MD-17533](https://wonder.atlassian.net/browse/MD-17533) | Block "0" in UOM Conversions | Story |
| [MD-17934](https://wonder.atlassian.net/browse/MD-17934) | UI - [Wonder Create] Support Component Image | Sub-task |
| [MD-17935](https://wonder.atlassian.net/browse/MD-17935) | UI - [Wonder Create] Store Nutrition for 88\* / 7\* item | Sub-task |
| [MD-17936](https://wonder.atlassian.net/browse/MD-17936) | UI - IK Plating Rules configuration on menu item | Sub-task |
| [MD-17937](https://wonder.atlassian.net/browse/MD-17937) | UI - Allow for non-integer serving size for Nutrition team in Menu Item | Sub-task |
| [MD-17949](https://wonder.atlassian.net/browse/MD-17949) | UI - Optimize: Remove unnecessary requests and improve access speed | Story |
| [MD-17955](https://wonder.atlassian.net/browse/MD-17955) | UI - 70\* item allergens for pantry | Sub-task |
| [MD-17963](https://wonder.atlassian.net/browse/MD-17963) | [AI-GENERATED] Attribute 页搜索框按 Enter 不触发搜索 | Task |
| [MD-17969](https://wonder.atlassian.net/browse/MD-17969) | [AI-GENERATED] 修复 Snyk 报告的前端高危安全漏洞 | Sub-task |
| [MD-17977](https://wonder.atlassian.net/browse/MD-17977) | UI - Enable 7\* in 7\* | Sub-task |
| [MD-17982](https://wonder.atlassian.net/browse/MD-17982) | UI - Enable "IK Eligible" for Cook Steps | Sub-task |

---

## 前端需求拆解

### [MD-17533](https://wonder.atlassian.net/browse/MD-17533)

禁止 UOM Conversions 中出现 "0"

> 主 ticket（Epic）：[MD-17264](https://wonder.atlassian.net/browse/MD-17264) @2026 Regular optimization in Cookbook

**背景**：曾有 item 迁移到 40\*/41\* 模型时生成了 `1 EA = 0 g` 的换算，触发线上 pager 事故。根因是该迁移 item 没有 net weight，但通用原则是 unit conversion 里不允许出现 0。

- unit conversion 的换算值禁止填 0（校验拦截）
- 隐藏 UI 上的 `Schedule 40 model` 按钮，从入口上规避该 edge case（Bonnie 补充要求）

### [MD-17934](https://wonder.atlassian.net/browse/MD-17934)

UI - [Wonder Create] 支持 component 图片上传

> 主 ticket：[MD-17863](https://wonder.atlassian.net/browse/MD-17863)

- **40\* item 的字段**（`page/HDRConsumable/detail/component/BasicInformation/index.tsx`）：增加文件上传能力，实现方式对齐 Ingredient
- 详情页展示也对齐 Ingredient：把当前的 **media carousel** 换成 Ingredient 那种逐个平铺的布局，不再用轮播
- **所有 Object-type 的 "Upload file" 字段**：新增 `Customer Facing Image` 勾选项
  - 同一时刻只能有一个生效，**新的勾选会替换掉之前的**（此行为不可省略）
- 帮助文案沿用现有支持的文本：`Accepted file formats: JPG, JPEG, PNG, GIF, MP4, MOV, XLS, XLSX, PDF, DOCX and maximum size per file is 10M.`
- 详情页中 `Customer Facing Image` **单独一行**展示：勾选了才显示该行，未勾选则整行不出现

### [MD-17935](https://wonder.atlassian.net/browse/MD-17935)

UI - [Wonder Create] 为 88\* / 7\* item 存储并展示营养信息

> 主 ticket：[MD-17873](https://wonder.atlassian.net/browse/MD-17873)｜Epic [MD-17762](https://wonder.atlassian.net/browse/MD-17762) Wonder Create Integration

**背景**：Cookbook 要把 40\*/70\* item 提供给 Wonder Create AI 团队用于菜品创建，component 的营养信息需要在 Wonder Create portal 上展示。

- **7\* item 的 nutrition & allergen card**：显示 `Item` 与 `Components` 两个 tab
  - Item tab：card 名称下方显示时间戳 `Updated: mm/dd/yyyy hh:mm AM/PM`；`serving size = 1 ea`；Allergens 与 Dietary Tags 按 7\* 的**第一层** component 汇总；显示 Calories、Total fat 等营养素
  - Components tab：展示 7\* 的第一层 component；tab 下方显示提示 `Real-time nutrition in components`；列参照 88\* item；把 `Nutrition Status` 列替换为 `Dietary Tags`
  - 显示 `Recalculate` 按钮，支持手动重算营养
- **88\* item**：nutrition card 右上角显示 `Recalculate` 按钮
  - Item tab：card 名称下方显示时间戳 `Updated: mm/dd/yyyy hh:mm AM/PM`；`serving size = 1 ea`
  - Components tab：展示第一层 component；tab 下方提示 `Real-time nutrition in components`；在 `Nutrition Status` 列右侧新增 `Dietary Tags` 列
- 7\* / 88\* item 需要展示 nutrition change log

### [MD-17936](https://wonder.atlassian.net/browse/MD-17936)

UI - menu item 上的 IK Plating Rules 配置

- 新增两种 attribute 类型供用户配置：**IK Dish Type** 与 **IK Plating Rules**
- **Line Build**：当某个 step 带有 `IK Eligible` 复选框时，在其 **sub-step 层级**增加 `IK Plating Rules`
  - 单选，两个选项：`Default` 与 `Custom`
  - **仅当**当前 menu item 已配置了 IK Plating Rule 时，才默认选中 `Default`
  - 选中 `Default` 时，`Custom` 下拉框**禁用**
  - 从 `Custom` 切回 `Default` 时必须**清空**此前选中的值
  - 允许用户取消选择
  - 若选了 `Default` 但该 menu item 并未配置 IK Plating Rule，显示 inline error `Missing default rule`、**阻断保存**，并把错误上抛到 task 层级（可参考既有的 `checkIkEligibleMachineEligible` 函数）
- `IK Eligible = false` 时，`IK Plating Rules` 为**可选**（允许为空）
- `IK Eligible = true` **且**已选择 Mapping option 时，`IK Plating Rules` **必填** —— 无论该 Mapping option 是否 machine eligible
- `IK Eligible` 由 true 切为 false 时，**保留**此前选中的 IK Plating Rules 值
- **Line Build 详情视图**：在 sub-step 位置展示所选值，格式为 `(IK Plating Rules: xxx)`；step 层级的展示详情页已有，无需新增
- 该变更要纳入 Change Log

### [MD-17937](https://wonder.atlassian.net/browse/MD-17937)

UI - Menu Item 的 Servings Per Dish 支持非整数

> 主 ticket：[MD-17918](https://wonder.atlassian.net/browse/MD-17918)

**背景**：Nutrition 团队需要录入「2.5 servings per dish」，现有校验只允许整数。

- `Servings Per Dish` 放开为可填小数：**最多 1 位小数**，允许小于 1；小数位**只能是 .5**（不允许 .1/.2/.3/.4/.6/.7/.8/.9）
- serving per dish 为非整数时，consumer 侧的 serving size 展示名要转成分数形式：`1/2.5 dish` → `2/5 dish`
- normal version 与 variant **都要支持**
- 新增 feature flag `Enable decimal serving size`（通过 `useGetFlags()` 读取），**默认 false**
  - flag = true 时允许录入小数
  - flag = false 时沿用原有「必须为整数」的校验
  - 默认关闭，等 Consumer App 团队支持后再开

### [MD-17949](https://wonder.atlassian.net/browse/MD-17949)

UI - 性能优化：移除多余请求，提升页面打开速度

> 主 ticket（Epic）：[MD-17264](https://wonder.atlassian.net/browse/MD-17264) @2026 Regular optimization in Cookbook

- item 详情页不再初始化请求 concept list 与 restaurant 两个接口（该页面并不需要）
- 一并清理相关的无用代码

### [MD-17955](https://wonder.atlassian.net/browse/MD-17955)

UI - 为 70\* item 提供 regulation labeling（供 Pantry 打标签用）

> 主 ticket：[MD-17702](https://wonder.atlassian.net/browse/MD-17702)

**背景**：Pantry 在 HDR 打印 70\* 标签，标签底部需要 allergens，但此前没有接口能返回 70\* 的 allergens。

- 为 70\* item 增加 **regulation labeling card**（展示态）
  - `Individual Use Statement` 默认为 true
- 支持编辑 regulation labeling card
  - 字段校验沿用 88\* item 的既有规则
  - 权限沿用现有 regulation labeling 的权限
- 该 card 要纳入 change log
- allergens 的聚合去重（从 published/non-dormant 41\*、或 SCC status=active 的 42\* 汇总）在后端完成

### [MD-17963](https://wonder.atlassian.net/browse/MD-17963)

[AI-GENERATED] Attribute 页：搜索框按 Enter 不触发搜索

> 主 ticket（Epic）：[MD-17263](https://wonder.atlassian.net/browse/MD-17263) @2026 Cookbook AI Agent

- Attribute 页的搜索输入框中按 Enter 应触发搜索并展示筛选结果（当前必须点搜索按钮，不符合常规交互习惯）
- 实现方式：给搜索输入框加 `onKeyDown` 监听 Enter 键并触发搜索

### [MD-17969](https://wonder.atlassian.net/browse/MD-17969)

[AI-GENERATED] 修复 Snyk 报告的前端高危漏洞

> 主 ticket：[MD-17952](https://wonder.atlassian.net/browse/MD-17952) MD 2026 Sprint 11 Testing Bugs for Feature Development

- 逐条核对 Snyk 报告中前端的 high severity 漏洞，定位受影响的依赖包
- 能直接升级的升到已修复版本；无法直接升级的采用替代方案或 workaround
- 重新跑 Snyk 扫描确认高危项清零，同时确认依赖升级没有引入功能回归

### [MD-17977](https://wonder.atlassian.net/browse/MD-17977)

UI - 支持在 7\* item 的 component 中加入另一个 7\* item

> 主 ticket：[MD-17962](https://wonder.atlassian.net/browse/MD-17962)

**背景**：CE 要推进 Cook-and-Chill 微波米饭试点，需要允许 7\* 出现在另一个 7\* 的 BOM 中（例如 hot holding 的 7\*002 由 chilled 的 7\*001 而来）。

- 创建 / 编辑 7\* item 的 component → add component 时，选项中要出现 `HDR Recipe`，从而可以把另一个 7\* item 加为 component
- **create new packaged item 流程中不需要支持**（menu item 编辑页与 HDR recipe 编辑页均如此）
- **Line Build**
  - Hot Holding eligible 校验：只校验与 sub-step 映射的**主** 7\* item
  - KDS Portion QTY：逻辑不变，仍只校验直接映射的 item
- **customization 映射 item / line build sub-step 的 component 展开逻辑**：允许选择 menu item component 下的 7\* item **以及其所有嵌套子层级**
  - 例：menu item 的 component 为 `7*02 x 0.1 ea`（其下 `7*01 x 0.1 ea` → 其下 `40*01 x 50g`）和 `40*03 x 100g`，则可选项应包含 `7*02`、`7*01`、`40*01`、`40*03`
- Pantry 侧的库存扣减顺序与展开逻辑由 Pantry 后端处理

### [MD-17982](https://wonder.atlassian.net/browse/MD-17982)

UI - Cook 类型的 step 也支持勾选 "IK Eligible"

> 主 ticket：[MD-17947](https://wonder.atlassian.net/browse/MD-17947)

**背景**：此前只有 GARNISH step 能勾 IK eligible，但有些需要烹饪的 component 也适用于 IK 机器，因此要扩展到 Cook 活动类型。

- activity type 为 `cook` 或 `garnish` 时，显示 `IK eligible` 勾选框
- 沿用既有校验：只有当该 step 的 sub-step 中存在 machine eligible 的 item 时才允许勾选，否则禁止勾选并报错
- **勾选 IK eligible 后必须保留该 step 原有的全部配置**：cook step 要保留 appliance、cooking time 及其他烹饪相关字段
  - 目的是当 item 没有被填入 HDR 的 IK 槽位时，CE 仍能按原有 step 配置用 a la minute 方式烹饪
- menu item 的 line build **不需要**额外增加「如何预烹饪」的引导 step（该部分由 SOP / 线下培训覆盖）
