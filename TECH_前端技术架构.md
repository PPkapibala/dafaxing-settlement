# 大发行打款分账系统 — 前端技术架构

*更新时间：2026-04-20（基于 PRD V2.0 同步补全）*

---

## 一、项目类型

企业管理后台（内部财务人员使用，大量表格/表单/审批流）

---

## 二、框架选型

| 项目 | 选型 |
|---|---|
| 框架 | UmiJS 4 + Ant Design Pro 6 |
| 语言 | TypeScript 5 |
| UI 组件库 | Ant Design 5 + @ant-design/pro-components |
| 图表 | @ant-design/charts |
| 状态管理 | Umi model（内置）|
| HTTP 请求 | @umijs/plugin-request（基于 axios 封装） |
| 构建工具 | Umi 内置 mfsu + Vite 模式 |

**选型原因**：内置权限管理、菜单配置、多角色路由，ProTable/ProForm 开箱即用，适合复杂审批流和财务报表场景。

---

## 三、角色与权限（RBAC）

系统有 4 个角色，路由与操作按角色隔离，通过 UmiJS `access` 插件实现。

| 角色 | 标识 | 可访问模块 |
|---|---|---|
| 系统管理员 | `admin` | 游戏字典、渠道字典、游戏-渠道关系、用户管理、审计日志 |
| 业务负责人 | `business` | 三方关系管理、数据看板（业务视图）、分成规则（只读） |
| 财务 | `finance` | 合作方主体、银行账户、分成规则、费用录入、收入池、打款申请、虚拟账户、数据看板（财务视图）、通知 |
| 资金组 | `capital` | 资金池余额查看、排队申请单查看、通知（资金相关） |

> `access.ts` 中按 `role` 字段定义各页面的 `canAccess` 条件；菜单通过 `menuData` 中 `access` 字段动态过滤。

---

## 四、路由结构

```
/                          → 重定向至 /dashboard
/login                     → 登录页（飞书 SSO 或账号密码）
/dashboard                 → 数据看板（财务/业务两种视图，按角色渲染不同卡片）
/basic-config
  /games                   → 游戏字典管理（admin）
  /channels                → 渠道字典管理（admin）
  /game-channel-rels       → 游戏-渠道关联（admin）
  /partner-rels            → 三方关系管理（business）
/partner
  /list                    → 合作方主体列表（finance）
  /detail/:id              → 合作方主体详情 + 银行账户（finance）
/rule
  /list                    → 分成规则列表（finance）
  /create                  → 创建规则（finance）
  /detail/:id              → 规则详情（finance/business 只读）
/settlement
  /income-pool             → 收入池查询（finance）
  /expense-bill            → 费用录入待办（finance）
  /records                 → 结算记录（finance）
  /virtual-accounts        → 虚拟账户（finance）
/payment
  /list                    → 打款申请单列表（finance）
  /create                  → 发起打款申请（finance）
  /detail/:id              → 申请单详情（finance）
  /queue                   → 排队中申请单（finance/capital）
/notification
  /list                    → 系统消息中心（all roles）
/audit
  /logs                    → 操作审计日志（admin/finance）
```

---

## 五、各模块组件方案（全量）

### 5.1 基础配置（系统管理员）

| 功能 | 组件 | 说明 |
|---|---|---|
| 游戏字典列表 | ProTable | 支持关键词搜索、状态筛选 |
| 游戏新增/编辑 | ProForm Modal | 游戏编码从女娲平台同步，只读；名称/状态可编辑 |
| 渠道字典列表 | ProTable | 同上，渠道编码来自渠道管理系统 |
| 渠道新增/编辑 | ProForm Modal | — |
| 游戏-渠道关联 | ProTable + 关联选择器 | 新增时联动选择游戏+渠道 |

### 5.2 三方关系管理（业务负责人）

| 功能 | 组件 | 说明 |
|---|---|---|
| 三方关系列表 | ProTable | 按游戏/渠道/合作方筛选 |
| 新增三方关系 | ProForm Drawer | 选择游戏-渠道关系（已有关联）+ 选择合作方主体 + 填写业务负责人 |
| 状态切换 | Switch + Popconfirm | 启用/停用二次确认 |

### 5.3 合作方主体（财务）

| 功能 | 组件 | 说明 |
|---|---|---|
| 合作方列表 | ProTable | 关键词/状态筛选，分页 |
| 创建合作方 | ProForm | 公司名称、信用代码、法人、联系人 |
| 合作方详情 | Descriptions + Tabs | Tab1：基本信息；Tab2：银行账户；Tab3：关联规则 |
| 银行账户新增/编辑 | ProForm Modal | 账号展示时脱敏（中间4位替换 `****`） |
| 操作日志 | Timeline | 在详情页 Tab4 展示账户变更历史 |

