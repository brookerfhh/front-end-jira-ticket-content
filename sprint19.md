# Front-end Jira Tickets — Sprint 19

> Sprint：`MD 2026 Sprint 19`，周期 **2026-09-07 → 2026-09-21**


## 前端需求拆解

### [MD-18497](https://wonder.atlassian.net/browse/MD-18497) —— 移除 'Create New WSKU Items' 权限

> 主 ticket。Epic：MD-17264 @2026 Regular optimization in Cookbook

**背景**

41* item 已废弃，不再支持新建。创建按钮已从系统中移除，现在要把对应的权限项一并删掉。

**需求**

删除权限码 `recipe-site:ITEMS:ITEM_GRID:CREATE_NEW_WSKU_ITEMS`（Jira 附的截图是权限配置页）。

**前端关注点**

> 创建入口**确实已经关掉了**，但不是删除，是注释掉的：
> `src/page/WSKU/list/components/SearchFormNew/index.tsx:18`（import 注释）、`:235`（`{/* <PageHeaderExtra onFinish={onFinish} /> */}`）

> 该权限码在前端**唯一还活着的用途是「编辑」而不是「创建」**：
> `src/page/WSKU/detail/component/BasicInformation/index.tsx:73` 渲染 `<WSKUAction data={...} />`，`WSKUAction` 内部按 `data ? "edit" : "create"` 分模式，两种模式共用同一个 `hasPermission(ITEMS_ITEM_GRID_CREATE_NEW_WSKU_ITEMS)` 门禁（`.../WSKUAction/index.tsx:266`）。
> ⇒ 权限一旦从权限系统删掉，`hasPermission` 恒为 false，**WSKU 详情页的编辑铅笔会静默消失**。

> `frontend/product-catalog-site-frontend/src/constants/permission_code.ts:6` 也声明了同一个常量，但全项目无引用，是死代码，可顺手删。

**疑问**

- 编辑入口要不要一起去掉？MD-18520 的背景写的是「41* 不再支持 create/update」，按这个口径编辑也该没；但如果还想保留编辑，就得给编辑单独配一个权限码，不能靠删这一个权限顺带实现。
- 后端删权限与前端删代码谁先上？先删权限后端会导致编辑入口在一个发版窗口内无声消失（不是报错，是按钮不见了），QA 容易漏测。

### [MD-18520](https://wonder.atlassian.net/browse/MD-18520) —— 40* item 缺失 active WSKU (42*) 的校验

> 主 ticket。Epic：MD-17501 Supply Chain Catalog Integration

**背景**

IKC 业务的 88* item 已正式迁到 SCC，新建的 W42* 不再设 cutover date；41* item 废弃。原先「40* item 是否缺少 active for ordering 的 41* item」这套校验要整体换成「40* item 是否缺少 SCC status=active 的 42* item」。

**需求**

1. 新增 missing info（error 级）校验，两条消息：`Active WSKU in Component (Missing)`、`Active WSKU in Customization (Missing)`；在 item grid 与 item 详情页展示，归属 **Procurement** team。
2. Component / Customization 卡片里，有问题的那条 40* item 行旁边显示 error icon，tip 文案 `Missing Active WSKU`。
3. 发布 draft menu item / 7* item 时，有 error 级 missing info 就阻断发布（复用现有 missing info 机制）。
4. **删除**原有两条发布期校验（SCC Source=false 查 Cookbook 的 41*、SCC Source=true 查 SCC 的 41*）。
5. 保存已发布 menu item / 7* item 的 component 或 customization 时，若存在缺 42* 的 40* item，弹 warning。
6. **删除**原有 component 侧 SCC Source=false 的 41* warning。
7. SCC Source=true 的 warning 逻辑不变，只调整文案：去掉 `for ordering 41* item`，改为 `missing active WSKU`。

**前端关注点**

> **需求 7 的文案已经是现状了**——MD-18305 已经统一成 `The following component item(s) is missing active WSKU in SCC, which might cause OOS, are you sure you want to save the configuration? Component(s): {list}`，共 4 处：
> `customizationV2/component/CreateOrUpdate/page.tsx:230`、`:418`、`customizationV2/component/CreateOrUpdate/hooks/useCreateOptionAction.tsx:508`、`item/detail/components/ComponentsV2/Action/index.tsx:193`

> 需求 4/5/6/7 前端**零改动**：这套 warning 完全由后端字段 `missing_active_for_ordering_item_numbers_warning` 驱动，前端只判断数组非空就弹框，代码里没有任何 41*/42* 的判定。语义从 41* 换成 42* 只改后端计算口径即可（字段名保留原样更省事，见疑问）。

> 需求 1 前端也基本零改动：item grid 的 missing info 列走 `formatMissFieldToList(validation_information, required)`（`page/menusV2/menuDetail/utils/index.tsx:60`）泛型渲染，tip 文字直接取后端 `InvalidDetail.tip`；详情页走 `transToMap(validation_information)` 按 `location_card` 分组（`item/detail/pages/basicInfomation/hooks/useBenchtopAnchors.tsx:27`），`ValidationCardAJAXView` 里 `COMPONENT` / `BOM` / `CUSTOMIZATION` 都已存在。owner team 也是后端配的（`validation_owner_configs`，`item/list/components/SearchFormNew/MissingInfo.tsx:29`），加 Procurement 无需前端改。

> 需求 1 唯一的前端触点：筛选器里的枚举文案映射 `src/constants/validationInfo.ts`（`ValidationInfoOptionMap`）。没有映射时会走 `firstLetterCapital` 自动转换，新枚举会显示成 `Active Wsku In Component`（WSKU 大小写错），需要补两条映射。

