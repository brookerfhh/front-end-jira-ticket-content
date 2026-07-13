# Front-end Jira Tickets — Assigned to Me (Sprint 15)

> 更新时间：2026-07-10
> 查询条件：`assignee = currentUser() AND statusCategory != Done`

| Key | 摘要 (Summary) |
|-----|----------------|
| [MD-18258](https://wonder.atlassian.net/browse/MD-18258) | UI - Add Unit Conversion ea → g for 7* (HDR Recipe) Items |
| [MD-18243](https://wonder.atlassian.net/browse/MD-18243) | [Tech] Frontend dependency security vulnerabilities detected by Snyk across recipe-site-frontend, pcs-frontend, and cookbook-frontend |
| [MD-18151](https://wonder.atlassian.net/browse/MD-18151) | UI - IK support for configuring portion-to-BOM unit conversion for component items |
| [MD-17869](https://wonder.atlassian.net/browse/MD-17869) | UI - Enable Bulk Swap Item from Customization Usages |
| [MD-18084](https://wonder.atlassian.net/browse/MD-18084) | Tech UI - Refactor: rewrite AddComponent & AddGuestPackage with antd Form |

> 说明：MD-18151 / MD-17869 / MD-18084 与 Sprint 14 重复，需求理解直接沿用（见下）。MD-17420 本 sprint 不做，已从清单剔除。

---

## 前端需求拆解

### [MD-18258](https://wonder.atlassian.net/browse/MD-18258)

UI - 为 7* (HDR Recipe) item 增加 ea → g 的 Unit Conversion

> 主 ticket：[MD-18199](https://wonder.atlassian.net/browse/MD-18199)

**背景**：minimal serving portion 以 portion → g（克）配置；7* (HDR Recipe) item 可能 machine eligible 或作为 Wonder Create 组件，其 BOM unit 为 `ea`（each），存在单位不匹配，需要把 recipe 的 `ea` BOM unit 与克制的 serving portion 打通。

- **UI 位置**：在 7* item 详情页的 food science card **上方**新增一张 Unit Conversion card，允许用户设置和管理单位换算（如 `ea → g`）
  - 换算的校验逻辑复用其他 object type item 上已有的 unit conversion 校验
  - 编辑的时候，如果7* item已经 publish过，用户修改 unit conversion 时 `ea → g` 必须保留、不允许删除，报error `Unable to save. Missing required unit conversion: ea → g.`
- **publish的时候需要校验**：`ea → g` 是 7* item 发布的必填换算，缺失时拦截发布并报错 `Unable to publish the version. Missing required unit conversion: ea → g.`
- **Change Log**：unit conversion 变更需记录到 change log

### [MD-18243](https://wonder.atlassian.net/browse/MD-18243)

[Tech] Snyk 扫出的前端依赖安全漏洞（recipe-site-frontend / pcs-frontend / cookbook-frontend）

> 主 ticket（Epic）：[MD-17231](https://wonder.atlassian.net/browse/MD-17231) @2026 Cookbook Technical Excellence

**问题**：Snyk Open Source 扫描在 3 个前端项目里发现多个 npm 依赖漏洞，均「无已知利用」但标为 Partially Fixable。

**漏洞清单**

| # | Snyk ID | 包 | 漏洞 | 受影响项目 |
| --- | --- | --- | --- | --- |
| 1 | SNYK-JS-ESBUILD-17750822 | @esbuild/aix-ppc64 | Resources Downloaded over Insecure Protocol (CWE-426/494) | recipe-site / cookbook / pcs |
| 2 | SNYK-JS-HONO-17356405 | @hono/node-server | Directory Traversal (CWE-22) | pcs / product-catalog-site |
| 3 | SNYK-JS-FASTURI-17675102 | fast-uri | Interpretation Conflict (CWE-436) | 全部 4 个项目 |
| 4 | SNYK-JS-BRACEEXPANSION-17706650 | brace-expansion | Inefficient Algorithmic Complexity (CWE-407) | product-catalog-site / pcs |
| 5 | SNYK-JS-HONO-17356430 | @hono/node-server | Permissive Cross-domain Policy (CWE-942) | product-catalog-site / pcs |

**根因（传递依赖引入）**
- `@esbuild/aix-ppc64` ← `@vitejs/plugin-react@4.5.2`
- `@hono/node-server`（CWE-22 & CWE-942）← `vite-plugin-checker@0.12.0`
- `fast-uri` ← `stylelint-config-recommended-less@3.0.1`
- `brace-expansion` ← `@wonder/web-ui-kit@11.2.2` 及 `vite-plugin-checker@0.12.0`

**修复方向**
1. 升级 `@vitejs/plugin-react` 到带 patched esbuild 的版本
2. 升级 `vite-plugin-checker` 修掉 `@hono/node-server` 的目录穿越 + 跨域策略
3. 升级 `stylelint-config-recommended-less`，或在 `package.json` 加 `resolutions`/`overrides` 把 `fast-uri` 钉到安全版本
4. 升级 `@wonder/web-ui-kit` 或为 `brace-expansion` 加 override
5. 升级后跑 `npm audit` / `yarn audit`（本仓库用 pnpm，跑 `pnpm audit`）验证
6. 重跑 Snyk 扫描确认关闭

**受影响文件**
- `frontend/recipe-site-frontend/package.json`
- `frontend/product-catalog-site-frontend/package.json`
- `cookbook-frontend`（独立 repo）

> 注意：本仓库统一用 **pnpm**，overrides 应写在 `pnpm.overrides` 下；升级后需保证 `pnpm build`（含 vite 类型检查）与 dev server 正常。


### [MD-18151](https://wonder.atlassian.net/browse/MD-18151)

暂时不做，等待确认。
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

> 状态延续：前端已用 mock 写完（Task 1–4），worktree `.claude/worktrees/MD-17869/`，等后端 API。

### [MD-18084](https://wonder.atlassian.net/browse/MD-18084)

Tech UI - 用 antd Form 重写 AddComponent & AddGuestPackage

> 主 ticket（Epic）：[MD-17501](https://wonder.atlassian.net/browse/MD-17501) Supply Chain Catalog Integration

**What**
- 把 `Add Component` 和 `Add Guest Packaging` 弹窗（`ComponentsV2/AddComponent/` 和 `ComponentsV2/AddGuestPackage/`）从 Formik（`@wonder/web-ui-kit` 的 `<Form>`）改为 antd `Form.useForm`
- 抽出共享的 `ComponentPicker` widget，消除两个模块间的重复

**Why**
- 代码库混用两套表单系统（Formik + antd Form），历史技术债
- `@wonder/web-ui-kit` 正在淘汰；`widget/HackDsComponent/AntdInputDsStyle / FormItemDsStyle / ...` 已作为带 ds 视觉的 antd 替代品存在
- 现有代码累积了 bug 和重复：`SearchForm.tsx`(411 行) 与 `SearchFormUI.tsx`(218 行) 近乎重复；`AddGuestPackage` 硬依赖 `AddComponent` 的 Formik hooks；表单状态双源；`useEffect` 依赖 bug
