# Front-end Jira Tickets — Sprint 18

> Sprint：`MD 2026 Sprint 18`，周期 **2026-08-24 → 2026-09-07**
> 本文档只覆盖 MD-18391 / MD-18454 / MD-18455 / MD-18465 四条。

| Key | 摘要 (Summary) | UI sub-task | 前端工作 |
| --- | --- | --- | --- |
| [MD-18391](https://wonder.atlassian.net/browse/MD-18391) | Display 'Unavailable' indicator for WSKU items without fulfillment in nutrition card | [MD-18467](https://wonder.atlassian.net/browse/MD-18467) | 小，1 个表格列 |
| [MD-18454](https://wonder.atlassian.net/browse/MD-18454) | Initialize Accounting Type & Accounting Sub-Type for 9\* items | [MD-18468](https://wonder.atlassian.net/browse/MD-18468) | 主体是后端数据初始化；前端只在选项集合有变化时需改硬编码的选项树 |
| [MD-18455](https://wonder.atlassian.net/browse/MD-18455) | Add free text label/description box for line builds | [MD-18469](https://wonder.atlassian.net/browse/MD-18469) | **已交付** |
| [MD-18465](https://wonder.atlassian.net/browse/MD-18465) | UI - Apply the template change to brand BYO/presets | 本身即 sub-task | 大，且影响面覆盖全部 menu item |

## 前端需求拆解


### [MD-18465](https://wonder.atlassian.net/browse/MD-18465)

把 template 的改动同步到 brand BYO / presets —— UI 部分

> 主 ticket：[MD-18458](https://wonder.atlassian.net/browse/MD-18458)（Story，assignee Felix），Epic [MD-17762](https://wonder.atlassian.net/browse/MD-17762) Wonder Create Integration
> 本 sub-task 自身 description 为空。**ticket 正文不完整，需求以 Confluence 为准**：[Wonder Create Template & Brand BYO Item Update Management](https://wonder.atlassian.net/wiki/spaces/RT/pages/5585993938)（最后更新 2026-08-20，含 case 表与 case sample）

**背景**：CDT 团队只维护一份 Wonder Create Template。Template 的改动要自动同步到 brand BYO item 及其 presets，同时禁止从 brand BYO / presets 上手动改。核心原则：**preset 的默认选项只能由 influencer 改** —— template 的改动（删 option、改 max options）会影响 preset 的既有配置，但**不得自动覆盖**；由 WC Portal 通知 influencer 手动调整，逾期未调整、校验不通过的 preset 由 WC Portal 置 inactive 并从 consumer app 下架。

**前端需求**

1. Delete option / delete customization 改为**只能在 scheduled version 操作**
   1. Confluence Case 4/5 明确标 **Need code change** —— 现状 normal menu item 可以在 active version 删
   2. Final（active）version 上置灰，tip：`Please delete it in future version.`
   3. **不分 normal menu item 还是 wonder create item**
2. Final version 上 `Min Options` / `Max Options` / `Ineligible` / `Required` / `Free Options` 置灰，tip：`Please revise it in future version.`；scheduled version 上仍可编辑
   1. 同样不分 normal / wonder create
3. `Duplicate Option with Different ID` 确认弹窗（**Confluence 独有，ticket 正文没写**）
   1. 触发：在 **template 的 scheduled version** 里保存 customization/option 时，与 active version 比对**同一个 customization 下**的 option name（大小写不敏感、必须完全一致），name 相同但 UUID 不同
   2. Header：`Duplicate Option with Different ID`
   3. 正文：`Identical options must use the same ID across active and scheduled versions. Is the option below essentially one and the same?`
   4. 展示：`V#（active version）{option name}` & `V#（current scheduled version）{option name}`
   5. 操作：`No` = 关闭弹窗并继续保存；`Yes` = 把 active version 的 option UUID 沿用到 scheduled version
4. 带 `wonder create` concept 的 menu item **整个 item detail 置为只读**（wonder template 本身除外）
   1. 只读的判定条件 = 后端给的 concept 标识字段 **且** 前端 feature flag 打开
   2. 加 flag 的目的是后续若要放开修改，直接关 flag 即可，不必改代码
5. Bulk swap / bulk edit BOM / customization usages 排除带 `wonder create` concept 的 menu item（**包括 template**）


**前端关注点**

> - 「active version 不许改、future version 才行」这套 gating **已有现成实现**：`isActiveVersion(service_start_time, service_end_time)`（`src/page/item/detail/utils/isActiveVersion.ts:3`）。`CreateOptionForm.tsx:408` 已经用它把 `Custom Type` 在 active 版本上置灰 —— 正好就是 Confluence 里「except custom type」那条。需求 2 本质上是把同一个条件复制到另外几个字段。
> - 需求 2 四个字段的落点（都在 `src/page/item/customizationV2/component/CreateOrUpdate/components/`）：
>   - `Min Options`：`CreateOptionForm.tsx:337-340`
>   - `Max Options`：**两处** —— `CreateOptionForm.tsx:385-387` 与 `:489-491`（不同 customization type 走不同分支，很容易只改一处）
>   - `Required` / `Ineligible`：`OptionValue.tsx` 的 `getRequiredDisabledReason`（`:46`）与 flag 列
>   - `Free Options`：`CreateOptionForm.tsx:431-436` —— 注意**界面上的 label 是 `Free Choices`**，字段名是 `freemium`，跟需求里的叫法不一致，搜 "Free Options" 找不到
> - `Required` 那块 MD-18470 刚改过（规则是「已勾选的框永不上锁」）。新加的 active-version 置灰要和它叠加，注意不要把「勾着却取消不掉」的死锁又造回来。
> - **两个删除入口目前完全没有版本 gating**，只看 `readOnly` 和权限 —— `CreateOptionValue.tsx:456-458` 的 `Delete Option`、`CreateOptionModalImpl.tsx:311-313` 的 `Delete Customization`。这正是 Confluence 说的 "Need code change"，改法是把 `disabled={readOnly}` 补上 `isActiveVersion(...)` 并套一层 `ShowPopover` 出 tip（`OptionValue.tsx:82` 的 `disabledDelete` + `:301` 就是现成范式）。
> - 若需求 1 需要判断「这个 item 有没有 scheduled version」，管道已存在：`useCreateOptionAction.tsx:184` 的 `hasNextVersion` 由 `customizationV2/component/CreateOrUpdate/page.tsx:484` 的 `getNextItemVersion` 填充，不用新建。
> - **`readOnly` 已经贯穿整条 customization 编辑链**（`CreateOptionModalImpl` → `CreateOptionForm` → `CreateOptionValue` → `OptionValue`，每个 Form.Item 都吃这个 prop），所以需求 4 不必逐字段改，在入口把 `readOnly` 置 true 即可。
> - 需求 4 的 feature flag 有现成机制：`useGetFlags()`（定义在 `src/page/cloudbees/index.tsx:60`），flag key 用 kebab-case，用法参考 `page/main/component/Nav.tsx:65-67` 的 `cookbook-enable-wonder-create-debug-page`。
> - **代码里目前没有任何 `wonder create` concept 的概念** —— 全仓库只有 MD-18385 建的 `wonderCreateDebug` 调试页。「这个 item 带 wonder create concept 吗」「它是不是 template」都需要后端给字段。
> - 需求 5 的落点：`src/page/item/detail/components/ItemUsage/BulkEditUsage/`（BOM / Components / Customization 三个 action hook）与 `components/UsagesBulkBar/`。MD-17869 的 Customization Bulk Swap 还在等后端，那条落地后也要一起排除。
> - **影响面提醒**：需求 1 和 2 都写了 regardless of normal menu item or wonder create item，也就是说这是对**所有 menu item** 的行为变更，不是 wonder create 专属。

- 
### [MD-18391](https://wonder.atlassian.net/browse/MD-18391)

40\* item 的 Nutrition 卡片里，给不参与营养计算的 WSKU 打 `Unavailable` 标

> 主 ticket（Epic）：[MD-17501](https://wonder.atlassian.net/browse/MD-17501) Supply Chain Catalog Integration

**背景**：40 item 的营养值优先由可用的 41/42 WSKU 计算得出，取不到时才从 `40*F` 继承。但一个 42 item 可能同时满足「没有配置 fulfillment option」+「没有对应的 `W42F` 可继承」+「不是 dormant」——它不参与 40 的营养计算，却照样列在 Nutrition 卡片的 Linked WSKU 里。它自己的营养数据也不会被自动清空，于是用户会误以为这行有效。

**需求**

1. 在 Nutrition 卡片 `Linked WSKU` tab 中，于 item number 下方增加一个 `Unavailable` chip/badge，表明该 item 不能用于营养计算。


---

### [MD-18454](https://wonder.atlassian.net/browse/MD-18454)

按用户给的表格重置 9\* item 的 `Accounting Type` / `Accounting Sub-Type`

前端这边要更新一下枚举的值

---

### [MD-18455](https://wonder.atlassian.net/browse/MD-18455)

给 line build 增加自由文本的 `Label` / `Description`

> UI sub-task [MD-18469](https://wonder.atlassian.net/browse/MD-18469) **已交付**：组件落在 `lineBuildList/component/LineBuildNote/`（`index.tsx` / `LineBuildDescription.tsx` / `LineBuildTabLabel.tsx`），新旧两个 tab 都已覆盖。下面的拆解保留作记录。

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
3. hover `Line build #` tab 时展示 `Label`
4. `Description` 展示在 line build header 信息块的**最后一行**（ticket 原文分三种情况写：单版本时在 `Apply to restaurants` 下方，其余在 `Option Value` / `Selected Option value amount` 下方 —— 对照设计稿，三种情况都等价于「排在最后」）
5. 没有 description 时，不展示 `Description` 这一行
6. `Label` 和 `Description` 要进 change log



### [MD-18477](https://wonder.atlassian.net/browse/MD-18477)

edit linebuild 页面 点击step的时候，展开当前task，收缩其他的task


