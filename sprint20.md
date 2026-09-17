# Front-end Jira Tickets — Sprint 20

> Sprint：`MD 2026 Sprint 20`，周期 **2026-09-21 → 2026-10-05**

Sprint 20 共 5 条 story，其中 `MD-18390`（dormancy job 排除 Wonder Create concept）与 `MD-18551`（40* 有 40F 时抑制 missing vendor SKU 告警）是纯后端/告警逻辑，不涉及前端，未纳入本文档。

| Key | 摘要 (Summary) | 类型 | 前端工作 |
| --- | --- | --- | --- |
| [MD-18564](https://wonder.atlassian.net/browse/MD-18564) | Support Sync To ERP WSKU Items On WSKUs & Consumables Pages | Story | 大（新建整套批量选择 + 同步流程） |
| [MD-18588](https://wonder.atlassian.net/browse/MD-18588) | Separate Wonder Create (WC) Items In Cookbook | Story | 中（4 个页面过滤 + chip） |
| [MD-18574](https://wonder.atlassian.net/browse/MD-18574) | Align Linebuild Export/Import Field Copying With Duplicate Logic | Story | 小（可能仅 1 处 inline error） |

## 前端需求拆解

### [MD-18564](https://wonder.atlassian.net/browse/MD-18564)

WSKUs & Consumables 页支持批量 Sync to ERP


**背景**

Finance 需要 Cookbook 把 40/42 item 同步到 ERP。Item Grid 页已有 Sync to ERP，但 42* item 单独放在 WSKUs & Consumables 页，没有入口。这类 item 同步失败后用户无法自助重试，只能等 Cookbook 工程师处理。

**需求**

1. WSKUs & Consumables 页新增 `Sync to ERP` 按钮，位置在 filter 行末尾（`All Filters` / `Status` / `HDR Consumable` 之后）
   - 允许同时选 41* 和 42*
   - dormant item 的 checkbox 置灰，tip 文案改为 `Dormant item cannot be selected`
   - 提交后若没有任何 item 可同步（只同步有 final version / SCC 状态为 active 的 41*/42*），报 `Unable to sync to ERP. No available item for syncing to ERP (only sync 41*/42* item which has final version/SCC status is active).`
2. 点按钮后才出现 checkbox 列
   - 勾选任意 item 弹出底部条：`{n} item selected` + `Sync Items to ERP` 按钮，**底部条只有这一个按钮**
   - 点 `Sync Items to ERP` 触发同步
   - 点 `x` 清空所有勾选并关闭底部条；取消全部勾选亦自动关闭
   - 列头 checkbox 全选
   - 底部条出现后禁用页面动作：搜索框、`Search`、`Clear`、`All Filters` 及各个 filter 下拉、`Show all filters`、`Columns`、`Create New`、`...`
3. 提交成功提示：`Syncing items to ERP is in process, please see the final result in the Sync Log Job or slack channel 'md-erp-sync-alerts' for final result later.`

**前端关注点**

> **WSKU 列表目前完全没有行选择能力。** `src/page/WSKU/list/state.ts` 的 `State` 只有 `pagination / items / searchRequest / total`，没有 `selectedRowKeysData / selectedRowsData`；`src/page/WSKU/list/components/ItemTableDS/index.tsx` 的 `ColumnConfigMap` 也没有 checkbox 列。这套要照 item grid 新建。

> **item grid 的零件可以照搬，但底部条要抽精简版而不是整体复用 `BulkBar`。** 设计稿里 item grid 底部条上除 `Sync Items to ERP` 以外的按钮（Add to Menu、Add to List、Recipe Export、ERP Item Export）全部被划掉。现有 `BulkBar`（`src/page/item/list/components/BulkBar/index.tsx`）把这些按钮和 `ItemV2_NEW` store 焊死在一起，WSKU 页需要一个只含单按钮的新壳子。
> - checkbox 列是手写的 Custom 列 `checkboxGroup`（`src/page/item/list/components/ItemTableDS/index.tsx:93-131`），不是 antd `rowSelection`；dormant 用 `disabled` + `ShowPopover` 包住
> - 底部条的容器和显隐 = `widget/FixedBar` + `useFixedContext()`（`src/widget/FixedBar/useFixedContext.tsx`），`{n} item(s) selected` 文案与 `CloseOutlined` 清空逻辑在 `BulkBar/index.tsx:82-89,350-388`，这部分可抄
> - 需求 2.f「禁用页面动作」已有现成套路：`useFixedContext().visible` 直接喂给控件的 `disabled`，item grid（`CreateItemButton/index.tsx:239-240`、`SearchFormNew/components/ItemStatus.tsx:117`）和 attributes 管理页（`attributesManagement/list/components/SearchFormNew/index.tsx:104`）都这么做
> - 同步动作与报错分支复用 `SyncItemToERP` 组件（`src/page/item/list/components/SyncItemToERP/index.tsx`）：它已经是「后端全拒 → `message.error` / 部分失败 → `message.warning` 列出失败项 / 全成功 → `message.success`」三分支，成功文案与需求 3 逐字一致，只需替换全拒分支的文案

> **需求 1.c 不是「选中 41* 就报错」，而是现有的「全拒分支」。** 设计稿里 41* item（如 `41059211`）确实和 42* 混在同一张列表里，可以勾选；`SyncItemToERP/index.tsx:39-41` 现有逻辑就是 `res.in_progress_item_numbers.length === 0` 时报这句。41* 通常没有 final version，所以自然落到这个分支 —— 前端不需要按 item number 前缀做特判。

> **需求 2「点按钮才显示 checkbox 列」是新交互，item grid 没有。** item grid 的 checkbox 列常驻，这里要额外加一个「列可见性」开关态。

> **列头全选在 item grid 是「全选所有搜索结果」而非当前页**（`getAllSelected`，`ItemTableDS/index.tsx:70-84`），并非只勾当前 25 行。设计稿里列表是 `Showing 1-20 of 2008 results`，全选语义需要留意。

> **41* 在 Cookbook 已被判定废弃**：`src/page/WSKU/list/components/SearchFormNew/index.tsx:234` 留着注释 `MD-18485: 41* items are deprecated in Cookbook, so the Create WSKU entry is hidden for now`，Create WSKU 入口已隐藏。

**疑问**

- 需求没写单次同步条数上限，但 item grid 有硬上限 50（`SyncItemToERP/index.tsx:20`，超了报 `Sync failed. Please select a maximum of 50 items to sync.`）。配上「列头全选 = 全部 2008 条搜索结果」，WSKU 页是否沿用同一个 50 上限与同一句报错？
- 权限码沿用 item grid 的 `ITEMS_ITEM_GRID_SYNC_ITEM_TO_ERP`，还是要为 WSKU 页新建一个？沿用的话，只有 item grid 权限的人也会拿到这个新按钮。
- 41* 已被 MD-18485 判为废弃，是否索性和 dormant 一样直接禁选，而不是让用户勾完再吃一个报错？

### [MD-18588](https://wonder.atlassian.net/browse/MD-18588)

Cookbook 中把 Wonder Create item 与普通 item 区分开

> 主 ticket（Epic: Wonder Create Integration）

**背景**

CDT 希望把 Wonder Create（WC）item 从普通 item 里分离出来。目前 WC item 与普通 item 混在一起，难以识别和单独管理。

**需求**

1. Items grid / Scheduled Changes / Items' attributes list / Line Build Cooking Groups 四个页面默认排除 WC item
   - 加一个 `Show Wonder Create Menu Items` 开关，默认不选。Item grid 放在 Show all filters 的 Search Items 弹窗 `General` 区末尾（`Sold Status` 之后、`Recipe` 区之前）；Scheduled Changes 的 `Items` tab 放在页面 filter 行右侧、`Clear Filters` 左边
   - 按 `Isexternal = true` 排除（即由 Wonder Create Portal 经 API 创建的）
   - template 默认仍然显示 —— template 是手工创建的，`Isexternal = false`，CDT 需要维护 template 数据
2. 在 menu item 名称末尾追加 `Wonder Create` chip
   - template 和 `Isexternal = true` 的 menu item 都要加
   - 加在：Item grid 页、Usages、Item details 页、Items' attributes list
   - item detail 上的位置是 name 行最末尾、item status badge（如 `Active`）之后

**前端关注点**

> **四个页面走四套不同的搜索接口，开关形态也不统一 —— 只有 Item grid 有 Search Items 弹窗，其余是页面级 inline 开关。**
> - Item grid → `ItemAJAXWebServiceV2.search`，`formList` 是数据驱动配置（`src/page/item/list/components/SearchFormNew/index.tsx:211+`，加一项 `categories: "General"` 即可；`FormBase` 已支持 `type: "checkbox"` / `"radio"`，见 `src/widget/FormBase/index.tsx:62,71`）
> - Scheduled Changes 的 `Items` tab → `ScheduleAJAXWebServiceV2.searchBomHeader`（`src/page/scheduledChangesV2/Item/page.ts:37`）。该页 tab 定义在 `scheduledChangesV2/Entry.tsx:17-18`（`Items` / `Change Tickets`），需求指的是 `Items` tab；它的筛选是列头 inline filter，没有 `formList` 弹窗，开关要插进 `Item/List/index.tsx:438-452` 那个右对齐 `Flex`（现有 `Clear Filters` + `ColumnPickerSelect` 就在这里）
> - Items' attributes list → `ItemAttributeValueAJAXWebServiceV3.searchV3` / `searchUsageV3`（`src/page/attributeV2/itemAttribute/page.ts:40,81`）
> - Line Build Cooking Groups → `ItemAttributeValueAJAXWebServiceV3.searchCookingGroupUsage`（`src/page/attributeV2/lineBuildCookingGroups/page.ts:32`）

> **后两页的开关位置建议直接定为「搜索表单末尾的 checkbox」。** 这两页用的是和 Item grid 同一个 `FormBase` 组件，只是以 `type="SearchVertical"` 内联渲染、没有 `categories` 分组（`itemAttribute/component/List.tsx:52,220`、`lineBuildCookingGroups/component/List.tsx:33,115`），两页都已有 `Clear` / `Search` 按钮。`FormBase` 本身支持 `type: "checkbox"`（`widget/FormBase/index.tsx:71`），所以各加一条 `formList` 条目即可，与 Item grid 的做法同构。attributes 管理页的 `include_deprecated` 就是这个模式的现成先例（`attributesManagement/list/components/SearchFormNew/index.tsx:47`）。

> **设计稿画的是圆圈 radio（`◯ Show Wonder Create Menu Items`），但语义是可反复开关的布尔值。** 单个 antd `Radio` 点选后无法取消，实现上得用 `Checkbox` 或两项 `Radio.Group`。

> **排除 WC item 这件事在 usage 侧已经做过（MD-18542）。** `exclude_wonder_create_item` 已是 usage 类接口的请求字段（`ItemUsage/BOMUsage/page.ts:49`、`CustomizationUsage/page.ts:59` 等），当时的结论是：判据 `is_wonder_create_item` 基于 external_id，**template 恒为 false 所以排不掉**。本需求 1.3 正好把那个「排不掉」定为期望行为，两边一致。但 usage 侧是硬编码 `true` 无开关，item grid 这次要给开关，两种形态会并存。

> **chip 的判据和排除的判据不是同一个**：排除只看 `Isexternal`（template 保留），chip 要「template + Isexternal=true」都加。现有 `is_wonder_create_item`（`src/type/api-part/ItemAJAXWebServiceV2Type.ts:242,264`）对 template 为 false，另有 `is_wonder_create_template`（`ItemDetailTitle/useVariantItem.tsx:125`）。chip 需要 `is_wonder_create_item || is_wonder_create_template` 两个字段同时出现在列表响应里。

> **名称旁挂 chip 有现成范式**：`FrozenThawedStateChip`（`src/page/item/list/components/ItemTableDS/helper.tsx`）已在 38 处调用，item grid 的 Name 列就是 `<FrozenThawedStateChip … /> {getLink()}`（`ItemTableDS/index.tsx:217`）。新 chip 照这个写即可，`short` 参数的取舍也可参考。

> **item detail 的落点很明确**：`ItemDetailTitle/index.tsx:85` 渲染 `{name} ({item_number})`，:89 紧接 `<ItemStatusWithIcon status={item_status} />`，chip 接在其后即可。注意设计稿里 chip 是描边方形样式，与其下那一排蓝色 attribute chip 不是同一种。

**疑问**

- 设计稿只给了 Item grid 和 Scheduled Changes 两处开关的位置，且两处形态不同（弹窗内 vs 页面 filter 行）。剩下的 Items' attributes list 和 Line Build Cooking Groups，按上面的建议放在各自搜索表单末尾做成 checkbox 可否？还是这两处**默认排除、不提供开关**？
- chip 只加在需求列的 4 处，还是所有渲染 item name 的地方？对照 Frozen/Thawed chip 铺了 38 处（component picker、customization 选项、Add Component 搜索表等），只做 4 处大概率会被当成漏做报 bug。

### [MD-18574](https://wonder.atlassian.net/browse/MD-18574)

Line build 的 export/import 字段对齐 Duplicate（补 IK Dish Type）

> 主 ticket（Epic: [KDS Support] Line Build & Appliance Settings support）

**背景**

用户反馈：导出再导入 line build 时，subtask 上的 IK Dish Type 丢失。原因是 export/import 一度被认为是即将废弃的低频功能，后续新增的字段只同步进了 Duplicate，没有同步进 export/import。但用户确认仍然需要 export/import —— menu item 有多个版本，而 Duplicate 只能在同一个 menu item 内复制，跨版本搬 line build 只能靠 export/import。

**需求**

1. 让 export/import 与 Duplicate 拷贝同样的字段，包含 subtask 上的 IK Dish Type，以及其他 Duplicate 已正确处理而 export/import 漏掉的字段与逻辑
2. export/import 补字段
   - task 层：IK Dish Type、Cooking Groups
   - sub step：KDS portion flag —— 若 KDS portion 与目标 menu item 匹配则保留该值，否则在 KDS portion flag 上显示 inline error
3. Duplicate：把 Breaking Line 一起复制到新 line build

**前端关注点**

> **export、import、duplicate 三条路径的字段拷贝全部在后端。**
> - Export = 后端下载接口，前端只有一个 `Download` 按钮：`GET /recipe-site/item/version/{version_uuid}/line-build/{id}/export-json`（`src/page/item/detail/pages/lineBuildList/component/LineBuildTable.tsx:53`，新 UX 在 `lineBuildListV2/component/LineBuildPanel.tsx:78`）
> - Import = 上传到后端解析：`POST /recipe-site/item/version/{versionId}/line-build/import-json`，返回 `ImportItemLineBuildJSONResponseV2`；前端只把响应灌进表单（`Header/ImportFile.tsx:81,89-148`）
> - Duplicate = 后端复制：`checkCreateLineBuild(sourceLineBuildId)` 后跳编辑页（`LineBuildTable.tsx:60-69`）

> **后端把字段加进响应后，task 层的两个字段前端大概率零改动。** `initFormProcedures` 用 `produce` 原样保留 task 对象，`renderTasks` 也是 `{...task}` 透传（`src/page/item/lineBuild/utils/getProcedures.ts:23-41`），没有字段白名单。IK Dish Type（MD-18150）与 Cooking Groups（MD-18175）的 task 层表单项已经存在。

> **缺口已定位到类型上**：`ImportItemLineBuildJSONResponseV2$Task`（`src/type/api-part/common.ts:488-494`）只有 `id / name / customization_option / procedures / warning_message`，确实没有 IK Dish Type 和 Cooking Groups；`ImportItemLineBuildJSONResponseV2$ProcedureStep`（同文件 532-549）也没有 KDS portion flag（编辑器里该字段是 `is_cook_readable_qty_selected`，见 `Header/saveHelper.tsx:886`）。

> **需求 3 的 Breaking Line 在 import 侧其实已经通了**：`ImportItemLineBuildJSONResponseV2$Procedure.has_breaking_line` 已存在（`common.ts:526`），且 `renderTasks` 会把它还原成 `STICK_LINE` 伪行（`getProcedures.ts:28-33`）。所以需求 3 只是后端 Duplicate 要补。

> **import 的 inline 提示已有整套现成机制**：后端按 task / procedure / step 三级返回 `warning_messages`，前端就地渲染并在用户改动后自动清除 —— `TaskInfoNOptions.tsx:35-36,127`、`LineBuildRow/Options/useOptions.tsx:18-24,131`、`SubSteps/StepOptions.tsx:28-31,157`。目前枚举只有 `ITEM_NOT_FOUND / CUSTOMIZATION_OPTION_NOT_FOUND / CUSTOMIZATION_OPTION_VALUE_NOT_FOUND` 三项（`common.ts:483-487`）。

> 另需注意：KDS portion 目前已有一套**保存时**的警告弹窗（MD-18030，`saveHelper.tsx:826,897,969`，用前端字段 `missing_multi_select_kds_portion`）。本需求的「inline error」与它是两个不同机制，不要混。

**疑问**

- 需求 2.2 的 inline error 走哪条契约？复用现有 import 的 `warning_messages` 枚举加一项（例如 `KDS_PORTION_NOT_FOUND`，前端照 `StepOptions.tsx` 的现成模式渲染 + 编辑后自动清除，改动极小），还是后端另开一个字段？
- 这条 ticket 是否需要前端 sub-task？按代码，需求 1、2.1、3 全部落在后端（export 下载、import 解析、duplicate 复制都在后端），前端唯一可能的活是需求 2.2 的 inline error，而它若复用 `warning_messages` 也只是加一个枚举分支。

## 设计稿仍缺的部分

设计稿已确认的要点都已并入上面各条需求。剩下唯一没画的是 MD-18588 里 Items' attributes list 与 Line Build Cooking Groups 两页开关的位置 —— 但这两页的搜索表单结构已查清，上面给了建议落点，会上确认即可，不必专门出图。

MD-18564 的报错（需求 1.c）设计稿只截了页面现状、没有弹窗形态，按现有 `message.error` 实现即可，同样不需要补图。

## 会上待确认

- MD-18574 是否拆前端 sub-task —— 结论直接决定 Sprint 20 的前端容量
- MD-18588 的 chip 范围（4 处 vs 全量）需要在拆 sub-task 前定，否则体量差一倍
- MD-18588 剩两个页面的开关位置（已给建议方案，只需确认要不要给开关）
- MD-18564 的 dormant tip 文案是否连带改 item grid，属于一句话决定、但不定就会留不一致
