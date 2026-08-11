# Front-end Jira Tickets — Sprint 17

> Sprint：`MD 2026 Sprint 17`，周期 **2026-08-10 → 2026-08-24**

## 前端需求拆解

### [MD-18339](https://wonder.atlassian.net/browse/MD-18339)

用新设计重做 Line Build 的 Training Card 导出（PDF）

前端清理一下代码，导出改为后端导出。
---

### [MD-18346](https://wonder.atlassian.net/browse/MD-18346)

给 Customization Option 增加 `Required` 标记，以驱动 Menu Item 置 OOS

> 主 ticket（Epic）：[MD-17762](https://wonder.atlassian.net/browse/MD-17762) Wonder Create Integration
> **时间要求（ticket 原文）**：team 需要它用于 **9/23 pizza launch**，因此「9 月初就要 ready」

**背景**：Chicken Salad 这个 menu item 在鸡肉库存不足时不会显示 OOS（因为其他 customization option 还有货），但业务上主料缺货时该 menu item 就应该不可售。

**需求**

1. 给 customization option 新增 `Required` 标记：
   1. 只适用于 customization type = `mandatory choice`（其中 `none` 选项排除，置灰）
   2. 可在 **main menu item** 和 **preset** 上分别配置
   3. 同一个 group 内允许勾选多个 option 为 required
   4. `ineligible` 与 `required` **不能同时勾选**，勾选其一则另一个置灰，并显示 tip
2. 当 max options 不为 null 时，required option 的 portion qty 或 option 数量应 `<= max options`（**ticket 原文标注 TBD**），报错文案：
   - `Unable to save. Required option amount should be <= max options ({value}).`
3. **示例（ticket 中为未决的二选一）**：preset `Chicken Bowl` 中 chicken 被标为 required、default portion = 2，若 chicken 库存只有 1，该 preset 是否 OOS？
   - 方案 a：库存 1 就 OOS → 顾客不能选 chicken x1 + tofu x1，需要在 Cookbook / Wonder App / Pantry 三侧都加校验逻辑，且要**新增参数**（如 chicken required, `min portion = 2`），否则 Pantry/App 无法识别 required 的份量
   - 方案 b：库存 1 时仍可售 → 顾客可以选 chicken x1 + tofu x1
4. `required` 标记要进 change log
5. copy new item / create new version 时**继承** `required` 标记
6. API 需把 `required` 标记返回给 Pantry / Consumer App
7. variant item 版本 Required gray out

> 前端关注点（代码定位，模块 `src/page/item/customizationV2/`）：
> - **Main menu item 侧** —— create/edit customization 弹窗 `CreateOrUpdate/components/CreateOptionModal.tsx`（内层依次为 `CreateOptionForm.tsx` 组级表单 → `OptionValues.tsx` option 列表 → `CreateOptionValue.tsx` 单个 option 表单）。互斥对象 `in_eligible` 就在 `CreateOptionValue.tsx:218-249`，形态是**一个 Switch + 右侧 `Yes`/`No` 文字**，label 带 Popover（`Ineligible` / `Only hide the option for the main menu item from Wonder App.`），variant item 时禁用。
> - **Preset 侧** —— `Presets/Edit/OptionTable/OptionTableEdit.tsx`，`in_eligible` 形态是**表格里的 Checkbox 列**，已选中的 option 会被禁用并提示 `Cannot set selected option as ineligible.`（`:383-388`）。
> - **`none` 选项的识别**：字段为 `is_none`（`CreateOrUpdate/type.ts:75`），无需额外判断逻辑。
> - **已存在一条互斥规则**：`in_eligible` 与 `is_default_value` 不能同时为真，报错 `Unable to save, cannot set default option to ineligible: {name}.`（`useCreateOptionAction.tsx:329-333`、`560-562`）。本次的 `required` ↔ `in_eligible` 是第二条互斥，两条需并存。按需求示例（chicken 为 required 且 default portion = 2），`required` 与 `is_default_value` 是允许共存的。
> - **数量校验有现成模式可复用**：现有 max/min choices 校验已在 4 处使用 `option_values.filter(it => !it.is_none).filter(it => !it.in_eligible)`（`CreateOptionForm.tsx:325/374/477/518`），required 的数量校验在此链上追加 required 过滤即可。
> - **`required` 的控件形态跟随 `in_eligible`**：main 侧用 Switch + `Yes`/`No`，preset 侧用表格 Checkbox 列 —— 两处形态不统一是既有实现的现状，`required` 各自沿用所在位置的形态，不另做统一。
> - 其余改动点：change history 对比（`changeHistory/widget/Customization.tsx`、`PresetsCustomization.tsx`）、copy item / new version 的字段继承。


### [MD-18356](https://wonder.atlassian.net/browse/MD-18356)

保存 line build 时自动清理已失效的 restaurant 引用

> 来源反馈：Slack `https://remarkable-foods.slack.com/archives/C01KS4JM38A/p1785520710104399`

**背景**：menu item 与 restaurant 的关系链是：menu item → 打的 concept → concept 关联 brand → brand 关联 restaurant。这条链上任一环变化，line build 里之前打的 restaurant 就会变成 **UUID** 显示，并**阻断保存**。用户必须手动一个个删掉这些无效 UUID 才能保存，而一个 brand 下可能有很多 restaurant，非常繁琐。

会导致 restaurant 变成 UUID 的操作：
- 从 line build 移除一个 concept
- 从 menu item 所打的 concept 上移除一个关联 brand
- 在 Merch Tool 里把某个 restaurant 从 brand 上移除

**需求**

1. 保存 line build 时，弹窗告知用户该 menu item 已不在某些 restaurant 售卖（**显示 restaurant 名称，不是 UUID**）；用户确认后自动批量移除这些无效引用，并让 line build 正常保存成功
2. 文案：
   - Header：`Invalid Restaurant`
   - 正文：`The main menu item has no longer sold in the below restaurants, is it ok to remove them for saving the line build? Invalid restaurants: {restaurant name1}, {restaurant name2}.`
   - 按钮：`Cancel`、`Yes`

> 前端关注点：
> - **保存流程已有现成的 warning modal 队列机制可直接挂载**：`src/page/item/lineBuild/component/Header/saveHelper.tsx`（1257 行）中已有 `KDSPortionWarningSection` / `buildKDSPortionWarningModal`（MD-18030 的 KDS portion warning）与后端驱动的 `not_machine_eligible_ik_step_warnings`（MD-18149），并已按「hard block（直接拦截）」与「non-blocking warning（确认后继续）」分层；`useSave.tsx`（564 行）按顺序 await 这个 ModalQueue。`Invalid Restaurant` 弹窗接入这套机制即可，无需新建流程。
> - **但它与现有 warning 的形态不同**：现有 warning 都是「确认后原样提交」，本条是「确认后需先移除失效引用、再提交」，属于队列里的新形态。
> - 属于当前正在改动的 line build 区域（`src/page/item/detail/pages/lineBuildList/`、`src/page/item/lineBuild/`），与 MD-18336 有代码重叠风险。
>- 假设某个 line build 只挂了 3 家餐厅，结果这 3 家全都因为 concept/brand 链变动而失效了。如果严格按 ticket 说的"确认后自动批量移除"，那移除完剩 0 个，请求体里 restaurant_ids 就是 null。用户点了一下"Yes"，本意是"删掉那几家失效的"，实际结果却是这个 line build 从"只作用于 3 家店"变成了"作用于全部门店"。
---

### [MD-18347](https://wonder.atlassian.net/browse/MD-18347)

从 Menu Item / 7\* Item 的 component 与 customization 中移除 `Create New Packaged` 功能（代码级清理）

> 主 ticket（Epic）：[MD-18363](https://wonder.atlassian.net/browse/MD-18363) Feature simplify

能清理的代码不多，20几行代码但是测试是需要回归测试的。

---

### [MD-18375](https://wonder.atlassian.net/browse/MD-18375)

在 Line Build 编辑页增加「回到顶部」快捷按钮

> 主 ticket（Epic）：[MD-17264](https://wonder.atlassian.net/browse/MD-17264) @2026 Regular optimization in Cookbook

**背景**：Line Build 编辑页的 step 数量很多，用户要靠长时间拖动 / 滚动才能在其间导航，编辑效率低。

- 在 Line Build 编辑页增加一键「back to top」功能，让用户不必手动滚过所有 step 就能快速回到页面顶部

---



### [MD-18383](https://wonder.atlassian.net/browse/MD-18383)

UI - 允许把整个 customization group 标记为 ineligible

> 主 ticket：[MD-18341](https://wonder.atlassian.net/browse/MD-18341)

**背景**：Cookbook 允许对 main BYO menu item 与 preset 分别配置 customization（ineligible / min options / max options 等），此前有一条「每个 customization group 至少要有一个 eligible option」的校验。而 Wonder Create 项目要求限制顾客把默认选项换成更低成本的选项，因此当某个 group 的所有 option 在某个 preset 里都没被选中时，Cookbook 要能把整个 group 在该 preset 上标记为 ineligible。后端已把「至少一个 eligible option」的下限**从 per-customization 改为 per-menu-item**，服务端不再拦截整组 ineligible。

**需求**

1. **去掉「每个 customization group 至少要保留一个 eligible option」这条限制**，允许把整个 group 的 option 全部标为 ineligible。以下操作都不再做这项校验：
   - 新建 / 编辑 customization（main menu item 侧）
   - 编辑 preset 的 customization
   - 删除 option —— 原本最后一个 eligible option 的删除按钮是禁用的，现在放开
2. **preset 的 customization 表格里，`Ineligible` 勾选框不再被置灰。** 原本的置灰规则是「快要没有 eligible option 时就不让再勾」，分两种：
   - **非 multi-select**：当剩余 eligible option 的数量降到 min options 时，其余仍是 eligible 的行就不能再勾，提示 `The number of eligible options should >= {min options}`
   - **multi-select**：当只剩最后 1 个 eligible option 时，那一行不能再勾，提示 `The number of eligible options should >= 1`
   - 后果是「整组 ineligible」这个状态在界面上永远走不到 —— 例如一个 group 有 3 个 option，勾掉前两个之后第三个就变灰了，卡死在「至少留一个」
3. **校验层级由 group 改为 menu item**：单个 group 全部 ineligible 是允许的；只有当该 menu item / preset 的**所有** featured customization 都没有 eligible option 时才拦下来，报错 `Unable to save. At least one eligible option is required for the entire featured customization.`
4. **保持不变**：已被选中（selected）的 option 仍不能设为 ineligible；min / max options 与 eligible 数量的校验规则原样保留

---

### [MD-18385](https://wonder.atlassian.net/browse/MD-18385)

UI - 调整 Wonder Create Debug Page 的 customization 逻辑

> 主 ticket：[MD-18372](https://wonder.atlassian.net/browse/MD-18372)

- 该 sub-task 与其主 ticket 均无描述，需求待补

---

### [MD-18401](https://wonder.atlassian.net/browse/MD-18401)

UI - New UX - Line build 分配：强制只有一个 'All' 选项，并提供优雅的切换流程

> 主 ticket：[MD-18392](https://wonder.atlassian.net/browse/MD-18392)
> 本 sub-task 只覆盖 **single version**；multi-usage / multi-version 见 [MD-18397](https://wonder.atlassian.net/browse/MD-18397)

**背景**：Hudson Square 把 Pizza 菜单指向新的 Raw Dough 试点菜单后出现分配问题 —— 受影响的 menu item 上每条 line build 都绑定了具体 restaurant，**没有一条应用到 `All`**，IKC 找不到可兜底的默认 line build。原以为已有护栏保证 `All` 始终存在，实际并没有。

根因是现状把两件事耦在了一起：`Apply To Restaurant(s)` 是 line build 编辑页表单里的一个字段，跟 task 一起保存。于是「这个 menu item 有没有 All」这个问题，只能在每条 line build 各自保存时局部地看，无法在正确的时机整体校验。

目标：
1. 一个 menu item 下**只能有一个** line build 应用到 `All`
2. 有 line build 的 menu item **必须**有一个 `All` line build（不在 IKC 烹饪的 menu item 没有 line build，不受此约束）


**需求**

1. **创建 line build**
  ![alt text](image-1.png)
   - 一条都没有时 → 面板显示空状态 `No line build yet` / `The first line build you create must apply to All restaurants.` + 居中按钮 `+ Create Line Build`；右上角原按钮去掉
   - **第一条恒为 `All`**，没得选所以**不弹窗**，点了直接进编辑页
   - ![alt text](image-2.png)
   - 已有 line build 时 → 创建入口挪到 sub-tab 末尾的 `+`，点了弹窗选 restaurant，至少选一个才能继续；**提供 `All` 选项**（能不能真存下去交给后端校验）
   - 点确认先调后端 check；不通过则弹窗不关、原地报错
   - **弹窗确认时不创建任何数据** —— 只收集 restaurant，真正的创建发生在编辑页点 Save 时。因此用户在编辑页放弃退出不会留下空的 line build
   - sub-tab 命名由 `Line Build 1` 改为 `Line Build 1(All)`，All 那条带 `(All)` 后缀

1. **编辑 restaurant（解耦出来的独立入口）**
   - 每条 line build 的 `⋮` 菜单新增 `Assign Restaurants`，打开弹窗、预填当前 restaurant
   - 可选具体 restaurant 或 `All`；保存时调后端 check，不通过则原地报错
   - **只改 restaurant 归属，不动这条 line build 的 task**
   - 配套：**编辑页里的 `Apply To Restaurant(s)` 全场景置灰** —— create / duplicate / edit 三种进入方式都不可编辑

2. **restaurant 选择器用现有组件**
   - 弹窗里的选择器**不新做**，直接用编辑页现在那个（抽成公用组件），保证两处长得一样、行为一致
   - 现有组件已自带：多选、搜索、已选 tag、一键清空、`All` 选项、HDR 品牌标签、长名 Tooltip、loading 态，以及「选 `All` 就清掉具体 restaurant、选具体 restaurant 就去掉 `All`」的互斥逻辑

3. **编辑页离开时的二次确认**
   - 从新页面进 create / edit 编辑页后，**只要表单动过，离开就弹确认**；没动过则直接走，不拦
   - 覆盖三个出口：`Cancel` 按钮、站内跳转 / 浏览器返回、刷新或关标签页
   - 参考 assembly 的现成实现（`assemblyInstructions/component/AssemblyInstructionIndex.tsx:167-180`）：标题 `Unsaved Changes`、正文 `You have unsaved changes on the form. Are you sure you want to leave?`、确认按钮 `Continue`；脏判断用一个 flag 记录用户是否真的改过
   - ⚠️ create 模式进来时页面已经自动生成了一个 3 步空脚手架，**不能把脚手架本身当成「改过」**，否则用户点错进来想退出也会被拦

4. **从其他 line build 复制 details**
   - `⋮` 菜单新增 `Copy from other Line Build`，二级菜单列出**除当前这条以外**的所有 line build，选中后二次确认「会覆盖当前这条的数据且不可撤销」
   - **只覆盖 task，不改 restaurant 归属** —— 这是它和 `Duplicate` 的根本区别
   - 只有一条时禁用，提示 `No other line build to copy from`
   - 用途：想让 line build 2 的做法成为默认，就把它的 details 复制到本来是 `All` 的 line build 1

4. **Duplicate**
   - 现状：直接跳编辑页，在编辑页里选 restaurant
   - 改为先弹窗分配 restaurant（复用创建那个弹窗），check 通过后跳编辑页、带入复制出来的数据，restaurant 置灰

5. **删除 line build**
   - 现状：无条件可删
   - 点 `Delete Line Build` 先调后端 check，按返回渲染三种结果之一：不允许删（展示后端给的原因）/ 允许删（二次确认）/ 允许删但要指定接班人（弹窗让用户从候选里选一条接管 `All`）
   - 只剩一条时可删（不在 KDS 烹饪的 menu item 本来就不需要 line build）

6. **校验全部由后端驱动**
   - 「有且只有一个 `All`」这条规则**前端不实现**，只调 check 接口 + 渲染返回结果。三个调用点：创建弹窗确认、`Assign Restaurants` 保存、`Delete` 点击

7. **本次只做 single version**
   - 新交互只对 `Is Multi-usage qty Item` 与 `Multi versions vs options` **都关闭**的 item version 生效；任一开启的保持现有界面完全不变（含 `Configuration` 下拉）。新界面上**不展示 `Configuration`**
   - 原因：开了开关的 item version，每条 line build 必须带 scope（`Apply To Option` / `Option Value`）。新创建流程不收集这些字段，让这类 item 走新流程会落下 scope 为空的脏数据或直接被后端拒绝


**待确认问题**

1. **在编辑页刷新怎么办？** 弹窗选的 restaurant 只在前端内存里，刷新就没了。目前方案是退回 Line Build tab 让用户重来（此时他一个字都还没填，损失为零）。若要求刷新后保住选择，弹窗确认时就必须落库，等于回到「会产生空 line build」的模型，要额外做清理逻辑
2. **`Assign Restaurants` 能不能真的把默认切到另一条？** Case 2 说保存时校验「有且只有一个 `All`」。那从 `{LB1=All, LB2=A,B}` 切成 `{LB1=C,D,E, LB2=All}` 该怎么操作？先改 LB1 会变成零条、先改 LB2 会变成两条，两边都触发这条校验。是不是意味着它只能增减具体 restaurant、换默认只能走第 4 点的 copy？
3. **隐藏 `Configuration` 的三个连带影响**：① 用户没法再把 single version item 改成 multi，过渡期能接受吗 ② 这两个开关是 **item version 维度**（不是 item 维度），同一个 item 切版本时界面会在新旧之间跳（V6 multi / V7 single）③ `Create New Version` 时开关继承吗，若继承则现存 multi item 在 MD-18397 之前永远进不了新界面

---

### [MD-18396](https://wonder.atlassian.net/browse/MD-18396)

UI - Concept / Brand 列表页的 Create 按钮位置优化

- 现状：首次打开 Concept / Brand 列表页时看不到 `Create` 按钮（需滚动或其他操作才出现）
- 调整按钮位置，使其在页面首屏即可见


### [MD-18377](https://wonder.atlassian.net/browse/MD-18377)

CLONE - [Tech] 给所有功能补齐 Amplitude 埋点

> 主 ticket（Epic）：[MD-17231](https://wonder.atlassian.net/browse/MD-17231) @2026 Cookbook Technical Excellence

- 上一轮已覆盖导入 / 导出入口，本轮继续排查全站其他关键操作的遗漏埋点

---