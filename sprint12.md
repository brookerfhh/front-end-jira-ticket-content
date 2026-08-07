# Front-end Jira Tickets — Sprint 12

> Sprint：`MD 2026 Sprint 12`，周期 **2026-06-01 → 2026-06-15**

| Key | 摘要 (Summary) | 类型 |
|-----|----------------|------|
| [MD-17938](https://wonder.atlassian.net/browse/MD-17938) | UI - SCC Support - Frozen & Thawed | Sub-task |
| [MD-17943](https://wonder.atlassian.net/browse/MD-17943) | UI - Add Object type field | Sub-task |
| [MD-17999](https://wonder.atlassian.net/browse/MD-17999) | UI - Support Hot Hold Configuration for 42\* Items in Cookbook | Sub-task |
| [MD-18019](https://wonder.atlassian.net/browse/MD-18019) | UI - Multi-Select Validation | Sub-task |
| [MD-18023](https://wonder.atlassian.net/browse/MD-18023) | UI - Line build 编辑页速度与体验优化 | Story |
| [MD-18031](https://wonder.atlassian.net/browse/MD-18031) | UI - BOM structure: 增加跳转 vendor SKU 页面的链接 | Sub-task |
| [MD-18032](https://wonder.atlassian.net/browse/MD-18032) | 修复 Snyk 报告的前端高危安全漏洞 | Task |
| [MD-18068](https://wonder.atlassian.net/browse/MD-18068) | Item grid 的 filter attribute 下拉框增加模糊搜索 | Story |

---

## 前端需求拆解

### [MD-17938](https://wonder.atlassian.net/browse/MD-17938)

UI - SCC 支持 Frozen & Thawed

> 主 ticket：[MD-17928](https://wonder.atlassian.net/browse/MD-17928) SCC Support - Frozen & Thawed Major Tasks

- 本 sprint 该主题的主体前端改动。sub-task 自身无描述，需求以主 ticket 为准
- 配套的展示与校验调整（随同一主 ticket 交付）：
  - 展示文案 `Fzn` 统一为全大写 `FZN`
  - HDR consumable（40\*）详情页的状态 icon 顺序调整为 `Dormant` 排在 `Frozen` / `Thawed` 之前
  - 只要**任一**配对状态（paired state）的 40 item 已有 usage，就把 `BOM unit` 字段置灰、不允许修改
  - 当配对的 40 item **全部已发布**时，dormant 操作要展示对应的 warning 文案

### [MD-17943](https://wonder.atlassian.net/browse/MD-17943)

UI - 用后端返回的 object type 替换前端硬编码判断

> 主 ticket：[MD-17941](https://wonder.atlassian.net/browse/MD-17941)

- 前端此前靠硬编码方式判断 object type，改为读取后端新增的字段
- 涉及两个接口的返回结构：`GetItemCustomizationNutritionAJAXResponse$RecipeItem`、`GetItemProcedureRelatedItemsAJAXResponse$CustomizationOptionItem`
- 清理对应的硬编码判断逻辑

### [MD-17999](https://wonder.atlassian.net/browse/MD-17999)

UI - 为 42\* WSKU item 支持 Hot Hold 配置

> 主 ticket：[MD-17984](https://wonder.atlassian.net/browse/MD-17984)

**背景**：为了加速 hot-holding SKU 的上线，需要让 42\* item 也具备 Hot Hold 卡片（41\* item 已有，42\* 缺失）。

- 为 WSKU 42\* item 新增 **Hot Hold card**
- 权限与 41\* 的 `edit hot holding` **保持一致**
- 42\* item 的 hot holding 编辑页沿用 41\* 既有的字段与限制
- 42\* 的 hot hold 要纳入 change log
- 当 88\* item 关联的是 42\* item（`SCC Source = true`）时，**隐藏** 88\* item 上 hot holding card 的编辑按钮；否则保持 88\* 的编辑按钮可用
- 41\* → 42\* 的 hot holding instruction 继承逻辑在后端处理

### [MD-18019](https://wonder.atlassian.net/browse/MD-18019)

UI - Multi-Select customization 的 KDS Portion 校验

> 主 ticket：[MD-17892](https://wonder.atlassian.net/browse/MD-17892)

**背景**：带 multi-select customization 的 item，必须在 line build 中启用 KDS portion 才允许发布。

- **保存 line build 时**校验：若有 sub-step 映射到 `Custom Type = Multi-select` 的 customization option 却没有启用 KDS portion，则报错拦截
  - 文案：`Unable to saving line build. Multi-select customization option of main item/presets should be enabled the KDS portion in line build. Please check the KDS portion flag before saving it:`，其下按 task 列出 `Task#: step #-{customization option name}; …`
- **multi-select sub-step 的 inline error**
  - KDS portion 未勾选：`Multi-select customization option should be enabled the KDS Portion`
  - KDS portion 已勾选但缺少对应的 KDS portion conversion：`Missing KDS Portion Conversion, please uncheck it or revise it on component item`
- **纳入 Missing Info**
  - 触发条件：sub-step 映射的 multi-select option 缺少 KDS portion（未勾选，或已勾选但缺 conversion）
  - missing info 文案：`KDS portion of multi-select step (Missing)`，筛选归入 `CDT` 组
  - item 详情页与 item grid 页都要展示该 missing info
  - 存在 missing info 时，Item Details / Customization tab 显示提示 `Bad data exist in line build. Please update the line build.`
  - 按 line build 分组展示明细，例如 `Bad data exist in line build #1` 下分列「缺少映射 step/sub step 的 option」与「缺少 KDS portion 的 multi-select option」两类
- 校验范围需同时检查**主 menu item 版本及其关联 preset item** 的 `Custom Type`

> 注：本条上线后，业务反馈存在需要忽略 partial selection 的例外场景，Sprint 13 的 MD-18092 将其中的 error 降级为 warning。

### [MD-18023](https://wonder.atlassian.net/browse/MD-18023)

UI - Line build 编辑页的速度与交互体验优化

> 主 ticket（Epic）：[MD-17264](https://wonder.atlassian.net/browse/MD-17264) @2026 Regular optimization in Cookbook

- ticket 描述仅为空模板，未列具体条目；实际范围为 line build 编辑页的加载速度与交互体验优化

### [MD-18031](https://wonder.atlassian.net/browse/MD-18031)

UI - BOM structure 中增加跳转 vendor SKU 页面的链接

> 主 ticket：[MD-18024](https://wonder.atlassian.net/browse/MD-18024)

**背景**：CE 团队提出，在 BOM 结构里看到某一行时没有快捷入口跳到对应的 vendor SKU 页面，只能手动导航。

- **40\* item 的 fulfillment option card**：当关联的 vendor SKU 为 **buyout vendor SKU** 时，增加可点击链接跳转到 Cookbook 的 vendor SKU 页面
- **menu item / HDR recipe 的 component（Bill of Materials）card**：同样在关联 buyout vendor SKU 时增加该跳转链接

### [MD-18032](https://wonder.atlassian.net/browse/MD-18032)

修复 Snyk 报告的前端高危漏洞

> 主 ticket（Epic）：[MD-17231](https://wonder.atlassian.net/browse/MD-17231) @2026 Cookbook Technical Excellence

- 核对 Snyk 报告中前端的 high severity 漏洞，定位受影响依赖及其升级路径
- 逐项修复（升级依赖 / 打补丁 / 改代码），确认修复后不破坏功能
- 重新扫描确认高危项清零，且依赖更新未引入回归

### [MD-18068](https://wonder.atlassian.net/browse/MD-18068)

Item grid 的 filter attribute 下拉框增加模糊搜索

> 主 ticket（Epic）：[MD-17264](https://wonder.atlassian.net/browse/MD-17264) @2026 Regular optimization in Cookbook

**背景**：item grid 页的 filter attribute 下拉项（`dish_type`、`object_type`、`station`、`menu_availability` 等）选项越来越多，用户只能手动滚动查找，效率低且易错。

- 为 filter attribute 下拉框增加模糊 / 部分匹配搜索
- 用户点进筛选字段开始输入时，下拉列表按**不区分大小写的部分匹配**实时过滤，无需滚动整个列表
