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


---

### [MD-18454](https://wonder.atlassian.net/browse/MD-18454)

按用户给的表格重置 9\* item 的 `Accounting Type` / `Accounting Sub-Type`

前端这边要更新一下枚举的值

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
3. hover `Line build #` tab 时展示 `Label`
4. `Description` 展示在 line build header 信息块的**最后一行**（ticket 原文分三种情况写：单版本时在 `Apply to restaurants` 下方，其余在 `Option Value` / `Selected Option value amount` 下方 —— 对照设计稿，三种情况都等价于「排在最后」）
5. 没有 description 时，不展示 `Description` 这一行
6. `Label` 和 `Description` 要进 change log

### [MD-18477](https://wonder.atlassian.net/browse/MD-18477)

edit linebuild 页面 点击step的时候，展开当前task，收缩其他的task


