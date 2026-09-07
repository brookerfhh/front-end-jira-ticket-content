# Front-end Jira Tickets — Assigned to Me (Sprint 16)

> 更新时间：2026-07-27
> 查询条件：`key in (MD-18318, MD-18317, MD-18316)`（本 sprint 按指定范围只写这 3 个 ticket）

| Key                                                      | 摘要 (Summary)                                                                                                     |
| -------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| [MD-18318](https://wonder.atlassian.net/browse/MD-18318) | UI - Enable to Search Cooking Group Tagged on Task                                                               |
| [MD-18317](https://wonder.atlassian.net/browse/MD-18317) | [Tech] - add Amplitude                                                                                           |
| [MD-18316](https://wonder.atlassian.net/browse/MD-18316) | Remove "Enable non-40 item" Flag and Add Validation Restricting Component/Customization Search to 40/70/90 Items |
|                                                          |                                                                                                                  |

> 说明：3 个 ticket 均为本 sprint 新增，无沿用 Sprint 15 的条目。Sprint 15 未完成的其余 ticket（MD-18258 / MD-18243 / MD-18151 / MD-17869 / MD-18084 等）本次未纳入清单。

---

## 前端需求拆解

### [MD-18318](https://wonder.atlassian.net/browse/MD-18318)

UI - 支持搜索 line build task 上打的 Cooking Group

> 主 ticket：[MD-18312](https://wonder.atlassian.net/browse/MD-18312)（本 sub-task 自身描述为空，需求以主 ticket 为准）

**背景**：[MD-18165](https://wonder.atlassian.net/browse/MD-18165) 已支持在 menu item 的 line build task 上打 cooking group 标签。现在用户希望：无论 cooking group 是打在 **menu item 本身（attribute card）** 还是打在 **line build task** 上，都能被搜索到。

- **Attribute search 页面（item 下方的 attribute usage 表）**：需要把打在 line build task 上的 cooking group 也一并展示

  加个checkbox 'Including Line Build'搜索字段, 默认不勾选
  - 列定义调整为：`Attribute Name`、`Type`、`Value`、`Tagged On`、`Note`、`Updated Time`、`Updated By`
  - **删除**现有的 `Created Time`、`Created By` 两列
  - `Tagged On` 取值：`Attribute Card` 或 `Line Build`
  - `Note`：当 tagged on line build 时，展示「版本号(版本状态)-line build 编号」，例如 `V2 (Scheduled)-Line Build 1`
- 排序：同一个 menu item 若在 attribute card 和 line build task 上都打了 cooking group，**attribute card 的 attribute 排在前面**

- tip修改：标题 `Items' Attributes Usage` 改名为 `Items' Attributes Usage Tagged on Attribute`




### [MD-18317](https://wonder.atlassian.net/browse/MD-18317)

[Tech] - add Amplitude（补齐埋点）

> 主 ticket（Epic）：[MD-17231](https://wonder.atlassian.net/browse/MD-17231) @2026 Cookbook Technical Excellence

- 给所有导入 / 导出入口加上 Amplitude 埋点（Amplitude 客户端已存在于 `src/utils/analytics/`，只需补覆盖）。
- 排查全站是否还有其他关键操作遗漏埋点，一并补上。

### [MD-18316](https://wonder.atlassian.net/browse/MD-18316)

移除 "Enable non-40 item" 开关，并限制 component/customization 搜索只能选 40/70/90 item

> 主 ticket（Epic）：[MD-17501](https://wonder.atlassian.net/browse/MD-17501) Supply Chain Catalog Integration

**背景**：stackable item 已全量迁移到 40 consumable 模型。迁移过渡期曾引入 `Enable non-40 item` 开关，用于紧急情况下把 stackable 模型的 item 加进 menu item。现在 40 consumable 模型已稳定运行，该过渡开关不再需要。

**需求**

1. **移除 `Enable non-40 item` 开关**，连同所有相关的 toggle / 逻辑分支——即所有「允许把非 40（stackable 模型）item 加到 menu item / 7\* item」的路径
2. **新增校验（New）**：对于 **Final Version** 的 menu item / 70\* item，在搜索 component / customization（mandatory choice / optional addition）时，只能返回和选择 **40 / 70 / 90** item

**已有逻辑（本次不改，仅作为校验矩阵的上下文）**

| 版本状态 | component / customization 搜索行为 | 状态 |
| --- | --- | --- |
| Final Version | 只返回 / 只可选 **40 / 70 / 90** item | **新增** |
| Scheduled Version | 只返回 / 只可选 **40 / 70 / 90** item | 已有 |
| Draft Version | 允许把 3\* benchtop item 加到 menu item / 7\* item 的 component | 已有 |
| Draft 发布时 | component / customization（mandatory choice / optional addition）中存在非 40/70/90 item 时报错拦截 | 已有 |

