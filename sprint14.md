# Front-end Jira Tickets — Assigned to Me

> 更新时间：2026-06-25
> 查询条件：`assignee = currentUser() AND statusCategory != Done`

| Key | 摘要 (Summary) |
|-----|----------------|
| [MD-18176](https://wonder.atlassian.net/browse/MD-18176) | UI - Add cook time and chilling temperature fields to 7* item prep procedure card for Cook & Chill bulk prep |
| [MD-18149](https://wonder.atlassian.net/browse/MD-18149) | UI - IK Support-Component machine eligibility warning and IK step ordering constraint |
| [MD-18150](https://wonder.atlassian.net/browse/MD-18150) | UI - IK Support-Side Component in Specific Dish Type |
| [MD-18175](https://wonder.atlassian.net/browse/MD-18175) | UI - Task Level Cooking Group — KDS routing configuration |
| [MD-18170](https://wonder.atlassian.net/browse/MD-18170) | Auto-select default IK plating rule when step marked as IK eligible |
| [MD-18174](https://wonder.atlassian.net/browse/MD-18174) | UI - Configure IK Portion Conversion at Component Level |
| [MD-18109](https://wonder.atlassian.net/browse/MD-18109) | Remove "Thaw" tag/suffix from all item views |
| [MD-18084](https://wonder.atlassian.net/browse/MD-18084) | Tech UI - Refactor: rewrite AddComponent & AddGuestPackage with antd Form |
| [MD-18151](https://wonder.atlassian.net/browse/MD-18151) | UI - IK support for configuring portion-to-BOM unit conversion for component items |
| [MD-17869](https://wonder.atlassian.net/browse/MD-17869) | UI - Enable Bulk Swap Item from Customization Usages |

---

## 前端需求拆解

### [MD-18176](https://wonder.atlassian.net/browse/MD-18176)

UI - 在 7* item 的 prep procedure card 中，为 Cook & Chill 增加「烹饪时长」和「冷却温度」字段

> 主 ticket：[MD-18172](https://wonder.atlassian.net/browse/MD-18172)

- 在 hdr recipe item 的 prep procedure card 中新增两个可选字段，均为非必填，未填写也可正常提交
- Cook Time（烹饪时长）：Hr / Min / Sec 三段输入，时分秒数值合理
- Chilling Target Temperature（冷却目标温度）：整数输入（支持负数），显示单位 `°F`（如 `45 °F`）
- Change Log 也要加上这两个新字段

### [MD-18149](https://wonder.atlassian.net/browse/MD-18149)

UI - IK 支持：组件 Machine Eligible 警告 + IK step 排序约束

> 主 ticket：[MD-18130](https://wonder.atlassian.net/browse/MD-18130)

**校验 1 — Machine Eligible 组件警告（WARNING，不阻塞保存）**

- 保存 line build 时，对每个 `IK Eligible=true` 的 step，检查其下所有 substep 映射的 component / customization option 是否都带「Machine Eligible」attribute（substep 映射的是 customization option 时，至少有 1 个 item 带 Machine Eligible 即可）
- 若存在未 Machine Eligible 的 Component：弹出确认弹窗，标题 `Are you sure`，正文提示并逐条列出 `Task X - Step Y：{组件编号} {组件名}；{option 名}…`，按钮为 Cancel / Save（不阻塞保存）
- 同样的提示在 line build 详情页的 bad data message 区域展示（逐条列出对应 task-step-组件）

**校验 2 — IK Eligible step 排序约束（ERROR，阻塞保存）**

- 保存含 `IK Eligible=true` step 的 line build 时校验：所有 IK Eligible step 必须排在 task 最前面、且连续（中间不能夹非 IK step）
- 没有映射 component / option 的纯指令 step（instruction-only）不允许出现在 IK Eligible step 之前或之间
- 校验不通过时阻塞保存，弹出错误提示并列出需要移动的 step，如 `Task1-Step1, Step4, Step5`、`Task2-Step1, Step3`

### [MD-18150](https://wonder.atlassian.net/browse/MD-18150)

UI - IK 支持：特定 Dish Type 下的 Side Component（在 task 上新增 IK Dish Type）

> 主 ticket：[MD-18115](https://wonder.atlassian.net/browse/MD-18115)

- 在 line build 的编辑视图和详情视图中，为 task 新增「IK Dish Type」字段，可选
- 选项与 attribute 中的一致：48oz Bowl、32oz Bowl、30oz Oval、Metal Bowl、Bellies Bowl、8oz Cup
- 不论 task 的 mapping option 是否为空、也不论 task 数量多少（即使只有一个 task），都允许 CE 为 task 指定 IK Dish Type；task 级别优先于 menu item 级别
- Change Log 也要加上 task 的 IK Dish Type 字段

copy linbebuild？
### [MD-18175](https://wonder.atlassian.net/browse/MD-18175)

UI - Task 级 Cooking Group（KDS 路由配置）

> 主 ticket：[MD-18165](https://wonder.atlassian.net/browse/MD-18165)

- 在 task 层级、「IK Dish Type」字段旁边新增「Cooking Group」字段，选项与 attribute 中 Cooking Group 的值一致
- 每个 task 支持配置多个 Cooking Group（多选）
- Change Log 也要加上 task 的 Cooking Group 字段，UI 的展示要商量一下
- ![alt text](image.png)

### [MD-18170](https://wonder.atlassian.net/browse/MD-18170)

Auto-select：step 勾选 IK eligible 时自动带出 default IK plating rule

- IK eligible checkbox 勾选 从 null → true：把该 step 下所有当前 IK plating rule 为 null 的 substep 自动选为「default」（不论 substep 映射的 item / option 是什么）
- IK eligible checkbox 取消勾选 从 true → null：清空该 step 下所有 substep 的 IK plating rule（default 和 custom 选择都移除）
- 在 IK eligible=true 的 step 下新增 substep 时，自动把 IK plating rule 选为 default；否则不自动选

12
### [MD-18174](https://wonder.atlassian.net/browse/MD-18174)

UI - 在 Component 级配置 IK Portion Conversion（份量换算）

> 主 ticket：[MD-18167](https://wonder.atlassian.net/browse/MD-18167)

- 在「Usages」card 新增 IK Portion Conversion 区块，点击「Add」打开配置弹窗（交互参考 KDS portion 的配置弹窗），仅需配置一个 `x portion = x g` 的换算，并显示提示 `The IK portion conversion is required for IK Pod.` success message？
- IK Portion Conversion 是 component 级别的：active final 或 scheduled version 任一改动都会同时作用于两个 version（active 的改动也同步到 draft）；当前为 final 或 scheduled version 时，才在「Add」按钮处以 tip 展示 `The change will be implied in both active and future versions.`
- 如果当前是draft version 中并且该item 存在已经publish 过的version 的时候，置灰按钮不可编辑，并在「Add」按钮处以 tip 展示 `Please update it in active version.`不包含过期
- 当 machine eligible=true 时，发布 component 需校验 portion → g 换算必填，缺失则拦截发布并显示 error 提示
- 在 edit attribute 弹窗保存时，遍历属性列表，只要其中有一项 attribute name = 「machine eligible」且值为 yes，就校验当前 item 是否已配置 IK Portion（portion → g 换算）：已配置则直接保存通过、不弹窗；未配置则弹出「配置 IK Portion Conversion」弹窗，用户配置保存后返回 edit attribute 弹窗，点取消同样返回 edit attribute 弹窗
- attribute 值设为 no（或未勾选）时不触发上述校验
- 「配置 IK Portion Conversion」弹窗需做成可复用组件（「Usages」card 常规入口与 edit attribute 联动流程共用）
- Move to variant 场景：variant 中 IK Portion Conversion 区块置灰禁止编辑，提示 `Please maintain it in normal version.`
- Change Log 也要加上 IK Portion Conversion 字段

> ⚠️ 6/30 更新（Bonnie）：功能更名为 **Minimal Serving Portion Conversion**；区块对**所有 object type**展示；提示语改为 `The minimal serving portion conversion is required for IK Pod and Wonder Create Item.`；发布/edit-attribute 校验触发条件增加 **wonder create=eligible**；新增两个权限点（common + benchtop）；输入框限正整数。
> 逻辑调整：draft 且有未过期已发布版本时 **Add 允许、只禁用 edit/delete**。

#### 测试用例（QA checklist）

**1. Usages 卡片 · 展示**
- [ ] 所有 object type 详情页都能看到「Minimal Serving Portion Conversion」区块（ingredient / recipe / by-product / packaged / HDR recipe / HDR consumable / benchtop / WSKU）
- [ ] 标题旁 info 图标 hover：`The minimal serving portion conversion is required for IK Pod and Wonder Create Item.`
- [ ] 未配置显示 `None`；已配置显示 `x portion = x g`

**2. 新建 / 编辑弹窗**
- [ ] 布局 `[Portion] portion = [BOM Usage] g`，单位与输入框对齐
- [ ] Portion / BOM Usage 只能正整数（不能小数 / 负数 / 0 / 字母），必填校验
- [ ] 保存成功 toast + 区块刷新
- [ ] 编辑预填当前值、改后生效
- [ ] 删除有二次确认弹窗，确认后变 None

**3. 版本管控（核心）**
- [ ] active(FINAL)/scheduled：可增删改；hover 按钮显示 `The change will be implied in both active and future versions.`
- [ ] draft 且有未过期已发布版本：**Add 可点**；编辑/删除置灰，hover `Please update it in active version.`
- [ ] draft 且只有过期已发布 / 从未发布：增删改都可用
- [ ] variant：Add + 编辑 + 删除全置灰，hover `Please maintain it in normal version.`
- [ ] 过期版本：只读无按钮
- [ ] 无权限：只读无按钮

**4. 权限（两个权限点）**
- [ ] common 权限 → 普通 item 可编辑
- [ ] benchtop 专属权限 → benchtop item 可编辑；无则只读
- [ ] 两权限独立（有 common 无 benchtop 时 benchtop item 只读）

**5. Edit Attribute 联动**
- [ ] Machine Eligible = Yes 且未配置 → 保存属性时弹出配置弹窗（不直接保存）
- [ ] Wonder Create = Eligible 同样触发
- [ ] 已配置 → 直接保存不弹窗；Machine Eligible=No/未选 → 不触发
- [ ] 弹窗标题下红字：`Minimal serving portion conversion is required for machine eligible/wonder create component.`
- [ ] 配置保存 / 取消 → 都回到 edit attribute 弹窗
- [ ] 无 configure 权限 → 弹窗 Save 置灰，hover `Missing configure minimal serving portion permission`
- [ ] 多条属性时只要有一条 Yes 就触发

**6. 发布校验（后端）**
- [ ] Machine Eligible=true 或 Wonder Create=eligible 且未配置 → 拦截发布，报错 `Unable to publish the version. Minimal serving portion conversion is required for machine eligible/wonder create component. Please configure it before trying again.`
- [ ] 已配置 → 正常发布

**7. Change Log**
- [ ] 增/改/删后 Change History 出现「Minimal Serving Portion Conversion」卡片
- [ ] Only show changes 开启：有变化才显示，换算行黄色高亮
- [ ] 左右版本对比各自显示 `x portion = x g`

**8. 版本继承 / 复制（后端行为，前端验证展示）**
- [ ] 复制新版本 / 新 item：换算一并复制
- [ ] Move to draft：draft 继承 active 换算
- [ ] Move to variant：不复制到 variant，且区块置灰
- [ ] active/scheduled 改动 → 两个版本都生效并同步 draft

> 重点回归：#3 版本管控（draft 的 Add 可用 / edit-delete 锁）、#5 edit-attribute 联动、#7 change log 高亮。


### [MD-18109](https://wonder.atlassian.net/browse/MD-18109)

移除所有 item 视图中的「Thaw」tag / 后缀

- 在所有 40* item 视图（首页、BOM 视图、item 详情页、搜索结果等）移除 / 隐藏「Thaw」tag / icon
- 创建 thawed 状态的 40* item 时，不再追加「Thaw」后缀
- 清除所有现存 thawed 状态 40* item 名称上的「Thaw」后缀

### [MD-18151](https://wonder.atlassian.net/browse/MD-18151)（暂时不做）

UI - IK 支持：为 component item 配置 portion → BOM unit 换算校验

> 主 ticket：[MD-18122](https://wonder.atlassian.net/browse/MD-18122)

- 向 menu item 的 component / customization 添加组件时，以 BOM unit 存储 usage；若该 component 已配置 IK portion conversion，则在 UI 显示换算后的 portion 数量作为参考提示，否则不显示
- 以下场景均需校验 usage qty 能否换算为整数 portion，不能则弹「Are you sure」警告弹窗（不阻塞，按钮 Cancel / Save；发布场景为 Cancel / Publish），并逐条列出对应的 menu item / component / customization：
  - 更新 portion unit conversion 时（忽略 expired version、variant、draft version）
  - 在 menu item component / customization 中更新 usage qty 时（component 与 customization 两种文案不同：component 列 item number；customization 列 `option name + item number + (IK portion)`）
  - 发布 menu item 时（其 component / customization item `machine eligible=true`，按钮为 Cancel / Publish）
  - Bulk edit component / customization usage（object type = menu item）时（文案含 `1 Portion = xx {BOM unit}`）
  - Bulk swap 替换组件 usage（object type = menu item）时（校验替换后组件的 usage，文案含 `1 Portion = xx {BOM unit}`）

### [MD-17869](https://wonder.atlassian.net/browse/MD-17869)

UI - 在 Customization Usages 支持 Bulk Swap Item

> 主 ticket：[MD-17832](https://wonder.atlassian.net/browse/MD-17832)

- 在 Customization Usages 新增「Bulk Swap」功能，参考 BOM usage 的 Swap Component 实现（`page/item/detail/components/ItemUsage/BulkEditUsage/BulkSwapItem.tsx`）
- 区别于 BOM usage：需支持数据聚合——同一 menu item 有多个 customization usage 时，按 menu item number 聚合展示，参考 Customization Usages 的 Bulk Edit（`page/item/detail/components/ItemUsage/BulkEditUsage/CustomizationUsage/CustomizationUsageBulkEdit.tsx`）
- change history 页增加 Customization Usages 的对比（comparison）
- 记录 change log



按主 item 类型的替换规则：

| 主 item 类型 | Customization Usages 行为 |
| --- | --- |
| 40 / 7\* item | 显示全部类型：mandatory choice / optional addition / extra request / on the side / Optional Subtraction / Dish Preference；能替换的也是只能是 40\*/70\*；新替换项 BOM unit 须与原项一致，否则报错 `Unable to bulk edit {tab name}. The BOM unit of replacement item should be the same as the original item ({BOM Unit}) is missing, please select another one.`； |
| recipe 80\* / 88\* / 5\* item | 仅显示：on the side / Optional Subtraction / Dish Preference；替换项可为 80\*/88\*/5\*/40\*/70\*；|
| 9\* item | 显示可映射非食品项的 customization 类型（Optional Subtraction 除外）；替换项须为 90\*（子类型 = guest packaged 或 internal packaged）； |



自动化测试分享：
Recon-before-write:先侦察真实页面拿到准确锚点。
闭环生成:recon 真实页面 → 出可审的用例(tests.md)→ 生成脚本 → 跑验证
"不留垃圾数据"策略:有删除→幂等 create→delete;只创建→开表单+校验+取消;只读→只验加载。可反复跑、不污染 QA。


Agent skill(regression-add-page):把整套流程固化进 repo,一句话生成。
知识库(PAGES.md):从源码沉淀每页元素/能力/隐藏规则 → QA 不读代码也能生成。
免 MCP recon:用诊断脚本侦察,零额外安装。
认证:SessionId 注入 / storageState 绕登录,跨 app 通用。

123
123