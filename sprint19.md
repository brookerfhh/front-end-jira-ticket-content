# Front-end Jira Tickets — Sprint 19

> Sprint：`MD 2026 Sprint 19`，周期 **2026-09-07 → 2026-09-21**


## 前端需求拆解

### [MD-18490](https://wonder.atlassian.net/browse/MD-18490) —— 前端 Snyk 漏洞修复（12 条，2026-08-25 扫描）

> 主 ticket。Epic：MD-17251 @2026 Cookbook Devops Action

修复 cookbook 和 pcs 的漏洞 
剩下一个 react-router



### [MD-18497](https://wonder.atlassian.net/browse/MD-18497) —— 移除 'Create New WSKU Items' 权限

> 主 ticket。Epic：MD-17264 @2026 Regular optimization in Cookbook

**背景**

41* item 已废弃，不再支持新建。创建按钮已从系统中移除，现在要把对应的权限项一并删掉。

**需求**

删除权限码 `recipe-site:ITEMS:ITEM_GRID:CREATE_NEW_WSKU_ITEMS`（Jira 附的截图是权限配置页）。


### [MD-18520](https://wonder.atlassian.net/browse/MD-18520) —— 40* item 缺失 active WSKU (42*) 的校验

> 主 ticket。Epic：MD-17501 Supply Chain Catalog Integration

**背景**

IKC 业务在用的 88* item 已正式迁到 SCC，新建的 W42* 不再设 cutover date；残留的 88* item（已停用或只在 B2B 用）不迁移；41* item 废弃，不再支持新增/修改。原先「40* item 是否缺少 active for ordering 的 41* item」这套校验要整体换成「40* item 是否缺少 SCC status=active 的 42* item」。

**需求**

1. 新增 missing info校验，两条消息：`Active WSKU in Component (Missing)`、`Active WSKU in Customization (Missing)`；在 item grid 与 item 详情页展示（详情页的 missing fields 区域也要出这条消息）item grid missing fields column 里面也要展示
2. Component / Customization card里，有问题的那条 40* item 行旁边显示 error icon，tip 文案 `Missing Active WSKU`。
3. publish draft menu item / 7* item 时，有 error 级 missing info 就阻断发布（复用现有 missing info 机制）。

4. 保存publish 过的 menu item / 7* item 的 component 或 customization 时，若存在缺 42*（SCC status=active）的 40* item，弹 warning。
   - SCC Source=true 的逻辑**不变**（仍从 SCC 取 41* 校验 `active for ordering=true`），只调整文案：
     - Header：`Are you sure`
     - 正文：`The following component item({40* item number1}, {40* item number2}) is missing active WSKU, which might cause OOS, are you sure you want to save the configuration?`（原文案里的 `for ordering 41* item` 去掉）
     - 按钮：Cancel / Save
   - **删除** SCC Source=false 的 40* item 从 Cookbook 校验缺 published `active for ordering=true` 41* 的那条 warning。


### [MD-18525](https://wonder.atlassian.net/browse/MD-18525) —— 保存时对齐 active / scheduled 版本间重名 option 的 UUID

> 主 ticket。Epic：MD-17762 Wonder Create Integration

**背景**

option UUID 是下游系统的主键。用户删掉一个 option 再建一个同名的，新 option 拿到的是新 UUID，下游按 UUID 认人，就会当成两个不相干的 option 而报错。

**需求**

1. 保存 menu item 的 scheduled 版本的 customization/option 时（**不区分是不是 Wonder Create**），把 scheduled 版本的 option name + UUID 和 active 版本比对，发现同名但 UUID 不一致就弹确认框。
2. 弹窗 Header：`Duplicate Option with Different ID`
3. 正文：`Identical options must use the same ID across active and scheduled versions. Is the option below essentially one and the same?`
4. 列出 `V# (the active version) {option name}` 与 `V# (current scheduled version) {option name}`
5. 按钮 No / Yes。Yes：把 active 版本的 UUID 沿用到 scheduled 版本；No：关闭弹窗并继续保存。
6. 若被替换掉的那个新 UUID 已经用在 line build 里，line build 那边也要自动替换。



### [MD-18526](https://wonder.atlassian.net/browse/MD-18526) —— Bulk Swap / Bulk Edit 排除 Wonder Create concept 的 menu item


**需求**

Bulk Swap / Bulk Edit BOM & Customization Usages 排除带 "Wonder Create" concept 的 menu item
在弹窗那边直接过滤掉 不展示


### [MD-18454](https://wonder.atlassian.net/browse/MD-18454)

按用户给的表格重置 9\* item 的 `Accounting Type` / `Accounting Sub-Type`

前端这边要更新一下枚举的值