### 5.4 分成规则（财务/业务负责人）

| 功能 | 组件 | 说明 |
|---|---|---|
| 规则列表 | ProTable | 按合作方/游戏/渠道/状态筛选；状态用 Badge 色标 |
| 创建规则 | StepsForm | Step1: 基本信息（合作方+游戏+渠道+时间+周期）；Step2: 成本费用项（EditableProTable）；Step3: 预留比例+备注 |
| 成本/费用子表 | EditableProTable | 支持多行增删；成本项展示浑天仪标识号字段 |
| 规则详情 | Descriptions（只读） | 审批通过后所有字段 disabled |
| 审批状态展示 | Steps（horizontal） | 展示 BPM 当前节点 |
| 撤回审批 | Popconfirm + Button | 仅"审批中"状态可见 |

### 5.5 收入池（财务，只读）

| 功能 | 组件 | 说明 |
|---|---|---|
| 收入池查询 | ProTable | 按合作方/游戏/渠道/日期范围筛选；金额展示两位小数，千分位 |
| 补录入口（仅管理员） | Button → ProForm Modal | 触发手动同步指定日期数据 |

### 5.6 费用录入（财务）

| 功能 | 组件 | 说明 |
|---|---|---|
| 待办列表 | ProList | 按状态分类（待录入/已录入/已超期/已结算）；待录入高亮展示截止时间 |
| 费用账单新建 | ProForm | 选择规则+费用项、实际产生账期、账单总额 |
| 账期拆分明细 | EditableProTable | 各账期拆分金额；底部展示"合计 vs 账单总额"校验提示 |
| 提交锁定 | Popconfirm | "提交后不可修改，确认？" |

### 5.7 结算记录（财务）

| 功能 | 组件 | 说明 |
|---|---|---|
| 结算记录列表 | ProTable | 按合作方/账期筛选 |
| 结算详情 | Descriptions + 明细表格 | 展示收入、成本扣减、费用扣减、分账金额、预留金额、转入待结算 |

### 5.8 虚拟账户（财务）

| 功能 | 组件 | 说明 |
|---|---|---|
| 虚拟账户列表 | ProTable | 按合作方筛选 |
| 账户详情 | Statistic Card 组 | 展示待结算/冻结/可打款/已结算/预留资金池余额 |
| 预留资金池明细 | ProTable 展开行 | 各期预留金额记录 |
| 预留比例修改 | InputNumber + Alert | Alert 提示"仅对未出账账期生效" |

### 5.9 打款申请（财务）

| 功能 | 组件 | 说明 |
|---|---|---|
| 申请单列表 | ProTable | 按审批状态/执行状态/日期筛选；双状态 Badge 展示 |
| 发起打款 | ProForm | 选游戏+渠道（联动自动填合作方及账户快照）；金额校验 ≤ 可打款金额 |
| 申请单详情 | Descriptions + Timeline | 执行状态时间线；展示账户快照（脱敏）；BPM 审批节点 |
| 重新打款按钮 | Button（failed/returned_released 可见） | 带 Popconfirm，跳转发起打款页并预填信息 |
| 退票确认解冻 | Button + Popconfirm | `returned_pending` 状态财务确认操作 |

### 5.10 排队申请单（财务/资金组）

| 功能 | 组件 | 说明 |
|---|---|---|
| 排队列表 | ProTable | 按审批通过时间升序展示；展示排队序号、金额、等待时长 |
| 资金池余额 | Statistic Card | 页面顶部展示当前可用余额（调用银企直联查询） |

### 5.11 数据看板

| 功能 | 组件 | 角色 |
|---|---|---|
| 全平台待结算汇总 | Statistic + Card | 财务 |
| 冻结金额汇总 | Statistic + Card | 财务 |
| 今日待打款金额 | Statistic + Card | 财务 |
| 排队中总额 | Statistic + Card | 财务/资金组 |
| 打款成功率趋势图 | Line Chart（@ant-design/charts） | 财务 |
| 分账明细汇总查询 | ProTable | 财务（按主体/游戏/渠道/日期） |
| 业务视图：所负责合作方结算/打款情况 | ProTable | 业务负责人 |
| 业务视图：关联规则配置情况 | ProList | 业务负责人 |

### 5.12 通知中心（所有角色）