> 需求 2 是**唯一需要实打实写代码的一项**。Component 卡片列在 `item/detail/components/ComponentsV2/useComponentColumn.tsx`，现有同类形态是 `<Alert16 color="surfaceWarningStrong" /> Requires packaging`（`:165`、`:180`），照抄即可；但需要后端在 BOM line / customization option item 上给一个**逐行**的标记字段，现在的 `missing_active_for_ordering_item_numbers_warning` 只是一串 item number，挂不到行上。

**疑问**

- 需求 2 的逐行标记，后端是新加一个行级布尔字段，还是让前端拿 `missing_active_for_ordering_item_numbers` 这串 item number 去和行数据做匹配？后者在嵌套 BOM（同一个 40* 出现在多层）下会全部标红，需要确认这是不是想要的。
- 需求 5「保存**已发布** menu item / 7* item 时弹 warning」——现在这 4 处 warning 是不分版本状态一律弹的。是要收窄成只在已发布版本弹，还是维持现状？
- 后端字段名 `missing_active_for_ordering_item_numbers_warning` 语义已经从 41* 变成 42*，建议**不要改名**：改名要同步动 4 处前端 + 若干 check 接口的返回类型，收益为零。

### [MD-18525](https://wonder.atlassian.net/browse/MD-18525) —— 保存时对齐 active / scheduled 版本间重名 option 的 UUID

> 主 ticket。Epic：MD-17762 Wonder Create Integration

**背景**

option UUID 是下游系统的主键。用户删掉一个 option 再建一个同名的，新 option 拿到的是新 UUID，下游按 UUID 认人，就会当成两个不相干的 option 而报错。

**需求**

1. 保存 menu item 的 scheduled 版本的 customization/option 时（**不区分是不是 Wonder Create**），把 scheduled 版本的 option name + UUID 和 active 版本比对，发现同名但 UUID 不一致就弹确认框。
2. 比对范围限定在**同一个 customization 内**（如 choose your protein）。
3. option name 大小写不敏感，其余必须完全一致。
4. 弹窗 Header：`Duplicate Option with Different ID`
5. 正文：`Identical options must use the same ID across active and scheduled versions. Is the option below essentially one and the same?`
6. 列出 `V# (the active version) {option name}` 与 `V# (current scheduled version) {option name}`
7. 按钮 No / Yes。Yes：把 active 版本的 UUID 沿用到 scheduled 版本；No：关闭弹窗并继续保存。
8. 若被替换掉的那个新 UUID 已经用在 line build 里，line build 那边也要自动替换。

**前端关注点**

> **这条需求前端已经写完过一次。** 它原本是 MD-18465 的需求 3，做完后被要求移交给别的 ticket，实现以 patch 形式存在仓库外：`front-end-jira-ticket-content/MD-18465-req3-duplicate-option-modal.patch`（新增 `useDuplicateOptionCheck.tsx`，改 `useCreateOptionAction.tsx` 两处 `onFinish`，`CreateOrUpdate/page.tsx` 加一个 `getCustomizationByVersion`）。需求 1~7 全部覆盖，文案逐字对齐。

> 那份实现走的是**纯前端**路线：保存前拉一次 active 版本的 customization 树，按 `trim().toLowerCase()` 比名字，弹窗，Yes 就把 payload 里的 id 换成 active 版本的再提交。
> 由此带来的两个边界，需求本身已经解决了一个：patch 里留了个 TODO —— 当时无法识别「template」（template 是在 Cookbook 手工建的，`is_wonder_create_item === false`，和普通 menu item 长得一模一样），只好退化成「所有非 Wonder Create item 都跑这个检查」。**本 ticket 明确写了「regardless of wonder create or not」，这个 TODO 自动作废。**

> 覆盖范围的缺口：patch 只挂在 customization 编辑弹窗的保存出口上。`CreateOrUpdate/page.tsx` 里另有两条独立的 option value 增改链路（`optionValueCreateCheck` `:223`、`optionValueUpdateCheck` `:396`），走的是「后端 pre-check 接口返回 warning 字段 → 前端链式弹确认框」的模式，没有接上这个检查。

> 需求 8（line build 联动替换）前端做不到——line build 是另一套数据，前端保存 customization 时不碰它。

**疑问**

- 判定放前端还是后端？需求 8 无论如何要后端做（前端换完 UUID 提交，后端得顺着把 line build 里的引用一起改）。既然后端已经要参与，是不是干脆把比对也挪到已有的 pre-check 接口里（`optionValueCreateCheck` / `optionValueUpdateCheck` / `replaceCheck` 都是这个模式），前端只负责弹框？这样能自然覆盖到全部三条保存链路，也不用每次保存多拉一次 active 版本。
- 需求 6 的弹窗正文，两行的 `{option name}` 按定义必然完全相同（就是靠同名匹配上的），显示出来就是两行一样的字只有 V# 不同。是不是本意想展示两边的 UUID，或者只想展示「V3 → V4：{option name}」一行？
- 除了 customization 编辑弹窗，Featured / Presets 侧单独增改 option value 的入口要不要也管？

### [MD-18526](https://wonder.atlassian.net/browse/MD-18526) —— Bulk Swap / Bulk Edit 排除 Wonder Create concept 的 menu item


**需求**

Bulk Swap / Bulk Edit BOM & Customization Usages 排除带 "Wonder Create" concept 的 menu item（**含 wonder template**）。


