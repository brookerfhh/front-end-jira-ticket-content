# Front-end Jira Tickets — Sprint 17 会前需求理解（Sprint 17）

> 更新时间：2026-08-05
> Sprint：`MD 2026 Sprint 17`（board 10，sprint id 50621），周期 **2026-08-10 → 2026-08-24**
> 查询条件：`sprint = 50621`（board 上共 9 条），本文档按要求排除 **MD-18313**、**MD-18254**
> 状态：**Sprint 17 尚未开 planning 会**。9 条全部为 `To Do`，且**均为 Story/Task 层级，尚未拆出 `UI - xxx` 前端 sub-task**（唯一例外是 MD-18122 已有前端 sub-task MD-18151）。本文档用于会前预读与提问准备。

| Key | 摘要 (Summary) | 前端工作 |
| --- | --- | --- |
| [MD-18339](https://wonder.atlassian.net/browse/MD-18339) | Deprecate existing training card with new design | ✅ 纯前端（大） |
| [MD-18346](https://wonder.atlassian.net/browse/MD-18346) | Add "Required" Flag to Customization Options to Enforce Menu Item OOS | ✅ 前端 + 后端（大） |
| [MD-18122](https://wonder.atlassian.net/browse/MD-18122) | IK Portion Conversion Validation in Cookbook | ✅ 前端 + 后端（大，sub-task MD-18151 已在我名下） |
| [MD-18356](https://wonder.atlassian.net/browse/MD-18356) | Auto-remove stale restaurant references from line build on save | ✅ 前端 + 后端（中） |
| [MD-18347](https://wonder.atlassian.net/browse/MD-18347) | Remove Create New Packaged Feature/Workflow from Menu Item/7\* Item | ✅ 纯前端（小，死代码清理） |


---

## 前端需求拆解

### [MD-18339](https://wonder.atlassian.net/browse/MD-18339)

用新设计重做 Line Build 的 Training Card 导出（PDF）

> 需求正文不在 Jira 上：Jira 只给了一个链接，完整规格在 Confluence [RA-New Training Card](https://wonder.atlassian.net/wiki/spaces/RT/pages/5521768552/RA-New+Training+Card)（作者 Bonnie Baohuan，2026-08-03 更新）
> 参考数据：`https://cookbook.foodtruck-uat.com/ItemV2/detail/basic/8011525?version_id=470e091b-3e30-476e-80c5-ca6d58039d2c`

**背景**：现有 training card 组件需要按新设计整体替换，以改善体验并对齐当前设计规范。

**需求（来自 Confluence 规格）**

- **Header**（左右分布，取自设计稿）
  - 左上：item name（设计稿示例 `K-TOWN FRIES`）+ `wonder` logo
  - 右上：`Dish ID: {menu item number}`、`Launch: {service start date}`（示例 `Launch: 07/15/2026`）
  - 现有实现里的 **restaurant names** 与 **版本行（`V1 Draft`）在新设计中都不存在**，需移除
  - Header 在每页重复（沿用现有 `fixed` 行为）

- **左栏**
  - 顶部成品图
  - **Guest Packaging**
    - 有则显示子标题 `Main Packaging`，无则不显示
    - 列出 component 里的 9\* item，每个都显示 item number
    - 若 customization option 里有 9\* item，显示子标题 `Customization Packaging`，列出并**按 item number 去重**
    - 设计稿呈现为 bullet 列表：`• Large vented clamshell: Clamshell 8X6X3-1/2" (9003794)`
  - **Equipment Needed**
    - 聚合 line build 中 cook activity 的 appliance 与 global configuration
    - 呈现为**一行、逗号分隔**：`Turbo Oven 100/90 475 °F, Fryer Basket 350 °F`
    - 名称取 **Cookbook 里的值**：`Turbo Oven`（不是 `TurboChef Oven`）、`Fryer Basket`（不是 `Fryer`），且要带 global configuration（`100/90 475 °F`，不是只有 `475 °F`）
    - 无 cook step 时整个 section 隐藏
  - **Customizations**（设计稿中标题写作 `Modifications:`）
    - 显示全部 customization，每个 bullet 一行，行内的 option 用逗号分隔。**两类 type 的分行维度不同**：
      - `Mandatory Choice` / `Optional Addition` / `Dish Preference` —— **按 group 分行**，冒号前为 group name：`{customization name} - {option name}, {option name}`
        - 例：`Choose Your Sauce - Sriracha Mayo (Spicy), Bulgogi Mayo`
      - `subtraction option` / `on the side` / `Extra Request` —— **按 type 分行**，同一 type 下所有 group 的 option 合并进一行，冒号前为 type name：`{customization type} - {option name}, {option name}`
        - 例：`Optional Subtraction - No Bulgogi Mayo, No Kimchi, No Togarashi`（这三项在 Cookbook 中是三个独立 group，设计稿合并为一行）
        - 例：`On the side - Tahina on the side, Red Sauce on the side`
    - 排序与 customization 数据一致；无 customization 时隐藏该 section
    - 与现有实现的差异：现在右栏 `Possible customizations` 是 **option 级别**（一个 option 一个带 `+` / `-` / `→` 图标的 tag 框），新设计改为上述行级文本，行数大幅压缩
  - **Ingredients Table** —— **两列表格**
    - 列：`Ingredient List` | `Tool`
    - `Ingredient List` 格式 `{item name} ({item number})`，可带 preparation（示例 `Kimchi (Drained) 4000315`）
    - `Tool` 取 packaged SKU 的 `SMALLWARE TOOL` **完整值**（示例 packaged SKU 上是 `Shaker / Shaker - White`，设计稿简写成 `White Shaker`，要求显示完整值）
    - 显示 component 里的 food item；无 food item 时隐藏整个表格
    - 长文本在单元格内自动换行（设计稿首行 `Fries, French, Fridge Friendly, 5/16" (Buyout) HC (4000053)` 即为两行）

- **右栏** —— **两列图片网格**（与现有「单列横向 box」完全不同）
  - 表头：`Line Build`
  - 多 task 时显示子标题 `Task {task name} ({customization name} - {option name})`，其下再列 step；**只有一个 task 时不显示子标题**
    - 按 line build 的 task 顺序排序
    - 例：`Task: Default`、`Task: Sriracha Mayo (Choose Your Sauce - Sriracha Mayo (Spicy))`
  - 每个 step 一个网格单元，**两列并排换行**，单元内自上而下为：图片 → step 标题 → substep bullet
    - 图片带**绿色边框**与**绿色圆圈序号角标**
    - cook 类型的 step 用 `{appliance} ({cook time})` 作为标题（示例 `Turbo Oven (2:30)`、`Fryer (4:00)`）；非 cook step 无此标题行，直接显示 substep bullet
  - 导出所有 step（**除 vend step**）
  - 图片只取 `step image=true` 的，一个 step 有多张时**取第一张**；没有则留空
  - 显示 step 下 substep 的 ingredient / step title
  - 打印页脚：`当前页/总页数`

- **其他**
  - 用户可能从 QA/UAT/PROD 导出，PDF **按环境继承主题色** —— 复用项目已有的 `PRIMARY_COLOR`（`src/utils/constants.ts:3`）：`config.env === "PROD"` 为 `#A0006B`，其他环境为 `#3368B5`（仅两档，非三套色值）。现有 PDF 未使用该变量，色值全为硬编码（`#276EF1`、`#1a1b25`、`#6a7383`、`#c42747`、`#bf6817`）
  - 左右栏内容超长时各自往下页续排（即现有默认的翻页行为，左右栏高度不等时短的一侧留白，与现状一致）
  - **取消**现有的「选择要导出哪些 step」弹窗
  - 按钮从 `Training Card` 改名为 `Export Training Card`
  - 已确认无需在 PDF 底部标注 apply to restaurants（training card 是 line build 级别）

> 前端关注点：现有实现在 `src/page/item/detail/pages/lineBuildList/component/TrainingCard/`，共约 874 行 —— `Preview/index.tsx`（395 行，PDF 布局主体）、`TrainningCardModal.tsx`（130 行，含要删掉的 step 选择弹窗）、`Preview/imageCache.ts`、`Preview/RenderImage.tsx`、`DataController/`（step 勾选器，本次删除）。PDF 由前端 `@react-pdf/renderer` 渲染（全项目仅此一处使用，当前 `3.1.17`）。新设计中左栏 5 个 section 全新、右栏由单列横向 box 改为两列图片网格，等价于重写 `Preview`。附带两点：`Preview/index.tsx:172` 处 `is_step_image_selected` 的过滤当前是注释掉的，新设计要求启用；表格与网格单元需逐个加 `wrap={false}` 以避免被页边界切开（现有 step box 未加，导出时会出现半个空框），但 `wrap={false}` 不可加在表格容器上，否则超过一页的部分会被裁掉。

**疑问**

- **需知情：内容跨页时，左栏各 section 标题与 `Ingredient List` 表头不会在第二页重复。** 现有 PDF 中 `Cooking steps` / `Possible customizations` 栏标题之所以每页都有，是因为显式加了 `fixed`（`Preview/index.tsx`），而 `Task: xxx` 子标题没加、所以不重复 —— react-pdf 的默认行为是不重复。`fixed` 的语义是「该元素在每一页都渲染一次」，只适用于页眉页脚；若给栏内中部的 section 标题加 `fixed`，它会在每页都画一遍并挤占内容位置，不是「续页补表头」的效果。因此对位于栏内中部的标题，技术上只有「不重复」一个可行选项（与已定的「Header 每页保留、其他 title 不重复」一致）。此项主要用于避免 QA 将「第二页表格没有表头」判为缺陷。
- **需知情：启用 `Step Image` 过滤后，历史数据里原本有图的 step 会变成没图。** 规则本身需求已写明（「Get the image with `step image=true`；没有则左栏 skip、右栏 leave blank」），无需再定。要提醒的是它对现存数据的影响：现有实现只判断 substep 是否上传了图片（`Preview/index.tsx:172`，按 `is_step_image_selected` 过滤的那行是注释掉的），而 `Step Image` 复选框（`StepDisplay.tsx:63`）是纯人工勾选、**默认不勾** —— 新建 substep 为 `null`（`initCreateProcedures.ts:43`），删除图片时重置为 `false`（`StepVisual.tsx:45`）。因此凡是「传了图但当初没勾」的 step，改造后图片会消失（该复选框上线前创建的 line build 字段必然全为 `null`）。按新规则这属于正确行为，但对用户是可见的倒退，需确认已知情。
- **取消 step 勾选器后，`Export Training Card` 的落点。** 现状是两段式：点 `Training Card` 打开弹窗（左侧为 step 勾选列表 + `Show possible customizations` 开关，右侧为实时预览），再点 `Download as PDF` 才下载。勾选器去掉后弹窗只剩预览 —— 需确认是点按钮直接下载，还是保留一个只含预览的弹窗。
- **导出前的等待时间是否可接受？** PDF 需等全部图片预加载完成（`preloadImages`，按 500x500 取）才能生成，新设计左栏增加了成品图，step 较多的 line build 等待会更明显。
- **主题色具体应用到 PDF 的哪些元素？**（PDF 是静态文档，没有按钮，需要逐项明确）设计稿中为绿色系、可能改为主题色的有：左栏 `Dish Details` 等 section 标题、`Ingredient List` / `Tool` 表头底色、右栏 `Line Build` 标题、`Task: xxx` 子标题（设计稿为红字）、图片边框、图片序号角标圆底、`wonder` logo。现有 PDF 已有的颜色有：`Done` 蓝块（`#276EF1`）、customization tag 三色（add `#276ef1` / subtraction `#c42747` / on the side `#bf6817`）、正文 `#1a1b25`、辅助文字 `#6a7383`、边框 `#ddd`、页脚 `grey`。
  - 其中 customization tag 的三色与 `Task: xxx` 的红字属于**语义色**（用于区分 subtraction / on the side / addition），若统一刷成主题色将失去区分能力 —— 需确认是否只有装饰性元素跟随主题色。
  - `wonder` logo 若需按环境变色，需要设计提供对应素材：项目现有资源仅 `asset/img/wordmark_white.webp`（白色版），设计稿中为绿色 wordmark。

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

> 前端关注点（代码定位，模块 `src/page/item/customizationV2/`）：
> - **Main menu item 侧** —— create/edit customization 弹窗 `CreateOrUpdate/components/CreateOptionModal.tsx`（内层依次为 `CreateOptionForm.tsx` 组级表单 → `OptionValues.tsx` option 列表 → `CreateOptionValue.tsx` 单个 option 表单）。互斥对象 `in_eligible` 就在 `CreateOptionValue.tsx:218-249`，形态是**一个 Switch + 右侧 `Yes`/`No` 文字**，label 带 Popover（`Ineligible` / `Only hide the option for the main menu item from Wonder App.`），variant item 时禁用。
> - **Preset 侧** —— `Presets/Edit/OptionTable/OptionTableEdit.tsx`，`in_eligible` 形态是**表格里的 Checkbox 列**，已选中的 option 会被禁用并提示 `Cannot set selected option as ineligible.`（`:383-388`）。
> - **`none` 选项的识别**：字段为 `is_none`（`CreateOrUpdate/type.ts:75`），无需额外判断逻辑。
> - **已存在一条互斥规则**：`in_eligible` 与 `is_default_value` 不能同时为真，报错 `Unable to save, cannot set default option to ineligible: {name}.`（`useCreateOptionAction.tsx:329-333`、`560-562`）。本次的 `required` ↔ `in_eligible` 是第二条互斥，两条需并存。按需求示例（chicken 为 required 且 default portion = 2），`required` 与 `is_default_value` 是允许共存的。
> - **数量校验有现成模式可复用**：现有 max/min choices 校验已在 4 处使用 `option_values.filter(it => !it.is_none).filter(it => !it.in_eligible)`（`CreateOptionForm.tsx:325/374/477/518`），required 的数量校验在此链上追加 required 过滤即可。
> - **`required` 的控件形态跟随 `in_eligible`**：main 侧用 Switch + `Yes`/`No`，preset 侧用表格 Checkbox 列 —— 两处形态不统一是既有实现的现状，`required` 各自沿用所在位置的形态，不另做统一。
> - 其余改动点：change history 对比（`changeHistory/widget/Customization.tsx`、`PresetsCustomization.tsx`）、copy item / new version 的字段继承。

**疑问**

- **variant item 上 `required` 是否也禁用？** 现有 `in_eligible` 在 variant item 上是禁用的（`CreateOptionValue.tsx:224/241`，`disabled={readOnly || isVariantItem(itemDetail?.version_id)}`），并在 Popover 里追加一句 `Update the field in normal version.`。`required` 是否沿用同样的处理（variant 上禁用 + 同样的提示），还是允许在 variant 上单独配置？

---

### [MD-18122](https://wonder.atlassian.net/browse/MD-18122)

IK Portion 与 BOM 单位换算的校验（前端 sub-task：[MD-18151](https://wonder.atlassian.net/browse/MD-18151)，已分配给我，状态 `To Do`）

**背景**：IK（Intelligent Kitchen）出餐时按 **portion 数量**投料，而 Cookbook 的 BOM component item 按具体单位（通常是 `ea` 或 `g`）记消耗。两者之间目前没有换算机制，导致 IK 出餐时 component item 的用量没有被正确扣减。需要支持配置 portion ↔ BOM 单位的换算映射：IK 仍按 portion 投料，Cookbook 侧按 ea/g 正确扣减。

**需求（6 个入口）**

1. **加 component 时**：menu item 的 component / customization 中添加 component，用量按 BOM 单位存储，若该 component 配置了 IK portion 换算，则在 UI 显示换算后的 portion 数作为参考 tip；未配置则不显示 tip
2. **修改 portion 单位换算配置时**：检查是否存在无法换算成整数 portion 的用量，若有则弹 warning
   - `Are you sure`
   - `The component usage in the following menu item cannot be converted to integer portion for IK. Are you sure you want to save the change?`
   - `{Menu item number1}-usage xx {BOM unit}, {Menu item number2}-usage xx {BOM unit}`
   - 按钮：`Cancel` / `Save`
   - **忽略 expired version、variant、draft version**
3. **修改 menu item component / customization 用量时**，若无法换算成整数 portion，弹 warning（component 与 customization **两套文案**）
   - component：`The following component cannot be converted to integer portion for IK. Are you sure you want to save the component?` + `{Component item number1}, {Component item number2}`
   - customization：`Customization: {option name1} {item number 1} ({IK portion}), {option name2} {item number 2} ({IK portion})`
   - 按钮：`Cancel` / `Save`
4. **发布 menu item 时**：component / customization item（`machine eligible = true`）的用量无法换算成整数 portion 时弹 warning
   - `The following component cannot be converted to integer portion for IK. Are you sure you want to publish the menu item?`
   - 同时列出 `Component` 与 `Customization` 两段
   - 按钮：`Cancel` / `Publish`
5. **Bulk edit** component / customization 用量（object type = menu item）：无法换算时弹 warning
   - `The component usage in the following menu item cannot be converted to integer portion for IK (1 Portion=xx {BOM unit}). Are you sure you want to save the change?`
6. **Bulk swap** component / customization 用量（object type = menu item）：替换后的 component 用量无法换算时弹 warning
   - `The replacement component usage in the following menu item cannot be converted to integer portion for IK (1 Portion=xx {BOM unit}). Are you sure you want to save the change?`

> 前端关注点：
> - **换算配置的入口已存在**，即 MD-18167 建的 `Minimal Serving Portion Conversion` 卡片 —— `src/page/item/detail/components/MinimalServingPortionConfiguration/`。弹窗（`CreateOrEditMinimalServingPortionConfiguration.tsx:152-197`，width 600，标题 `Create / Edit Minimal Serving Portion Conversion`）表单只有一行：`[portion_qty] portion = [bom_usage_qty] {unit}`，两个输入框都是**正整数**（`InputNumber` min 1 / precision 0），单位是**固定文本**而非下拉。已有权限控制（无权限时 Save 禁用 + Tooltip）与 machine eligible 必填提示（`REQUIRED_FOR_MACHINE_ELIGIBLE_TIP`）。需求第 2 点「修改 portion 单位换算配置时校验」即挂在这个弹窗的 Save 上。
> - 其余 5 个入口分布很广 —— ComponentsV2（加 component / 改用量）、customization 编辑、publish 校验弹窗、BulkEditUsage、BulkSwapItem。其中 bulk edit / bulk swap 两处本身就是复杂组件。
> - 所有弹窗都是「非阻断 warning + 二次确认」模式，与 MD-18149（IK Machine Eligible warning）的处理方式一致，可复用其模式。

**疑问**

- **单位为 `ea` 的 component 如何配置换算？** 需求说 BOM component 的单位「typically **ea** (each) or **g** (grams)」，各处 warning 文案也用 `{BOM unit}` / `1 Portion=xx {BOM unit}` 占位符，暗示单位是可变的。但现有换算配置的单位是**写死为 grams** 的 —— `const unit = DEFAULT_BOM_UNIT;`，代码注释为 `The minimal serving portion conversion is always to grams (MD-18167 #1.1)`，UI 上也是固定文本而非下拉。因此 `ea` 单位的 component 目前配不了换算：是需要把该配置扩展为可选单位，还是本次仍只支持 grams、文案里的 `{BOM unit}` 恒为 `g`？
- **第 4 点限定 `machine eligible = true`，第 3、5、6 点未写这个前提** —— 是所有入口都只校验 machine eligible 的 item，还是只有 publish 这一处有此限制？（现有配置卡片已有 machine eligible 必填提示，两者口径需一致。）

---

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
> - **但它与现有 warning 的形态不同**：现有 warning 都是「确认后原样提交」，本条是「确认后需先移除失效引用、再提交」——属于队列里的新形态，需确定移除动作由哪一侧执行（见疑问）。
> - 属于当前正在改动的 line build 区域（`src/page/item/detail/pages/lineBuildList/`、`src/page/item/lineBuild/`），与 MD-18336 有代码重叠风险。

**疑问**

- **用户点 `Yes` 之后，失效引用由哪一侧移除？** 前提是后端在保存接口里返回带 name 的失效 restaurant 列表（这样「名称从何而来」属于后端内部实现，前端不需关心）。确认后有两种契约：① 前端自行把失效项从表单数据中剔除，再重新提交完整 payload；② 前端带一个「已确认移除」的标志重新提交，由后端剔除。这决定前端是否要写移除逻辑，也是本条与现有 warning（均为「确认后原样提交」）的唯一结构性差异。

---

### [MD-18347](https://wonder.atlassian.net/browse/MD-18347)

从 Menu Item / 7\* Item 的 component 与 customization 中移除 `Create New Packaged` 功能（代码级清理）

> 主 ticket（Epic）：[MD-18363](https://wonder.atlassian.net/browse/MD-18363) Feature simplify

**背景**：以前用户在 menu item component 里用 stackable item（88\*/7\*）时，可以先把 ingredient 5\* / recipe 80\* 加到 component，再用 `create new package` 按钮选中它来创建 88\*/7\* item。由于 menu item component 只应支持 40\*/70\* item，该按钮**已经在 UI 上隐藏**。用户现在改为在 item grid 页创建 88\*/7\* item。

**需求**

1. 从 menu item / 7\* 的 **component 与 customization** section 中，**在代码层面**移除 `Create New Packaged` 功能与流程
2. 目的是消除死代码、避免后续维护踩坑
3. **88\* item component 里的 `Create New Packaged` 保持不变**

> 前端关注点：
> - **menu / 7\* 的屏蔽已经实现**：`ComponentsV2/Action/index.tsx:56` 的 `const showCreatePackagedItem = object_type !== MENU && object_type !== HDR_RECIPE;` 控制着入口按钮 `Create packaged item`（`:557-585`），menu / 7\* 上该按钮已不渲染。
> - **由于 88\* 要保留该功能，`CreatePackage/` 组件、入口按钮、弹窗都必须留着**，`showCreatePackagedItem` 这行判断本身也不能删（删了 menu / 7\* 反而会重新显示）。因此本条实际可清理的范围很小，逐处如下：
>
> | 引用点 | 触发条件 | menu / 7\* 上是否可达 | 处置 |
> | --- | --- | --- | --- |
> | `Action/index.tsx:499` | `editType === "Step2AddPackaging"`，只能由受 `showCreatePackagedItem` 保护的按钮设置 | 不可达 | 88\* 仍需要，保留 |
> | `Action/Edit.tsx:4` | 仅有 `import CreatePackageItem`，全文件无任何使用点 | — | **纯死 import，可删** |
> | `Action/ShowTable.tsx:189` | `handleEdit`（`:127-135`）中 `if (r.isCreateNewItem) setIsPackageModalOpenModalOpen(true)`，**无 item type 判断** | 取决于 7\* 上是否会出现 `isCreateNewItem` 的行 | **需补 type 判断** |
>
> - 可疑点：`ShowTable.tsx:443` 存在 `if (record.isCreateNewItem && object_type === HDR_RECIPE)` 分支，说明代码里预期 7\* 的 component 行上会出现 `isCreateNewItem` —— 需确认是入口按钮隐藏前的遗留，还是仍有路径可达。

**疑问**

- **需知情：本条实际可清理的代码极少，与 ticket 描述的「消除死代码」预期不符。** 因为 88\* item 要保留 `Create New Packaged`，共享的 `CreatePackage/` 组件、入口按钮、弹窗都必须留下；而 menu / 7\* 的屏蔽早已由 `showCreatePackagedItem` 一行判断完成，那行判断本身也不能删。真正能落地的只有两件事：删掉 `Action/Edit.tsx:4` 那个未使用的 import，以及给 `ShowTable.tsx:130` 的 `isCreateNewItem` 分支补上 item type 判断。需要跟 ticket 提出方对齐这个范围预期。
- **`ShowTable.tsx:443` 的 `isCreateNewItem && object_type === HDR_RECIPE` 分支是遗留还是仍可达？** 若 7\* 的 component 行确实不会再出现 `isCreateNewItem`，该分支连同 `handleEdit` 的对应路径都可以清掉；若仍可达，则说明 7\* 上还存在一个没堵住的 `Create New Packaged` 编辑入口。
- 这条属于纯代码清理、**无 UI 变化**，需给出 QA 回归范围（88\* item 的 create new packaged 全流程 + menu item / 7\* 的 component 正常增删编辑）。