| 功能 | 组件 | 说明 |
|---|---|---|
| 顶部 Bell 图标未读数 | Badge + IconButton | 全局 Layout Header 常驻 |
| 通知下拉预览 | Dropdown + List | 最近 5 条，点击跳转对应业务页 |
| 系统消息中心列表 | ProTable | 按已读/未读、通知类型筛选；支持全部标为已读 |

### 5.13 操作审计日志（管理员/财务）

| 功能 | 组件 | 说明 |
|---|---|---|
| 审计日志列表 | ProTable | 按对象类型/操作类型/操作人/日期筛选；只读，无删除入口 |
| 日志详情 | Modal + JSON 展示 | 展示操作前后字段 diff |

---

## 六、组件使用原则

- 表格、表单、弹窗、日期选择器等通用组件**必须使用现有库**，不手写
- 同一项目不混用多套组件库
- 组件库无法满足需求时，先搜索开源方案（GitHub Stars > 1000，近6个月有更新），再考虑手写
- 金额字段统一用 `toLocaleString('zh-CN', {minimumFractionDigits:2})` 格式化，不使用第三方数字库
- 敏感字段（银行账号）前端只展示脱敏值，不存储明文

---

## 七、接口层规范

```typescript
// src/utils/request.ts（基于 umi-request）
// 统一响应结构：{ code: number, message: string, data: T }
// code !== 0 时 toast 错误信息并 reject

// 分页请求参数统一：page（从1起）, page_size（默认20）
// 分页响应统一：{ list: T[], total: number, page: number, page_size: number }

// ProTable 的 request 属性统一封装为：
// async (params) => { const res = await api.xxx(params); return { data: res.list, total: res.total, success: true } }
```

**错误处理：**
| 场景 | 处理方式 |
|---|---|
| code=1003（状态非法）| message.error + 刷新当前页 |
| code=2001（金额超限）| Form.setFields 标红金额字段 |
| HTTP 401 | 跳转 /login |
| HTTP 500 | message.error('服务异常，请稍后重试') |

---

## 八、Mock 服务说明（Demo 阶段）

Demo 阶段外部系统（浑天仪、BPM、银企直联）通过后端内置 Mock 端点模拟，前端通过以下**调试面板**手动触发：

| 触发操作 | 路径 | 说明 |
|---|---|---|
| 模拟 BPM 审批通过 | 申请单/规则详情页「调试」按钮 | 调用 `/api/v1/mock/bpm/approve` |
| 模拟 BPM 审批驳回 | 同上 | 调用 `/api/v1/mock/bpm/reject` |
| 模拟银行转账成功 | 申请单详情页「调试」按钮 | 调用 `/api/v1/mock/bank/success` |
| 模拟银行转账失败 | 同上 | 调用 `/api/v1/mock/bank/fail` |
| 模拟银行退票 | 同上 | 调用 `/api/v1/mock/bank/return` |
| 模拟浑天仪T+1收入写入 | 收入池页「调试」按钮 | 调用 `/api/v1/mock/hty/sync` |

> 调试按钮仅在 `NODE_ENV=development` 或 `DEMO_MODE=true` 时渲染，生产构建自动隐藏。

---

## 九、开发顺序规划（与后端对齐）

按以下顺序逐模块交付，每个 Stage 完成后前后端联调可体验：

| Stage | 模块 | 主要页面 |
|---|---|---|
| 1 | 基础字典配置 | 游戏/渠道/游戏-渠道关联/三方关系 |
| 2 | 合作方主体 + 银行账户 | 列表/创建/详情/账户管理 |
| 3 | 分成规则 + Mock BPM | 创建/列表/详情/调试面板 |
| 4 | 收入池 + 费用录入 + 结算 | 收入池/待办/账单拆分/结算记录 |
| 5 | 虚拟账户 + 打款申请 + Mock 银行 | 账户看板/申请/详情/调试面板 |
| 6 | 排队/失败/退票处理 | 排队列表/状态处理/重新打款 |
| 7 | 数据看板 + 通知 + 审计日志 | 看板/消息中心/日志 |

---

## 十、待确认事项

- [x] 飞书通知推送由后端处理，前端只负责系统内消息提醒展示
- [x] 多角色权限路由（财务/业务负责人/系统管理员/资金组）需在 UmiJS access 配置中定义
- [ ] 登录方式最终确认：飞书 SSO 还是账号密码（Demo 阶段使用账号密码 + 固定角色切换）
- [ ] 生产部署域名与 CORS 配置
