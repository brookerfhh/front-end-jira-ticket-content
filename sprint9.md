# Front-end Jira Tickets — Sprint 9

> Sprint：`MD 2026 Sprint 9`，周期 **2026-04-22 → 2026-05-04**

| Key | 摘要 (Summary) | 类型 |
|-----|----------------|------|
| [MD-17503](https://wonder.atlassian.net/browse/MD-17503) | UI - Tech Item detail refactor: 修复 URL 上 item number 不正确 | Task |
| [MD-17751](https://wonder.atlassian.net/browse/MD-17751) | UI - Adapt 40\* "Production BOM" card to the Menu Item BOM card | Sub-task |
| [MD-17752](https://wonder.atlassian.net/browse/MD-17752) | UI - Storage Type 字段从 Logistics Card 移到 Food Science Card | Sub-task |
| [MD-17754](https://wonder.atlassian.net/browse/MD-17754) | UI - sold status followup | Sub-task |
| [MD-17798](https://wonder.atlassian.net/browse/MD-17798) | 把 40 item 的 "Fulfillment Options" 改名为 "Fulfillment Mappings" | Sub-task |
| [MD-17825](https://wonder.atlassian.net/browse/MD-17825) | UI - IK Eligible 标记从 sub-step 层级移到 step 层级 | Sub-task |
| [MD-17833](https://wonder.atlassian.net/browse/MD-17833) | UI - 40 item 内 W42 item 的 Amount 改由 usable quantity 推导 | Sub-task |
| [MD-17836](https://wonder.atlassian.net/browse/MD-17836) | UI - Show Active Buyout Vendor SKU | Sub-task |

---

## 前端需求拆解

### [MD-17503](https://wonder.atlassian.net/browse/MD-17503)

UI - Item detail 重构：修复 URL 中 item number 与实际页面不一致

> 主 ticket（Epic）：[MD-17231](https://wonder.atlassian.net/browse/MD-17231) @2026 Cookbook Technical Excellence

- 复现路径：item 详情页跳转后 URL 上带的 item number 被复制过去，点左上角 `←` 返回上一页时，页面上的 item number 与 URL 里的不一致
- 需修正详情页的路由参数与页面状态同步逻辑，保证前进 / 返回后 URL 与页面展示的 item 始终一致

### [MD-17751](https://wonder.atlassian.net/browse/MD-17751) / [MD-17836](https://wonder.atlassian.net/browse/MD-17836)

UI - 把 40\* 的 fulfillment option 结构并入 menu item 的 component card，并展示 active buyout vendor SKU

> 主 ticket：[MD-17600](https://wonder.atlassian.net/browse/MD-17600)

**背景**：40\* 的 "Production BOM" card 原本的意图是扩展 menu item 的 BOM card，让用户能从 menu item 的一级 component 一路下钻 40\* → 41\* → 88\* / vendor SKU。

- **Menu item component card**
  - 把 40 item 的 fulfillment option card 并入 menu item component card，保留 40\* 既有的 fulfillment option 结构
  - **不**按 40 item 在 menu item 中的用量去缩放子 component 数量
  - component 40\* item 若关联了 41\* / W42\*，显示 `+` 图标可展开 41\* / W42\* → 88\* → 88\* 的子 component
  - 列与 menu item component 对齐
- **40\* item 详情页 → fulfillment options card**
  - 88\* 取其 active version
  - 40\* 关联多个 41\* / W42\* 时全部展示（**排除已删除、dormant、以及 inactive 的 W42**）
  - 41\* 关联多个 88\* 时全部展示（**排除 41\* 的 inactive 关联**）
  - W42\* 关联多个 88\* 时全部展示（**排除 dormant/已删除的 88\*、以及 W42\* 的 inactive WSKU UOM**）
  - 关联的 WSKU 为 W42\* 时，version / service start / service end 不展示实际值，显示为 `--`
  - W42\* 关联 buyout vendor SKU 时，展示 **active** 的 buyout vendor SKU

### [MD-17752](https://wonder.atlassian.net/browse/MD-17752)

UI - 把 `Storage Type` 的维护入口从 Logistics Card 移到 Food Science Card

> 主 ticket：[MD-17619](https://wonder.atlassian.net/browse/MD-17619)

**背景**：`Storage Type` 实际由 Food Science 团队负责（此前一直由 Anthony 代为在 88\* 的 Logistics Card 维护），因此把维护入口交给 Food Science Card。这是纯 UI/UX 调整，**数据仍存在 logistics 区段，DB 字段不变**。

- item 详情页的 **Logistics Card 与 Food Science Card 都要展示** `Storage Type`
- 无论 item object type 是什么，`Storage Type` **只能在 Food Science Card 中修改** —— 从 edit logistics 页面中隐藏该字段
- **Add food science data 弹窗**：在 `Add Measurement` 按钮上方新增 `Storage Type` 字段
  - 可选值与 Logistics Card 中一致：`Ambient`、`Chilled`、`Frozen`、`Missing`
  - 新建 88\* item 时默认 `Storage Type = Missing`，且不允许手动清空
- **Food Science Card → Measurement Trials**：在 measurement 表格上方展示 `Storage Type` 字段
- **Missing Info**
  - 沿用既有校验：recipe 80\* / byproduct 6\* / packaged 88\* 发布版本时 `Storage Type` 必填，ingredient 5\* 发布时可选
  - missing info 提示要显示在 Food Science Card 上
  - missing info 搜索按 `Food Science` 团队归类
- Change log 不变，仍在 Logistics Card 上展示
- Food Science 的导出动作**不需要**包含 `Storage Type`
- 配套的字段可见性调整：`Benchtop item` / `Benchtop byproduct` 的 Food Science card 不显示 `Storage Type`；non food item（9\*）的 Food Science card 需可保存

### [MD-17754](https://wonder.atlassian.net/browse/MD-17754)

UI - sold status 后续调整

> 主 ticket：[MD-17705](https://wonder.atlassian.net/browse/MD-17705)

前端相关部分（sold status 的数据来源与推导规则均在后端 / 数仓侧）：

- **Dormant 41\* item**：把原本的 error 拦截改为**非阻断 warning**
  - 触发条件：dormant 一个 sold status 为 `for sale` / `scheduled` 的 41\* item
  - 弹窗文案：标题 `Are you sure`，正文 `Are you sure you want to dormant main item of which sold status is {sold status}?`，按钮 `Cancel` / `Dormant`

### [MD-17798](https://wonder.atlassian.net/browse/MD-17798)

把 40 item 的 `Fulfillment Options` 改名为 `Fulfillment Mappings`

> 主 ticket：[MD-17600](https://wonder.atlassian.net/browse/MD-17600)

- 纯文案改动：40 item 上的 `Fulfillment Options` 一律改为 `Fulfillment Mappings`

### [MD-17825](https://wonder.atlassian.net/browse/MD-17825)

UI - `IK Eligible` 标记从 sub-step 层级移到 step 层级

> 主 ticket：[MD-17820](https://wonder.atlassian.net/browse/MD-17820)

- `activity = garnish` 时，在 **garnish step 层级**显示 `IK Eligible` 复选框，**无默认值**，需用户手动勾选；其他 activity 不显示该复选框
- 把 activity 切换成非 garnish 时，隐藏 `IK Eligible` 复选框，并**清空**原先的 `IK Eligible` 值
- **保存 line build 时校验**：若某 step `IK Eligible = true`，但其关联的 component / customization option 中没有任何 `Machine Eligible = yes`，则报错拦截
  - 错误信息：`The following IK Eligible step is missing 'Machine Eligible' component:`，其下按 task 列出 `Task 1 -step 2, step 3` / `Task 2 -step 2, step 4`
  - 同时在复选框下方显示 inline error：`Missing attribute Machine Eligible.`
- 编辑 line build 时同样适用上述 inline error 与错误信息

### [MD-17833](https://wonder.atlassian.net/browse/MD-17833)

UI - 40 item 内 W42 item 的 Amount 改由 usable quantity 推导

> 主 ticket：[MD-17758](https://wonder.atlassian.net/browse/MD-17758)

- 40 item 内 W42 item 的 amount 改为取 SCC 的 usable quantity 字段
- **40 item 详情 → Linked WSKUs**：列名 `QTY` 改为 `Usable QTY`；若改名会引起其他问题，则保留原列名并在列头加提示 `It's usable QTY.`
- W42 详情页的对应展示同步调整
