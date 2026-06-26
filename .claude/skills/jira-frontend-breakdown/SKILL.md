---
name: jira-frontend-breakdown
description: Use when 需要把 sprint 文档（如 sprint14.md）里的 Jira 工单整理成前端需求点并写入该文档。触发词：整理前端需求、拆解工单、列前端要做什么、前端需求拆解。
---

# Jira 前端需求拆解

## Overview

把 sprint 文档工单表格里的 Jira ticket，逐个拆解成「前端要做什么」的需求点，按统一格式写回同一个文档。

## 流程

1. 读取 sprint 文档，定位工单表格，从第一个开始逐个处理（除非用户指定某个）。
2. 用 Jira MCP（`getJiraIssue`，cloudId `wonder.atlassian.net`，`responseContentFormat: markdown`）读取工单。
3. 若工单是 Sub-task 且 description 为空，先取 `parent` 字段，再读主 ticket 拿完整描述。
4. 按下方格式整理前端需求点，写入 sprint 文档的「前端需求拆解」一节。

## 输出格式

```markdown
### [KEY](Jira 链接)

<工单 summary（功能描述）>

> 主 ticket：[KEY](链接)｜[Figma 设计稿](链接)

- <前端需求点>
- ...
```

- 标题用工单 Key 链接；下一行写 summary；再用引用行放主 ticket、Figma 等参考链接。
- 引用主 ticket 时统一称「主 ticket」，不要叫「父工单」。
- 需求点用无序列表，全程中文。

## 内容范围（只列前端）

不写后端对接的内容，常见的后端项包括：

- 通知 HDR / 其他下游系统（后端对接）。
- Copy new item / Copy new version / Move to variant / Move to draft（后端接口处理）。

## 书写约定

- 同一字段的逻辑写在**同一行**，输入形式 + 校验规则合并，不要拆成多行。
  - 例：`Chilling Target Temperature（冷却目标温度）：整数输入（支持负数），显示单位 °F（如 45 °F）`
- 跨字段的通用约定（是否必填、可否空值提交）单独成行。
- Change Log 相关写「Change Log 也要加上 XXX 字段」，不要写「纳入 Change Log 展示」。

## 示例

### [MD-18176](https://wonder.atlassian.net/browse/MD-18176)

UI - 在 7* item 的 prep procedure card 中，为 Cook & Chill 增加「烹饪时长」和「冷却温度」字段

> 主 ticket：[MD-18172](https://wonder.atlassian.net/browse/MD-18172)｜[Figma 设计稿](https://www.figma.com/design/o6ZcIP0rGkCILw3MU23KCX/Inventory-Manager?node-id=3945-798)

- 在 prep procedure card 中新增两个可选字段，均为非必填，未填写也可正常提交
- Cook Time（烹饪时长）：Hr / Min / Sec 三段输入，时分秒数值合理
- Chilling Target Temperature（冷却目标温度）：整数输入（支持负数），显示单位 `°F`（如 `45 °F`）
- Change Log 也要加上这两个新字段
