# Front-end Jira Tickets — Sprint 18

> Sprint：`MD 2026 Sprint 18`，周期 **2026-08-24 → 2026-09-07**
> 本次仅预读 MD-18391 / MD-18454 / MD-18455 三条。

| Key | 摘要 (Summary) | 前端工作 |
| --- | --- | --- |
| [MD-18391](https://wonder.atlassian.net/browse/MD-18391) | Display 'Unavailable' indicator for items without fulfillment in nutrition card | 小，1 个表格列 |
| [MD-18454](https://wonder.atlassian.net/browse/MD-18454) | Initialize Accounting Type & Accounting Sub-Type for 9\* items | 主体是后端数据初始化；前端仅在选项集合有变化时需改硬编码的选项树 |
| [MD-18455](https://wonder.atlassian.net/browse/MD-18455) | Add free text label/description box for line builds | 大，弹窗 + 两套 tab + change log |

## 前端需求拆解

### [MD-18391](https://wonder.atlassian.net/browse/MD-18391)

40\* item 的 Nutrition 卡片里，给不参与营养计算的 WSKU 打 `Unavailable` 标

> 主 ticket（Epic）：[MD-17501](https://wonder.atlassian.net/browse/MD-17501) Supply Chain Catalog Integration

**背景**：40 item 的营养值优先由可用的 41/42 WSKU 计算得出，取不到时才从 `40*F` 继承。但一个 42 item 可能同时满足「没有配置 fulfillment option」+「没有对应的 `W42F` 可继承」+「不是 dormant」——它不参与 40 的营养计算，却照样列在 Nutrition 卡片的 Linked WSKU 里。它自己的营养数据也不会被自动清空，于是用户会误以为这行有效。

**需求**

1. 在 Nutrition 卡片 `Linked WSKU` tab 中，于 item number 下方增加一个 `Unavailable` chip/badge，表明该 item 不能用于营养计算。

**前端关注点**

> - 40\* 走的是 HDRConsumable 详情页 —— `src/utils/siteMap/siteMapRoute.ts:40` 明确把 `40` 开头的 item number 路由到 `/ItemV2/hdr-consumable/detail/...`；41/42 才走 `wskus-consumables`。
> - 落点唯一：`src/page/HDRConsumable/detail/component/Nutrition/index.tsx:65` 定义 `Linked WSKUs` tab，表格在 `components/WSKUsTab.tsx`，第一列（`WSKUsTab.tsx:34-45`）就是 item number 链接，chip 加在这个 render 里。
> - 数据源是 `nutrition.sub_items`，类型 `GetItemHDRConsumableNutritionAJAXResponse$SubItem`（`src/type/api-part/ItemHDRConsumableItemAJAXWebServiceType.ts:136-146`）。**当前没有任何字段能表达「无 fulfillment / 无 W42F / dormant」**，现有的只有 `scc_wonder_product_status`，需要后端加一个标志位。
> - 同一页还有一个 Pack Size 卡片，锚点名也叫 `Linked WSKUs (N)`（`detail/hooks/useHDRConsumableAnchors.tsx:37`）。ticket 指的是 Nutrition 卡片里的 tab，不是这个卡片。
> - 只加 chip 展示，不联动卡片顶部已有的 warning（`Nutrition/index.tsx:59`，条件是 `!nutrition_fact || scc_wonder_product_status === INACTIVE`）—— 两者互相独立。行内的营养数值照常展示。

---

### [MD-18454](https://wonder.atlassian.net/browse/MD-18454)

按用户给的表格重置 9\* item 的 `Accounting Type` / `Accounting Sub-Type`

**背景**：Cookbook 里 9\* item 的 `Accounting Type`、`Accounting Sub-Type` 现有值已经过时，需要按最新的对照表重新初始化。

> **定性**：这是一条后端 ticket。数据初始化、清废弃值、change log 全在后端。前端唯一的触点是需求第 2 条「更新选项」—— 且仅当新对照表的 Type/Sub-Type 集合或配对与现状不一致时才需要动（原因见下方关注点）。

**需求**

1. 初始化文件：[Google Sheet](https://docs.google.com/spreadsheets/d/1MvPf5-onJ6jzMlTcFP13DeIqOK-KdsBXd5I4p_VCTC0/edit?gid=0#gid=0)
2. 更新 `Accounting Type` / `Accounting Sub-Type` 的选项
   1. `Categories` sheet 的 B 列 `Main Accounting Category` = Accounting Type，E 列 `SUB accounting category` = Accounting Sub-Type
   2. 现存选项中不在初始化文件里的，要 flag 出来给用户判断是否废弃；flag 的信息为 Accounting Type、Accounting Sub-Type、Applied to Items（Applied to Items 忽略 dormant 的 9\*）
3. 为所有 9\* item 初始化这两个字段
   1. 值已被废弃的 9\* item，清空该值
   2. 只初始化 sheet 2 `SKU List` 和 sheet 3 `Add to SKU List` 里列出的 9\*
   3. 两个字段都是单选，所以 sheet 2 / sheet 3 里重复出现的 9\* 要 flag 出来
   4. 变更要记进 change log

**前端关注点**

> - 9\* = non-food item。编辑入口是 `src/page/item/detail/pages/editNonFood/index.tsx:253-258`：**一个 antd `Cascader`**，label 为 `Accounting Type & Accounting Sub-Type`，两级即 Type → Sub-Type。
> - **选项树是前端硬编码的**：`src/page/item/detail/pages/editNonFood/accountingTypeOption.ts`，7 个 Type × 各自固定的 Sub-Type 列表（`accountingTypeOption.ts:41-47`）。枚举值本身来自后端（`AccountingTypeAJAXView` 7 项 / `AccountingSubTypeAJAXView` 14 项，`src/type/api-part/ItemAJAXWebServiceV2Type.ts:780-803`），但**后端枚举里没有父子关系**，Type→Sub-Type 的合法配对只存在于这个前端文件里。
> - 展示名同样在前端：`accountingTypeOption.ts:23` 的 `accountTypeMap` + `firstLetterCapital` 兜底。只读展示 `ERP/pageComponent/BasicInfo/component/OverviewTab.tsx:174-180`、change history `changeHistory/widget/ERPInformation.tsx:71-72` 都复用它。
> - 所以「更新选项」= 后端改枚举 + `pnpm api` + **手工重写** `accountingTypeOption.ts` 的配对树和 `accountTypeMap` 的展示名，三处缺一不可。
> - 需知情：如果后端把某个枚举成员整个删掉，历史版本 change history 里那个旧值会失去映射，退化成 `OPERATING_SUPPLIES_MULTI_USE` 这种原始大写下划线串。
> - 顺手可修：`accountTypeMap` 现有一处拼写错误 —— `"Custome-facing Disposables (Orde-and Dish- level)"`（漏了两个 `r`）。

**疑问**

- 「flag 出不在初始化文件里的旧选项供 review」的交付形态是什么？一次性跑个清单发给 PM，还是 Cookbook 里要有个界面/导出？这决定了这条 ticket 前端到底有没有活。
- Type→Sub-Type 的合法配对目前只存在于前端 `accountingTypeOption.ts`。`Categories` sheet 的 B/E 两列配对，是不是就是完整且权威的来源？（若初始化写进去的组合在前端选项树里不存在，用户一进编辑页 Cascader 就选不出来，保存即丢值）

---

### [MD-18455](https://wonder.atlassian.net/browse/MD-18455)

给 line build 增加自由文本的 `Label` / `Description`

**背景**：一个 menu item 现在可以有多个 line build，但用户没有直观手段区分它们，管理多套 line build 配置时容易混淆。

**需求**

1. 增加一个 `Line Build Label` 入口：在每个 line build 的 kebab（⋮）菜单里，作为**第一项**，排在 `Export to JSON` 之上
2. 点击后弹窗，字段如下：
   1. Header：`Add Line Build Note`
   2. `Label`：选填，自由文本，上限 5000 字符
   3. `Description`：选填，自由文本，上限 5000 字符
   4. 操作：`Cancel`、`Save`
   5. 超长时内联报错
   6. 成功提示：`Successfully save the line build Note.`
3. Export / Duplicate / Copy from other line build 时**不带** `Label` 和 `Description`
4. hover `Line build #` tab 时展示 `Label`
5. `Description` 展示在 line build header 信息块的**最后一行**（ticket 原文分三种情况写：单版本时在 `Apply to restaurants` 下方，其余在 `Option Value` / `Selected Option value amount` 下方 —— 对照设计稿，三种情况都等价于「排在最后」）
6. 没有 description 时，不展示 `Description` 这一行
7. `Label` 和 `Description` 要进 change log

**前端关注点**

> - **现在有两套 line build tab**（MD-18401 加的新 tab 与旧 tab 并存），header 区块是各写各的，要改两处：
>   - 旧 tab：`src/page/item/detail/pages/lineBuildList/component/LineBuildTable.tsx:86-125`
>   - 新 tab：`src/page/item/detail/pages/lineBuildListV2/component/LineBuildPanel.tsx:95-110`
> - **两张设计稿都画在旧 tab 上**（有 `Create New Line Build` + `Configuration`，tab 标题为 `Line Build 1/2` 无 `(All)` 后缀，kebab 里也没有 `Copy from other line build`）。
> - header 信息块现有三个条件行（`LineBuildTable.tsx:103/111/121`）：`Apply Restaurants` 恒显示；`Apply to Option` 在 `is_multiple_usage || is_multiple_version` 时显示；`Selected Option value amount` 仅 `is_multiple_usage`；`Option Value` 仅 `is_multiple_version`。ticket 分三种情况描述 `Description` 的位置，**三种情况都落在最后一行**，所以实现上不需要按 flag 分支，直接把 `Description` 追加到 Stack 末尾即可（设计稿第二张即 `is_multiple_usage` 场景，`Description` 就在 `Selected Option value amount` 之下）。
> - tab caption 也是两处：旧 `lineBuildList/component/index.tsx:70`（`Line Build ${index + 1}`）、新 `lineBuildListV2/component/lineBuildLabel.ts:6`（`Line Build N(All)`）。
> - **命名冲突**：`lineBuildLabel.ts` / `getLineBuildLabel()` 已经被占用，指的是 tab 上的标题文字，跟本次用户输入的 `Label` 不是一回事，实现时要么改名要么另起。
> - `Line Build Label` 入口在 kebab 菜单的第一项：旧 `LineBuildTable.tsx:34-82`（现为 Export / Duplicate / Delete），新 `LineBuildPanel.tsx:58-86`（现为 Training Card / Export / Copy from other line build / Duplicate / Delete）。
> - **Export 是后端生成的**：`/recipe-site/item/version/{version}/line-build/{id}/export-json`（`LineBuildTable.tsx:41`、`LineBuildPanel.tsx:68`），前端只是下载，"export 不带 Label/Description" 只能后端做。
> - Duplicate / Copy-from 是前端行为，要显式清掉：copy-from 走 `CopyFromOtherLineBuild` + `src/page/item/lineBuild/utils/detachLineBuildRecordIds.ts`（该函数目前只清 4 个 id 字段，不动业务字段）；duplicate 走 `getDuplicateUrl` / `checkCreateLineBuild` 进编辑器。
> - Import JSON（`lineBuild/component/Header/ImportFile.tsx`）不用单独处理：export 既然不带这两个字段，导入的文件里就不会有。
> - change log 落点：`src/page/item/detail/pages/changeHistory/widget/LineBuild.tsx` 的 `ListLineBuild`（`:53` 起的 `FieldRow` 列表），后端需要把两个字段加进 `LineBuildCardAJAXView$LineBuild`；高亮路径在 `changeHistory/useGetStyle.tsx` 里对 `line_build_card_line_builds` 有专门分支，新字段要一并照顾。
> - 文案上同一个东西出现了三种叫法：菜单 `Line Build Label`、弹窗标题 `Add Line Build Note`、成功提示 `line build Note`。实现时按 ticket 原文照抄。

**疑问**

- 设计稿画在旧 tab 上。MD-18401 新加的那个 Line Build tab 要不要一起做？（两套 header 和两套 tab caption 是分开的代码，工作量差一倍）
- `Label` 上限 5000 字符，但唯一的展示位置是 tab hover 的 tooltip —— 需要给一个截断长度或展示上限，否则一个长 Label 会糊满整屏。
