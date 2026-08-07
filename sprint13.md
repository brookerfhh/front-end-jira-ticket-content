# Front-end Jira Tickets — Sprint 13

> Sprint：`MD 2026 Sprint 13`，周期 **2026-06-16 → 2026-06-30**

| Key | 摘要 (Summary) | 类型 |
|-----|----------------|------|
| [MD-17664](https://wonder.atlassian.net/browse/MD-17664) | Block Linking DISH Buyout Vendor SKU to Raw Material in Cookbook | Story |
| [MD-17791](https://wonder.atlassian.net/browse/MD-17791) | UI - Tech 优化 menu item 详情页的 customization tab | Story |
| [MD-18057](https://wonder.atlassian.net/browse/MD-18057) | UI - Wonder Create debug page | Sub-task |
| [MD-18088](https://wonder.atlassian.net/browse/MD-18088) | UI - Copy Presets Along with BYO Menu Item | Sub-task |
| [MD-18090](https://wonder.atlassian.net/browse/MD-18090) | UI - Allow to Set Price=0 for Freemium Option | Sub-task |
| [MD-18092](https://wonder.atlassian.net/browse/MD-18092) | UI - Multi Select option 未映射 line build sub step 的 warning 校验 | Sub-task |
| [MD-18093](https://wonder.atlassian.net/browse/MD-18093) | UI - 切换 Display Style 到 In-Drawer 时清空 Featured 专有字段 | Sub-task |
| [MD-18094](https://wonder.atlassian.net/browse/MD-18094) | UI - 支持手动重算 Thawed 40 item 的 nutrition | Sub-task |
| [MD-18095](https://wonder.atlassian.net/browse/MD-18095) | UI - Block Linking DISH Buyout Vendor SKU to Raw Material | Sub-task |
| [MD-18099](https://wonder.atlassian.net/browse/MD-18099) | UI - 在 7\* item information card 上展示计算出的 net weight | Sub-task |
| [MD-18126](https://wonder.atlassian.net/browse/MD-18126) | Wonder Create debug page：弹窗右侧增加 `Deselect` 按钮 | Sub-task |
| [MD-18127](https://wonder.atlassian.net/browse/MD-18127) | UI - 支持手动覆盖 menu item 的 GF/GFO 标签 | Sub-task |
| [MD-18147](https://wonder.atlassian.net/browse/MD-18147) | Tech - UI - Node.js 升级到 24.17.0 并临时移除 ts check | Story |

---

## 前端需求拆解

### [MD-17664](https://wonder.atlassian.net/browse/MD-17664) / [MD-18095](https://wonder.atlassian.net/browse/MD-18095)

禁止把 DISH 的 buyout vendor SKU 关联到原材料（5\* / 9\*）

> 主 ticket（Epic）：[MD-17501](https://wonder.atlassian.net/browse/MD-17501) Supply Chain Catalog Integration
> MD-18095 是 MD-17664 的 UI sub-task，两者为同一需求

**背景**：Finance 要求一个 vendor SKU 不能同时关联多个 item（5\* 或 41\*）。由于 vendor SKU 的 delivery location（CK / DISH）在 VCS 里尚未维护，因此改为判断「该 vendor SKU 是否已在 SCC 中关联了 WSKU item」来间接识别 DISH 用途。

- 给 5\* / 9\* item 关联 vendor SKU 时，校验该 vendor SKU 是否已在 SCC 中关联 WSKU item（限 `SCC source = active` 且 `WSKU UOM = active`）
- 命中时在 5\* / 9\* 的 edit vendor item card 中报错：
  - 标题 `Unable to add vendor SKU`
  - 正文 `The vendor SKU has linked with WSKU {WSKU item number} in SCC. Please remove the linkage from SCC first.`
  - 按钮 `Cancel` / `Confirm`
- **通过模板文件批量编辑 vendor SKU 映射时同样校验**：命中的记录跳过，并在批量结果中输出错误信息
  - `Vendor SKU has linked with WSKU item in SCC:` 其下按 item 列出 `{Cookbook Item number}: {Vendor SKU Number} - {SCC WSKU item number}, …`

### [MD-17791](https://wonder.atlassian.net/browse/MD-17791)

UI - Tech 优化 menu item 详情页的 customization tab

> 主 ticket（Epic）：[MD-17264](https://wonder.atlassian.net/browse/MD-17264) @2026 Regular optimization in Cookbook

- customization 相关逻辑较为混乱且存在性能问题，本条为梳理与优化：识别问题点并改造

### [MD-18057](https://wonder.atlassian.net/browse/MD-18057)

UI - Wonder Create debug page

- 新增 Wonder Create 的调试页面（路由 `/wonder-create-debug`），供排查 Wonder Create 集成问题使用
- ticket 无描述，具体范围以实现为准；配套的交互补充见 MD-18126

### [MD-18088](https://wonder.atlassian.net/browse/MD-18088)

UI - 复制 BYO menu item 时一并复制其 presets

> 主 ticket：[MD-17989](https://wonder.atlassian.net/browse/MD-17989)

**背景**：BYO menu item 可以有 presets（各自有 80\* item number）。此前复制 menu ID 时 presets 不会被带过去，Culinary 团队必须逐个手工重建每个 preset 的 customization 配置、默认选项与 ineligible 选项。

- **Copy Item 弹窗**：当主 BYO menu item 存在 preset 时，在 `Item Information` 选项**上方**增加 `Presets` 选项；没有 preset 则不显示
  - 勾选 `Presets` 后，`Components (Bill of Materials)`、`Nutrition`、`Customization`、`Line Build` 自动勾选且**不可取消**
- **复制 preset 时**要带上原 preset 的全部配置：
  - 新 preset 名称默认在原名后追加新的 preset item number，例：`… Royal Greens AB PRESET` → `… Royal Greens AB PRESET 8000212`
  - customization 与 options
  - custom type（Single-select / Multi-select / Partial-select）、default option、default portion、ineligible options
- **异步化与结果反馈**：存在 preset 时，复制动作改为**异步**执行，提交后给出动画提示，结果在 Action History 面板中查看
  - 提交提示：`Copy menu item with presets is proceeding and will be completed in a few minutes, please check the action result in Action History.`
- **Action History 的改动**
  - 标题由 `export history` 改名为 `Action History`
  - Report Type 增加 `Copy Menu Item with Presets`
  - Note 文案：全部成功 `Successfully copied the menu item and all presets from {from BYO menu item number}`；有失败 `Failed to copied the below menu item(s): {menu item number1}, {menu item2}`
  - Status：`Final`（主 item 与所有 preset 均复制成功）/ `Failed`（主 item 或任一 preset 失败）；Action 列为空
- preset 与新 BYO menu item 的关联、复制后营养重算由后端处理

### [MD-18090](https://wonder.atlassian.net/browse/MD-18090)

UI - Freemium 的 option 允许价格为 0

> 主 ticket：[MD-17965](https://wonder.atlassian.net/browse/MD-17965)

**背景**：customization section 应用 `Freemium` 后，系统会报错 `Pricing will be adjusted outside of CB, typically in Merch tool.` 并阻断保存，迫使用户先填一个占位价格。而 Freemium 的定价本来就在 Merch tool 外部管理，这个校验不应拦住 Cookbook 的保存。

- customization section 应用 `Freemium` 时，**移除**「default price 必须 > 0」的校验报错
- 允许在 option price = 0 的情况下保存带 Freemium 的 customization section
- **非 Freemium** section 的价格校验行为保持不变

### [MD-18092](https://wonder.atlassian.net/browse/MD-18092)

UI - Multi Select option 未映射 sub step 的 warning，并把 KDS Portion 校验从 error 降为 warning

> 主 ticket：[MD-18030](https://wonder.atlassian.net/browse/MD-18030)

**背景**：此前「multi-select option 缺 KDS portion」是阻断保存的 error（见 sprint12.md 的 MD-18019）。业务反馈已有需要忽略 partial selection 的例外场景，因此改为 warning，不再阻断保存。

- **原有 error 改为非阻断 warning**（inline error 保留，但不再阻止保存 line build）
  - 场景一：sub step 勾了 `KDS Portion` 但缺少对齐的 KDS portion conversion
  - 场景二：sub step 映射了 multi-select option 但未勾 `KDS Portion`
  - warning 文案：标题 `Are you sure`，正文 `The bellow substep(s) is missing KDS portion, it will impact the cooking instruction in KDS, are you sure you want to save it?`，其下分列「缺少对齐的 KDS portion conversion」与「映射 multi-select option 却缺 KDS portion」两组明细，按钮 `Cancel` / `Save`
- **新增 warning**：若某个 multi-select 的 option 没有映射到**任何** sub step，保存 line build 时提示
  - 标题 `Are you sure`，正文 `The below multi select option is missing mapped substep, are you sure you want to save it?`，其下列出 `{customization name}-{option name}`，按钮 `Cancel` / `Save`
- **既有「option 未映射 substep」的 warning 文案增强**：multi-select 的排在最前，且 customization 名称后要加**加粗**的 `(multi select)`
  - 例：`The following option value name(s) is missing mapped step/sub step: Protein(multi select)--Chicken; Sauce--Ketchup, Mayo; Side--Fries`
- **Missing Info**
  - 新增 `Multi Select Missing Line Build Substep`（发布时为 warning），筛选归入 `CDT` → line build 组
  - 判定：custom type = multi select 的 customization 下，任一 option 未映射至少一个 sub step；检查范围含主 menu item 与所有关联 preset；customization type 限 `Mandatory Choice` / `Optional Addition`；忽略 Mandatory Choice 的 `none` 项；不论 `ineligible` 标记如何
  - item 详情页与 item grid 页都要展示
  - 原有两类 KDS portion 的 missing info 由 error 降级为 warning
  - 按 line build 分组的提示中增加第三类：`Multi select option(s) is missing mapped substep: …`

### [MD-18093](https://wonder.atlassian.net/browse/MD-18093)

UI - 切换 Display Style 到 In-Drawer 时清空 Featured 专有字段

- customization 的 `display style` 从 `FEATURED` 切到 `IN_DRAWER` 时，清空仅 Featured 模式适用的字段：`Free Choices`、`Custom Type`、`Display Options`
- 同时重置 `option_values` 上的相关字段：`default_portion` 置 null、`is_default_value` 置 false、`in_eligible` 置 null
- 从 `IN_DRAWER` 切回 `FEATURED` 时，按 customization type 恢复默认 `custom_type`（`MANDATORY_CHOICE` / `OPTIONAL_ADDITION` → `SINGLE_SELECT`，`DISH_PREFERENCE` → null），避免 Custom Type 选择框为空
- 目的是防止 In-Drawer 的 customization 残留过期的 Featured 模式数据

### [MD-18094](https://wonder.atlassian.net/browse/MD-18094)

UI - 支持手动重算 Thawed 40 item 的 nutrition

> 主 ticket：[MD-18055](https://wonder.atlassian.net/browse/MD-18055)

- 40 item 的 nutrition card 上显示 `Recalculate` 按钮，点击后重算该 40 item 的 nutrition
  - 若主 item 为 40 thawed 且其 nutrition 来源于配对的 frozen 40\*，则从 frozen 40 取最新 nutrition
  - 若主 item 为 40 thawed 且其 nutrition 是由自身关联的 WSKU 计算而来，则沿用既有的重算逻辑
- thawed 40 的 nutrition 来源标记（只能来自 frozen 40）与字段继承（含系统计算字段及手工标记的 `Hide in Wonder App`）由后端处理

### [MD-18099](https://wonder.atlassian.net/browse/MD-18099)

UI - 在 7\* item information card 上展示计算出的 net weight

> 主 ticket：[MD-17992](https://wonder.atlassian.net/browse/MD-17992)

**背景**：7\*（HDR_RECIPE）item 此前没有 net weight 的计算与展示（88\* 已有）。当 7\* 的 BOM component 里含有其他 7\* item 时，因为 7\* 的 BOM unit 是 `ea` 且 `ea` 与 `g` 之间没有换算，导致 component card 上的 `Total Weight` 算不出来。

- 在 7\* item information card 上展示计算出的 net weight
- 计算 7\* 的 total weight 时，使用子 component 7\* item 的 net weight 参与计算
- 配套修正 component 编辑页上 `Total weight` 与 `Total Yield` 的计算结果
- net weight 本身的计算逻辑（复用 88\* 的算法）在后端

### [MD-18126](https://wonder.atlassian.net/browse/MD-18126)

Wonder Create debug page：弹窗右侧增加 `Deselect` 按钮

- 在 `/wonder-create-debug` 页面中，选中若干 item 后，于弹窗右侧显示 `Deselect` 按钮，用于一键取消已选项

### [MD-18127](https://wonder.atlassian.net/browse/MD-18127)

UI - 支持手动覆盖 menu item 的 GF / GFO 标签

> 主 ticket：[MD-18121](https://wonder.atlassian.net/browse/MD-18121)

**背景**：GF（gluten-free）标签目前依据 BOM 中是否含小麦自动打上，但有些商品包装上带 "may contain wheat" 声明（如草莓芝士蛋糕），导致系统显示 GF 与实物警示不一致，已有客户投诉。

- menu item 的 nutrition card 上新增开关 `Hide GF and GFO from Wonder App?`，**默认 false**，可由用户手动开启
  - 提示文案：`Determine whether to show GF and GFO tag for Wonder App.`
- 该开关要纳入 change log
- copy new version、copy new item、menu item variant 都要带上这个 flag
- 开关为 true 时不再向 consumer app 返回 GF / GFO（GF/GFO 自身的计算逻辑不变）由后端处理

### [MD-18147](https://wonder.atlassian.net/browse/MD-18147)

Tech - UI - Node.js 升级到 24.17.0 并临时移除 ts check

- 前端构建环境的 Node.js 版本升级到 `24.17.0`
- 作为过渡，**临时**移除构建流程中的 TypeScript 检查（后续由 [MD-18192](https://wonder.atlassian.net/browse/MD-18192) 统一各前端 pipeline 的 Node 版本并恢复 `vite build` 的类型检查）
- ticket 描述仅为空模板，以上依据实现与后续 ticket 的衔接整理
