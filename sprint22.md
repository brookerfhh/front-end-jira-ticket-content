# Front-end Jira Tickets — Sprint 22

## 前端需求拆解

### [MD-18676](https://wonder.atlassian.net/browse/MD-18676)

40* 对齐 5*：补字段 + 可以作为 80* / 88* / benchtop / preparation 的组件

> 主 ticket：[MD-18581](https://wonder.atlassian.net/browse/MD-18581)（Epic: Support 42* Transactions in SD1）；前端对应 UI 子任务 [MD-18678](https://wonder.atlassian.net/browse/MD-18678)

**背景**

SD1 要支持 42* 交易：生产和 IKC 共用的食材，在 80* / 88* BOM 里改用 40*，不再用 5*。所以 40* 要补上 5* 的部分字段，还要能像 ingredient 一样被加到 BOM 和 preparation 里。Kafka 事件、5*/9* ↔ 42* fulfillment mapping、5* → 40* 数据回填由 MD-18581 下的其他票负责。

**需求**

1. 新增字段 Pick Offset（5* / 9* / 40*）
   - 放在 **Production** card 里展示和编辑，输入形式 `xx days xxx hours`（两个输入框），合计必须 > 0
   - 新建 item 默认 1 天（24 小时），不能清空
2. 40* 详情页新增 card
   - **Cooking and Preparations**：Yield、Preparations，字段和交互同 5*
   - **Production**：字段同 5* Production card，去掉 Product Coverage Group，加上 Pick Offset；Production Type 沿用现有 5*/40* 逻辑
   - **Nutrition & Allergens**：保留 40* 现有营养信息，只加两个字段
     - Ingredient Statement：保存时必填（RA 已确认）；tip `It is for regulation labeling`。自动生成、跟随 Wonder App Ingredients 更新都是后端逻辑，前端只做必填校验
     - Bioengineered?：只有 Yes / No，选填；tip `It is for regulation labeling`
   - 5* 的 Cooking and Preparations / Production 组件改成通过 props 传数据，40* 和 5* 共用一套
   - 新建 40* 弹窗里的 Item Type 分类树按 FOOD / NON_FOOD 区分，40* 走食品树
3. Change History：Pick Offset、Cooking and Preparations、Ingredient Statement / Bioengineered，以及 40* 上新开放编辑的 Production 字段都要能对比
4. 40* 作为 80* / 88* / benchtop 的 BOM 组件（MD-18692）
   - 88* 的 Add Component 弹窗可以搜到、选中 40*
   - 80* recipe 和 benchtop 的 edit recipe component 页，搜索和筛选都加上 40*
   - 40* 按 ingredient 处理：加组件第二步不要求填 Lead Time
5. Preparation 支持 40*
   - Create Preparation 弹窗的组件搜索可以加 40*；下拉显示 item number，40* 显示 Ingredient 标签
   - edit recipe component 页里原来只对 ingredient 生效的 preparation 逻辑扩到 40*：加 40* 组件时也会检查 preparation 是否已存在
6. 不加 missing-info 校验，40* 不新增 publish 阻断，5*/9* 现有校验不变
   
> 待确认：在 80* recipe 里加 5*、选的 preparation 全是 PROCESS 类型时，会多出一个 Lead Time 输入框（提前几天处理）。现在这个输入框只对 5* 出现，40* 选了 PROCESS preparation 也不会出现（加组件弹窗、编辑弹窗、列表 LEAD TIME 列 3 处）。另外 preparation recipe 里 5* 行隐藏 Edit 按钮，40* 行目前不隐藏。这两点 40* 要不要跟 5* 一样，先不动。

### [MD-18628](https://wonder.atlassian.net/browse/MD-18628)

Template 的 Customization 限制 Display Style 和 Customization Type 选项

**背景**

Influencer Create 流程生成的 item 会成为 preset，preset 只支持 Display Style = Featured。如果 template 的 Customization 允许 In-Drawer 或某些 Customization Type，用这个 template 生成 preset 时就会产生非法数据。

**需求**

1. 只对 wc template item 生效，Create Customization 和 Edit Customization 两个弹窗都要改
2. Display Style：置灰 `In-Drawer`，只能选 `Featured`
3. Customization Type：置灰 `Optional Subtraction`、`On the Side`、`Extra Request`、`Dish Preference`

### [MD-18627](https://wonder.atlassian.net/browse/MD-18627)

Preset 营养的三个字段从父 BYO 自动继承

> 主 ticket：[MD-17762](https://wonder.atlassian.net/browse/MD-17762)（Epic: Wonder Create Integration）；前端对应 UI 子任务 [MD-18722](https://wonder.atlassian.net/browse/MD-18722)

**背景**

一个父 BYO menu item 经常挂 10 个以上 preset。preset 营养里有三个字段应该始终和父 BYO 一致，现在只能逐个 preset 手动改。

**需求**

1. **所有** preset 的 Edit Nutrition 置灰三个字段：`Serving Per Dish`、`Discrete Unit?`、`Consumer Serving Size Display Name`
2. 这三个字段取父 BYO 的值，不能在 preset 上单独编辑
3. 字段置灰后保存时 payload 里仍要带上这三个值

### [MD-18638](https://wonder.atlassian.net/browse/MD-18638)

去掉vendor SKU link 时的一个的校验

> 前端对应 UI 子任务 [MD-18724](https://wonder.atlassian.net/browse/MD-18724)

**背景**

Finance 不再要求这条校验。

**需求**

1. 5*/9* 单个编辑 vendor SKU mapping：去掉提示 `Unable to add vendor SKU  The vendor SKU has linked with WSKU {WSKU item number} in SCC. Please remove the linkage from SCC first.`
   - 前端位置：`src/page/item/detail/pages/ERP/pageComponent/SupplierItems/component/LinkVendorItem.tsx:466`
2. 通过模板文件批量编辑 vendor SKU mapping：去掉 `Vendor SKU has linked with WSKU item in SCC: xxx`
   - 前端位置：`src/page/vendorV2/skuMappings/components/BulkEdit.tsx:172`
3. 后端去掉校验后，前端这两处的分支和展示代码一起删掉，拉 api 后清理相关引用


### [MD-18313](https://wonder.atlassian.net/browse/MD-18313)

[Tech] 删除 5 个无流量的 recipe-service-v2 接口和 transfer BOM 页面

> 主 ticket：[MD-17231](https://wonder.atlassian.net/browse/MD-17231)（Epic: @2026 Cookbook Technical Excellence）；前端对应 UI 子任务 [MD-18726](https://wonder.atlassian.net/browse/MD-18726)

**What**

- 后端删除 5 个无流量接口，其中一个唯一的调用方是 recipe-site 的 transfer-BOM / hidden BOM 页面
- **前端前置工作**：删除 transfer-BOM / hidden BOM 页面
  - 调用点：`src/page/hiddenPage/management/page.ts:26` → `InventoryItemAJAXWebService.getConsumableItems`
  - 页面删除后，连同路由、菜单入口、相关组件一起清理；后端删接口后拉 api 覆盖

**Why**

Datadog APM 显示这些接口近 30 天生产流量为 0，每个都有承载真实流量的同类接口，留着只会增加维护成本。

**注意**

- 删页面前确认 `hiddenPage/management` 里除了 consumable items 以外有没有别的功能还在用（同目录还有 `EditFlag` / `EditItemStockFlag` 等 action），避免误删整页
- 前端先上，后端才能删 BFF

### [MD-18684](https://wonder.atlassian.net/browse/MD-18684)

[Tech] 修复前端 Snyk 漏洞（2026-10-10 扫描，12 个）

> 主 ticket：[MD-17251](https://wonder.atlassian.net/browse/MD-17251)（Epic: @2026 Cookbook Devops Action）

**What**

1. 9 个已经修好，只是没进 master：MD-18637（commit `9981dc14d97`）把 `stylelint`、`stylelint-config-standard`、`stylelint-config-recommended-less`、`vite-plugin-checker`、`vite-plugin-stylelint` 从 `dependencies` 挪到 `devDependencies`，只合了 develop。Snyk 扫的是 master
   - 动作：把 MD-18637 合进 master，不用写新代码
   - 涵盖：SNYK-JS-PROXYADDR-19812342（Critical）、SNYK-JS-HONO-19499068 / -19499091、SNYK-JS-URIJS-19963963、SNYK-JS-ZOD-20510278、SNYK-JS-MODELCONTEXTPROTOCOLSDK-20579853、SNYK-JS-FASTURI-19502739 / -19502854、SNYK-JS-POSTCSSSELECTORPARSER-20510276
2. 还需要处理的 3 个
   - SNYK-JS-BRACES-19963945、SNYK-JS-BRACEEXPANSION-20244948：经 `@wonder/web-ui-kit` 11.2.2（运行时依赖）引入，移到 devDependencies 解决不了，要加 pnpm override 或升级 web-ui-kit
   - SNYK-JS-REACTROUTER-18313144（Open Redirect）：只有 react-router 7.18.0 修了，core-fe 2.4.10 基于 v6 API。调研已在 `openspec/changes/upgrade-core-fe-react-router-v7`（目标 core-fe 2.5.10 + override 7.18.2；5 处 `useBlocker` 有回归风险）。要决定这个 sprint 做还是继续延后

**Why**

master 上还挂着 1 个 Critical（User Impersonation）。

**范围**：recipe-site-frontend、product-catalog-site-frontend、pcs-frontend、cookbook-frontend

### [MD-18672](https://wonder.atlassian.net/browse/MD-18672)

KDS Portion Unit 下拉加 `Click` 并按字母排序

> 主 ticket：[MD-17762](https://wonder.atlassian.net/browse/MD-17762)（Epic: Wonder Create Integration）

**需求**

1. KDS Portion Unit 下拉新增选项 `Click`
2. 下拉选项按字母 A–Z 排序，N/A 也是按顺序排
