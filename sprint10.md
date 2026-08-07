# Front-end Jira Tickets — Sprint 10

> Sprint：`MD 2026 Sprint 10`，周期 **2026-05-06 → 2026-05-18**

| Key | 摘要 (Summary) | 类型 |
|-----|----------------|------|
| [MD-17682](https://wonder.atlassian.net/browse/MD-17682) | UI - Optimize item detail page loading performance | Story |
| [MD-17750](https://wonder.atlassian.net/browse/MD-17750) | UI - Adjustments according to SCC WSKU Item/WSKU UOM Status/Outbound Date | Sub-task |
| [MD-17753](https://wonder.atlassian.net/browse/MD-17753) | UI - Restore Ingredient Data Granularity: 40\* Item Name Mapping & VendorSKU Updates | Sub-task |
| [MD-17755](https://wonder.atlassian.net/browse/MD-17755) | UI - Add SCC URL/Tip for WSKU in Cookbook View | Sub-task |
| [MD-17757](https://wonder.atlassian.net/browse/MD-17757) | UI - Enable SCC WSKU Item Number for View/Search | Sub-task |
| [MD-17786](https://wonder.atlassian.net/browse/MD-17786) | UI - Tech upgrade core-fe then move logger to datadog | Story |
| [MD-17868](https://wonder.atlassian.net/browse/MD-17868) | UI - Enable Max Options for In-drawer Customization | Sub-task |
| [MD-17852](https://wonder.atlassian.net/browse/MD-17852) | 40 item 详情页：在 Linked WSKUs tab 为 dormant 41 item 显示 `Dormant` chip | Sub-task |
| [MD-17871](https://wonder.atlassian.net/browse/MD-17871) | UI - Translate Logistics Card UOMs to base UOMs for Vendor Catalog | Sub-task |
| [MD-17926](https://wonder.atlassian.net/browse/MD-17926) | CLONE[UI] - Enable to Configure Wonder App Ingredients on Byproduct 6\* Item | Sub-task |

---

## 前端需求拆解

### [MD-17682](https://wonder.atlassian.net/browse/MD-17682)

UI - 优化 item 详情页的加载性能

> 主 ticket（Epic）：[MD-17264](https://wonder.atlassian.net/browse/MD-17264) @2026 Regular optimization in Cookbook

- ticket 描述仅为空模板，未列具体条目；实际范围为 item 详情页的加载性能优化
- 同 Epic 下的相关优化见 sprint11.md 的 MD-17949（移除详情页多余的 concept list / restaurant 初始化请求）

### [MD-17750](https://wonder.atlassian.net/browse/MD-17750)

UI - 按 SCC 的 WSKU / WSKU UOM 状态与 outbound date 调整展示

> 主 ticket：[MD-17659](https://wonder.atlassian.net/browse/MD-17659)

**背景**：SCC 侧可以 deprecate 一个 WSKU 或 deactivate 某个 WSKU UOM，Cookbook 需要跟随这些状态调整展示。

- **40\* item 详情 → Linked WSKUs tab**：展示**所有**关联的 WSKU item
  - SCC WSKU 为 inactive 时，在 ID 旁显示 `Inactive` chip（沿用该 tab 里 dormant Cookbook WSKU 的 UI）
  - 隐藏 `Active for Ordering Start Date`、`Active for Ordering End Date` 两列
  - 移除 `Active for Ordering Only` toggle
- **W42\* item 详情 → Fulfillment Option**
  - 移除 `Active for Ordering Only` flag
  - 隐藏 `Active for Ordering Start Date`、`Active for Ordering End Date` 两列
  - buyout vendor SKU 也要展示
  - 按 `Active for Ordering = false` 的当前状态展示 vendor SKU：排除已删除的 88\* item；WSKU UOM 为 inactive 时，其关联的 vendor SKU 不展示；关联的 88\* 为 dormant 时显示 dormant chip；关联的 buyout vendor SKU 为 inactive 时显示 inactive chip
- `active for ordering` 不再默认为 true，改为按「W42 item status=active + WSKU UOM active + 至少一个可用 vendor SKU」计算（计算在后端）

### [MD-17753](https://wonder.atlassian.net/browse/MD-17753)

UI - 恢复 ingredient 数据粒度：40\* item name mapping 与 vendor SKU 更新

> 主 ticket：[MD-17529](https://wonder.atlassian.net/browse/MD-17529)

- 该 sub-task 自身无描述，需求以主 ticket 为准；同一主 ticket 下的另一项前端改动见 MD-17926（byproduct 6\* 的 Wonder App Ingredients 配置）

### [MD-17755](https://wonder.atlassian.net/browse/MD-17755)

UI - 在 Cookbook 中为 WSKU 增加跳转 SCC 的入口

> 主 ticket：[MD-17651](https://wonder.atlassian.net/browse/MD-17651)

- **WSKU 详情页**（`SCC Source = true`）：显示超链接 `SCC View`，点击在**新标签页**打开 SCC 中对应的 WSKU 详情页
- **40\* item 详情 → Linked WSKU 表格**：在 `TYPE/SUBTYPE` 列的 WSKU 旁显示超链接 `SCC View`，同样新标签页跳转
- SCC 地址按环境区分：
  - QA `http://supplychain.foodtruck-qa.com/product-catalog/supply-chain-catalog`
  - UAT `https://supplychain.foodtruck-uat.com/product-catalog/supply-chain-catalog`
  - PROD `https://supplychain.remarkablefoods.net/product-catalog/supply-chain-catalog`
- 原方案里「在 toggle 下方显示 active for ordering 日期来源说明」的提示文案已划掉，不做

### [MD-17757](https://wonder.atlassian.net/browse/MD-17757)

UI - 支持按 SCC WSKU item number 查看与搜索

> 主 ticket：[MD-17655](https://wonder.atlassian.net/browse/MD-17655)

- **WSKU grid**（Cookbook / PCS）中，若 WSKU 的 `SCC Source = true`：item status 与 version status 不展示实际值，统一显示为 `--`
- 搜索框 placeholder 改为 `Search by Name or Item Number without W`

### [MD-17786](https://wonder.atlassian.net/browse/MD-17786)

UI - 升级 core-fe 并把日志上报切到 Datadog

> 主 ticket（Epic）：[MD-17231](https://wonder.atlassian.net/browse/MD-17231) @2026 Cookbook Technical Excellence

- 升级 `@wonder/core-fe` 到 `2.4.9`，使 cookbook 与 pcs 的 axios 版本 `>= 1.15.0`
- 改造日志代码以支持 Datadog 上报（原有 logger 迁移到 Datadog 以便日志分析）

### [MD-17868](https://wonder.atlassian.net/browse/MD-17868)

UI - 为 In-Drawer 类型的 customization 支持配置 Max Options

> 主 ticket：[MD-17799](https://wonder.atlassian.net/browse/MD-17799)

**背景**：顾客点两份 side dressing 时一个碗装不下，merch 团队希望能对 Optional Add-Ons 设数量上限，Consumer 端据此显示 `Choose up to 1`。

- 当 customization 的 `display style = In-Drawer` 且 type 为 `Optional Addition` 或 `Extra Request` 时，显示 `Max Options` 字段
  - 在 customization **group 层级**配置，非必填
  - 沿用已有的 max options 校验：max options 不得超过该 customization 的 option 数量；必须 `> 0`；删除 option 时需校验 max options `<=` 剩余 option 数
- customization 详情页显示 `(Max: #)`，为 null 时不显示
- Change Log 也要加上 max options 字段

### [MD-17852](https://wonder.atlassian.net/browse/MD-17852)

40 item 详情页：在 `Linked WSKUs` tab 为 dormant 的 41 item 显示 `Dormant` chip

- 40 item 详情页的 `Linked WSKUs` tab 中，若关联的 41 item 为 dormant，在其旁显示 `Dormant` chip
- 与 MD-17750 中「SCC WSKU 为 inactive 时显示 `Inactive` chip」是同一套 chip 展示规则
- 注意仅在 40 item 的 `scc source = true` 时才在 `ID` 列展示该 chip

### [MD-17871](https://wonder.atlassian.net/browse/MD-17871)

UI - 为 88\* item 增加 Pack Size per PK 字段（配合 Vendor Catalog 的 base UOM 换算）

> 主 ticket：[MD-17866](https://wonder.atlassian.net/browse/MD-17866)

**背景**：Logistics Card 用包装关系定义 UOM（如 1 CS = 12 EA），而 Vendor Catalog 需要按 item 的 `base_uom` 表达换算系数。翻译层本身在后端完成，**Logistics Card UI 不需要改**；前端要做的是主 ticket 中 Bonnie 补充的字段部分。

- 为 88\* item 新增 `Pack Size per PK` 字段（与 base UOM 搭配）
  - `Pack size qty per PK` 与 `Base UOM` 任一非空时，两者都变为必填；都为空时均可选
  - inline error：缺数量提示 `Missing qty`，缺单位提示 `Missing unit`
- `Pack size qty per PK` 始终可手工编辑（无论 88\* 的 `sync to VCS` 状态）；`Base UOM` 一旦发布即不可再改
  - 修改需同时作用于 active version、future version 与 variant
  - variant 中不可编辑 `Pack size qty per PK` 与 `Base UOM`
  - variant 复制为 draft version 时，这两个字段取自该 88\* item 的 normal version
- 配套展示调整：`item information` card 中当 `Pack Size qty per pk` 为 null 时，在 `Pack Size per PK` 旁显示 base unit 值

### [MD-17926](https://wonder.atlassian.net/browse/MD-17926)

CLONE[UI] - 支持在 byproduct 6\* item 上配置 Wonder App Ingredients

> 主 ticket：[MD-17529](https://wonder.atlassian.net/browse/MD-17529)

- **byproduct item 的 nutrition card 展示态**：在 `Wonder App Name` 下方显示 `Wonder App Ingredients`（沿用 vendor SKU 的既有 UI）
  - 每个值以 chip 形式展示
  - 最多显示 4 个 chip，超出时在最后一个 chip 后显示 `(+#)`（`#` 为剩余数量），hover 展示全部
- **byproduct item 的 nutrition 编辑弹窗**：在 `Wonder App Name` 下方显示 `Wonder App Ingredients`，支持为 byproduct item 打标（沿用 vendor SKU 的既有 UI）
- 88\* item 汇总 wonder app ingredients 时，需**包含**该 88\* 中使用的 byproduct component
- 配套的字段展示调整：`cookbook-enable-wonder-app-name-derivation` flag 开启时，byproduct item 详情页隐藏 `Wonder App Name`；`Nutrition Data` 与 `Serving Size` 合并展示；`Wonder App Ingredients` 与 `allergens` 的上下位置按设计稿调整
