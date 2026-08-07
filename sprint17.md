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

---

### [MD-18347](https://wonder.atlassian.net/browse/MD-18347)

从 Menu Item / 7\* Item 的 component 与 customization 中移除 `Create New Packaged` 功能（代码级清理）

> 主 ticket（Epic）：[MD-18363](https://wonder.atlassian.net/browse/MD-18363) Feature simplify

能清理的代码不多，

> 前端关注点：menu / 7\* 的屏蔽已由 `ComponentsV2/Action/index.tsx:56` 的 `showCreatePackagedItem = object_type !== MENU && object_type !== HDR_RECIPE` 实现，入口按钮已不渲染；88\* 要保留该功能，所以 `CreatePackage/` 组件、按钮、弹窗都必须留着，那行判断本身也不能删。实际可清理的只有两处：`Action/Edit.tsx:4` 未使用的 `import CreatePackageItem`，以及 `Action/ShowTable.tsx:130` 的 `handleEdit` 中 `if (r.isCreateNewItem)` 分支缺少 item type 判断。

---

### [MD-18375](https://wonder.atlassian.net/browse/MD-18375)

在 Line Build 编辑页增加「回到顶部」快捷按钮

> 主 ticket（Epic）：[MD-17264](https://wonder.atlassian.net/browse/MD-17264) @2026 Regular optimization in Cookbook

**背景**：Line Build 编辑页的 step 数量很多，用户要靠长时间拖动 / 滚动才能在其间导航，编辑效率低。

- 在 Line Build 编辑页增加一键「back to top」功能，让用户不必手动滚过所有 step 就能快速回到页面顶部

---

### [MD-18377](https://wonder.atlassian.net/browse/MD-18377)

CLONE - [Tech] 给所有功能补齐 Amplitude 埋点

> 主 ticket（Epic）：[MD-17231](https://wonder.atlassian.net/browse/MD-17231) @2026 Cookbook Technical Excellence

- 本条是 Sprint 16 的 [MD-18317](https://wonder.atlassian.net/browse/MD-18317) 的 clone，需求相同：Amplitude 客户端已存在于 `src/utils/analytics/`，只需补覆盖
- 上一轮已覆盖导入 / 导出入口，本轮继续排查全站其他关键操作的遗漏埋点

---

### [MD-18383](https://wonder.atlassian.net/browse/MD-18383)

UI - 允许把整个 customization group 标记为 ineligible

> 主 ticket：[MD-18341](https://wonder.atlassian.net/browse/MD-18341)

**背景**：Cookbook 允许对 main BYO menu item 与 preset 分别配置 customization（ineligible / min options / max options 等），此前有一条「每个 customization group 至少要有一个 eligible option」的校验。而 Wonder Create 项目要求限制顾客把默认选项换成更低成本的选项，因此当某个 group 的所有 option 在某个 preset 里都没被选中时，Cookbook 要能把整个 group 在该 preset 上标记为 ineligible。后端已把「至少一个 eligible option」的下限**从 per-customization 改为 per-menu-item**，服务端不再拦截整组 ineligible。

**1. 移除前端的前置校验（阻塞项）**

前端自己也在调接口前做了「每个 group 至少一个 eligible option」的校验，**不删掉的话后端改动对用户完全不可见**（浏览器里照样被拦）。三处：

- `src/page/item/customizationV2/component/CreateOrUpdate/components/CreateOptionForm.tsx:547` —— `Promise.reject("Please ensure at least 1 eligible option.")`
- `src/page/item/customizationV2/component/Featured/OptionTableView.tsx:180` —— validator 返回同一字符串
- `src/page/item/customizationV2/component/Presets/Edit/PresetFrom.tsx:197`、`:220` —— `message.error(...)` 同一字符串

服务端的对应校验是**被删除**而非放宽 —— `Unable to save. Please ensure at least 1 eligible option.` 这条消息在后端已不存在。

**2. 需要承接的新服务端报错**

per-group 规则被 per-menu-item 规则替代，后端原样返回：

```
Unable to save. At least one eligible option is required for the entire featured customization.
```

仅当该 menu item / preset 的**每一个** featured customization 都没有 eligible option 时才触发；按设计它**不指名**具体是哪个 customization。

**3. 消息优先级变化**

新的 item 级校验**最后**执行。当 per-group 规则（min/max options、default 不可 ineligible 等）与 item 级规则同时被违反时，用户看到的是 per-group 消息（因为它指名了具体 customization）。任何断言消息顺序的测试 / 快照需要更新。

---

### [MD-18385](https://wonder.atlassian.net/browse/MD-18385)

UI - 调整 Wonder Create Debug Page 的 customization 逻辑

> 主 ticket：[MD-18372](https://wonder.atlassian.net/browse/MD-18372)

- 该 sub-task 与其主 ticket 均无描述，需求待补
- 涉及页面为 Sprint 13 建的 Wonder Create 调试页（路由 `/wonder-create-debug`，见 MD-18057 / MD-18126）

---

### [MD-18386](https://wonder.atlassian.net/browse/MD-18386)

UI - Line build 分配：强制只有一个 'All' 选项，并提供优雅的切换流程

> 主 ticket：[MD-18369](https://wonder.atlassian.net/browse/MD-18369)

**背景**：Hudson Square 把 Pizza 菜单指向新的 Raw Dough 试点菜单后出现了分配问题 —— 很多试点 line build 没有可供 IKC 兜底的 "All" 选项，只有两个 line build 分配给了少量 IKC。原以为已有护栏保证 "All" 始终存在，实际并没有。现状是：创建 line build 时 `Apply to Restaurant` 默认为 `All`，但用户可以手动改成具体 restaurant，而 line build 是各自独立创建 / 编辑的，没有任何机制防止「缺少 All line build」。

**基本约束**

- 一个 menu item 下**只能有一个** line build 映射到 `All` restaurants
- 有 line build 的 menu item **必须**有一个 `All` line build（不在 IKC 烹饪的 menu item 没有 line build，不受此约束）
- **强制第一个**创建的 line build 应用到 `All`，并把 `Apply To Restaurant(s)` 置灰

**多版本场景下的「每个维度都要有 All」**

- `Multi versions vs options = true` 时，**每个 option value** 都需要一个 apply to `All` 的 line build
  - 例：customization `Choose Your Protein` 的 option value 为 Chicken / Tofu / Beef / No Protein，若 line build 1 是「Chicken + All」，则还必须有 Tofu / Beef / No Protein 各自的 All line build
  - 不满足时，在 line build 页 **Bad Data 消息的最顶部**提示
- `Is Multi-usage qty Item = true` 时，**每个 qty 区间**都要有 `applied restaurant = All` 的 line build
  - 例：已有「Cresskill + 1-3」和「Jake Downtown + 4-max」两个 line build，则还必须有「All + 1-3」和「All + 4-max」
  - 不满足时同样在 Bad Data 消息最顶部提示

**创建 / 编辑时的校验**

- 创建非第一个 line build 且 `multiple versions = false`：用户可选 restaurant，若选了 `All` 则检查是否已存在其他 `All` line build，存在则报错并**阻止把值填入** `Apply To Restaurant(s)`
  - `There is already one line build for All' restaurants.`
- 创建非第一个 line build 且 `multiple versions = true`：若选了 `All` 且指定了 `Option Value` / `Apply To Value(s)`，检查是否已存在「映射该 option 且 apply to All」的 line build，存在则报错并阻止填入
  - `There is already one line build mapped with this option/option value for All' restaurants.`
- 上述两条校验同样要作用于**任何 line build 的编辑**（用户可能在修正历史脏数据）
- 创建非第一个 line build 且 `multiple versions = true`、用户指定了 `Apply To Value(s)`：检查是否已存在「映射该 option 且 apply to All」的 line build
  - 已存在 → 无动作
  - 不存在 → 弹窗把 `Apply To Restaurant(s)` 默认改为 `All`
    - Header `Change Apply to Restaurants`
    - 正文 `There must be line build applied to All restaurants for {option name}, will change the apply to restaurant to All. Please create another line build for specific restaurant later.`
    - 按钮 `Cancel` / `Confirm`；`Cancel` 清空 `Apply To Value(s)` 里的该 option，`Confirm` 把 `Apply To Restaurant(s)` 改为 `All` 并置灰
  - `selected option value amount` 场景遵循同样逻辑

**删除时的「提升」流程**

- 只有一个 line build 时，**不**置灰删除操作
- 超过一个 line build 时，删除的是 `All` 那个，则弹窗让用户指定另一个现有 line build 作为 `All`
  - Header `Promote Line Build to All`
  - 正文 `There must be line build applied to All restaurants, please promote another line build to 'All' before deletion.`
  - 单选列表，每项格式 `Line build #-{option name}（或 selected option value amount {amount}），apply to restaurants: {restaurant 1}, {restaurant 2} (show more)`（默认显示 2 个 restaurant，hover 展示全部）
  - 按钮 `Cancel` / `Confirm`
  - 候选列表规则：排除被删除的那个；排除其他 `All` line build（multiple 开启时不同 option 可各有一个 All）；排除 option 不同的；排除 selected option value amount 不同的；按 line build 编号**升序**排列

**移除的旧校验**

- 移除现有「缺少与所选 restaurant + 指定 option 相匹配的 line build」这条校验 —— 因为现在已保证存在覆盖全部 restaurant 的 line build

---

### [MD-18396](https://wonder.atlassian.net/browse/MD-18396)

UI - Concept / Brand 列表页的 Create 按钮位置优化

- 现状：首次打开 Concept / Brand 列表页时看不到 `Create` 按钮（需滚动或其他操作才出现）
- 调整按钮位置，使其在页面首屏即可见
