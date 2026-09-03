# 全局设置页 - PRD

> **文档版本**: v1.0 · 2026-09-03
> **作者**: 平台产品组
> **文档状态**: 评审中
> **关联 HTML 原型**: `settings.html`
> **关联后端服务**: `tenant-service` / `platform-config-service` / `ai-service`
> **关联业务**: 创也·乾坤 —— 已开通租户的后台配置中心

---

## 0. 一句话定位

让租户主账号在**一个页面**完成「店铺授权、平台配置、AI 模型接入、财务汇率」等所有非业务侧的配置项，降低租户的运维成本。

---

## 1. 业务背景与痛点

### 1.1 现状描述

- 配置项散落在 8 个不同的子页面（AI 服务、店铺、财务汇率、OZON 跨境汇率、AI 服务配置、AI 服务预设…）
- 平台接口（OZON/WB）配置需要分多次提交，且没有"配置清单"作为引导
- AI 服务接入需要租户自己看文档填 Base URL，新手租户配置失败率高

### 1.2 痛点本质

1. **配置项分散**：用户不知道"我需要改的功能在哪"，平均配置时间 30+ 分钟。
2. **平台接入门槛高**：OZON/WB 平台接口有"多 Client ID"概念，普通卖家搞不清。
3. **AI 配置文档散落**：Base URL 格式、模型名等文档在 wiki 里，没整合到产品内。
4. **缺少"配置清单"**：用户配置到一半被打断，下次回来不知道从哪继续。

### 1.3 价值主张

- **对用户**：所有配置在一页搞定，AI 服务有"预设"一键填入。
- **对业务**：租户配置成功率从 60% 提升到 90%，客服压力下降。
- **对运营**：配置数据统一在后端 schema，便于做埋点和转化分析。

---

## 2. 角色模型与权限边界

### 2.1 涉及角色

| 角色 | 角色定义 | 在本模块中的典型动作 |
| --- | --- | --- |
| **租户主账号 (Tenant Admin)** | 租户创建者，所有权限 | 修改所有配置 |
| **运营人员 (Operator)** | 被主账号邀请 | 可改 AI 配置、汇率，不可改租户级别配置 |
| **财务 (Finance)** | 仅财务相关权限 | 仅看财务汇率 |
| **只读 (Viewer)** | 审计 / 老板查看 | 仅查看，不可改 |

### 2.2 权限矩阵

| 能力 | Tenant Admin | Operator | Finance | Viewer |
| --- | --- | --- | --- | --- |
| 店铺设置（增删改） | ✓ | ✓ | ✗ | ✗ |
| 平台接口配置 | ✓ | ✓ | ✗ | ✗ |
| 财务汇率 | ✓ | ✗ | ✓ | ✗ |
| AI 服务配置 | ✓ | ✓ | ✗ | ✗ |
| OZON 跨境汇率 | ✓ | ✓ | ✗ | ✗ |
| 保存全局设置 | ✓ | ✓ | ✓（仅汇率） | ✗ |

---

## 3. 信息架构与页面规划

### 3.1 页面入口

```
侧边栏 → 全局设置 → [默认：店铺设置 tab]
```

### 3.2 页面分区

```
┌──────────────────────────────────────────────────────────────┐
│  [Header] Logo Y + 「YeiYei AI · 创也智能」  [保存全局设置]    │
├────────┬─────────────────────────────────────────────────────┤
│ 侧边栏  │  [Tabs 横栏] 店铺设置 / 平台设置 / AI 服务 / 汇率    │
│ (260px)├─────────────────────────────────────────────────────┤
│        │  [店铺设置 tab]                                       │
│        │  ├ 顶部：租户到期横幅（红色/黄色/无）                  │
│        │  ├ 工具栏：已启用数 / 平台租户到期 / [新增店铺] [指引] │
│        │  └ 店铺表格                                          │
│        │                                                      │
│        │  [平台设置 tab]                                       │
│        │  ├ 平台类型选择                                       │
│        │  └ 多 Client ID 列表                                  │
│        │                                                      │
│        │  [AI 服务 tab]                                       │
│        │  ├ 服务商预设（下拉）                                  │
│        │  ├ Base URL / API Key                                │
│        │  ├ 文本/视觉模型选择                                  │
│        │  └ [测试 AI 连接] [测试视觉连接]                       │
│        │                                                      │
│        │  [汇率 tab]                                          │
│        │  ├ OZON 跨境汇率                                     │
│        │  └ 财务汇率                                          │
└────────┴─────────────────────────────────────────────────────┘
```

### 3.3 关键交互清单

| 编号 | 触发元素 | 用户动作 | 预期反馈 | 数据流 |
| --- | --- | --- | --- | --- |
| INT-001 | 侧边栏「全局设置」 | 点击 | 跳转 settings.html | URL 跳转 |
| INT-002 | Tab 切换 | 点击 | 切换右侧内容区 | 前端切换 |
| INT-003 | [新增店铺] | 点击 | 弹窗打开，平台类型 / 接口数 / 启停用 | 前端表单 |
| INT-004 | [保存全局设置]（顶部） | 点击 | 调保存接口，按钮 loading → 成功 toast | `POST /api/tenant/settings` |
| INT-005 | 表格行 [编辑] | 点击 | 复用新增弹窗，回填字段 | — |
| INT-006 | 表格行 [删除] | 点击 | 二次确认弹窗 → 删除 | `DELETE /api/stores/:id` |
| INT-007 | 启用 switch | 点击 | 切换启用状态，立即保存 | `PATCH /api/stores/:id` |
| INT-008 | AI 服务商预设下拉 | 切换 | 自动填入 Base URL / 模型名 | 纯前端 |
| INT-009 | [测试 AI 连接] | 点击 | loading → 通过绿色 ✓ 或失败红色 ✗ | `POST /api/ai/test` |
| INT-010 | 续期租户 / 升级 链接 | 点击 | 新窗口跳 /subscribe.html | URL 跳转 |

---

## 4. 核心功能模块设计

### 4.1 模块 A：店铺设置

#### 4.1.1 租户到期横幅状态

| 状态 | 触发条件 | 样式 | CTA |
| --- | --- | --- | --- |
| 已过期 | `today > expire_at` | 红色背景 + 脉冲 | 「续期租户 / 升级」→ subscribe.html |
| 即将过期 | 0 < diff ≤ 7 天 | 黄色背景 | 「立即续费」→ subscribe.html |
| 永久 | `expire_at == null` | 蓝紫渐变 + 盾牌图标 | 「续期租户 / 升级」 |
| 正常 | diff > 7 天 | 不显示 | — |

#### 4.1.2 店铺字段定义

| 字段 | 类型 | 必填 | 校验 | 说明 |
| --- | --- | --- | --- | --- |
| `store_name` | string | ✓ | 长度 1-200，唯一 | 店铺名称 |
| `platform_type` | enum | ✓ | OZON/本土、OZON/跨境、WB/本土、WB/跨境 | 平台 + 类型组合 |
| `currency` | enum | ✓ | RUB（默认）/ USD / CNY | 本系统币种 |
| `interfaces` | array | ✓ | 长度 1-N | 平台接口（每个含 client_id + api_key） |
| `enabled` | bool | ✓ | — | 是否启用 |

#### 4.1.3 新增店铺弹窗

```
[店铺名称] ────────────────────────
[平台+店铺类型] OZON/本土店铺 ▼
[本系统币种]   RUB（固定）
[平台接口] ──────────────────────────
  ┌─ 接口 1 ─────────────────┐
  │ [Client ID] ───────────── │
  │ [API Key]   ───────────── │
  │ [删除此接口]              │
  └──────────────────────────┘
  [+ 添加接口]
[启停用] ● 启用 / ○ 停用
                              [取消] [保存]
```

**业务规则**：
- 接口至少 1 个，可加多个
- 每个接口需 client_id + api_key
- 保存时调 `POST /api/stores`，成功后表格新增一行

#### 4.1.4 状态机

```
[草稿] -- 保存 --> [已保存]
[已保存] -- 启用/停用 --> [已保存 enabled/disabled]
[已保存] -- 编辑 --> [草稿]
[已保存] -- 删除 --> [删除]
```

### 4.2 模块 B：AI 服务配置

#### 4.2.1 服务商预设

| 预设名 | Base URL | 默认文本模型 | 默认视觉模型 |
| --- | --- | --- | --- |
| 创也智能（推荐） | `https://api.chuangye-intelligence.com/v1` | `chuangye-text-pro` | `chuangye-vision-pro` |
| OpenAI | `https://api.openai.com/v1` | `gpt-4o` | `gpt-4o-vision` |
| DeepSeek | `https://api.deepseek.com/v1` | `deepseek-chat` | — |
| 自定义 | 用户手填 | 用户手填 | 用户手填 |

#### 4.2.2 字段定义

| 字段 | 类型 | 必填 | 校验 | 说明 |
| --- | --- | --- | --- | --- |
| `provider` | enum | ✓ | 上述预设之一 | AI 服务商 |
| `base_url` | string | ✓ | URL 格式 | API 基础地址 |
| `api_key` | string | ✓ | 长度 ≥ 20 | API 密钥 |
| `text_model` | string | ✓ | — | 文本模型名 |
| `vision_model` | string | ✗ | — | 视觉模型名 |
| `text_timeout` | int | ✓ | 1-300 | 文本超时（秒） |
| `vision_timeout` | int | ✓ | 1-300 | 视觉超时（秒） |
| `vision_max_size` | int | ✓ | 256-4096 | 视觉预览最大尺寸（px） |

#### 4.2.3 业务规则

- 选择预设后，**自动填入**所有字段（除 api_key），用户可手动覆盖
- api_key 始终为空（出于安全，需要用户自己填）
- 「测试 AI 连接」只测文本模型，「测试视觉连接」测视觉模型
- 测试通过显示绿色 ✓，失败显示红色 ✗ + 错误原因

### 4.3 模块 C：财务汇率

#### 4.3.1 字段定义

| 字段 | 类型 | 必填 | 校验 |
| --- | --- | --- | --- |
| `cny_to_rub` | decimal | ✓ | > 0, ≤ 100 |
| `usd_to_rub` | decimal | ✓ | > 0, ≤ 200 |
| `cny_to_usd` | decimal | ✓ | > 0, ≤ 10 |
| `effective_at` | date | ✓ | YYYY-MM-DD |
| `source` | enum | ✓ | 手动 / 自动同步（中央银行） |

#### 4.3.2 业务规则

- 汇率修改**立即生效**，但只影响**未来订单**
- 历史订单的汇率**不可变更**
- 「恢复默认」按钮把所有汇率重置为系统默认值

### 4.4 模块 D：OZON 跨境汇率（高级配置）

> v1.1 计划，本期只占位

---

## 5. 数据契约

### 5.1 后端 API

| 接口 | 方法 | 路径 | 请求体 | 响应体 | 错误码 |
| --- | --- | --- | --- | --- | --- |
| 查租户到期信息 | GET | `/api/tenant/info` | — | `{plan_name, expire_at, days_remaining}` | — |
| 查店铺列表 | GET | `/api/stores` | — | `{stores: [Store]}` | — |
| 新增店铺 | POST | `/api/stores` | `Store` | `{store_id}` | 40301-40399 |
| 编辑店铺 | PUT | `/api/stores/:id` | `Store` | `{store_id}` | — |
| 删除店铺 | DELETE | `/api/stores/:id` | — | `{success: boolean}` | — |
| 启用/停用 | PATCH | `/api/stores/:id` | `{enabled: bool}` | `{success: boolean}` | — |
| 查 AI 配置 | GET | `/api/ai/config` | — | `AIConfig` | — |
| 保存 AI 配置 | POST | `/api/ai/config` | `AIConfig` | `{success: boolean}` | — |
| 测试 AI 连接 | POST | `/api/ai/test` | `{provider, text_model}` | `{success, latency_ms, error?}` | — |
| 测试视觉连接 | POST | `/api/ai/test-vision` | `{provider, vision_model}` | `{success, latency_ms, error?}` | — |
| 查汇率 | GET | `/api/exchange-rates` | — | `ExchangeRates` | — |
| 保存汇率 | POST | `/api/exchange-rates` | `ExchangeRates` | `{success: boolean}` | — |

### 5.2 关键字段定义

```typescript
interface Store {
  store_id: string;
  store_name: string;
  platform_type: 'ozon-local' | 'ozon-cross' | 'wb-local' | 'wb-cross';
  currency: 'RUB' | 'USD' | 'CNY';
  interfaces: StoreInterface[];
  enabled: boolean;
  created_at: string;
}

interface StoreInterface {
  client_id: string;
  api_key: string;          // 返回时脱敏为 '****'
}

interface AIConfig {
  provider: 'chuangye' | 'openai' | 'deepseek' | 'custom';
  base_url: string;
  api_key: string;
  text_model: string;
  vision_model?: string;
  text_timeout: number;     // 秒
  vision_timeout: number;
  vision_max_size: number;  // px
}

interface ExchangeRates {
  cny_to_rub: number;
  usd_to_rub: number;
  cny_to_usd: number;
  effective_at: string;
  source: 'manual' | 'auto';
}
```

---

## 6. 交互细节与校验规则

### 6.1 表单校验

| 字段 | 触发时机 | 规则 | 错误提示 |
| --- | --- | --- | --- |
| 店铺名称 | 失焦 + 提交 | 非空，1-200，唯一 | 店铺名称已存在 |
| Client ID | 失焦 + 提交 | 非空，长度 4-50 | 请输入 Client ID |
| API Key | 失焦 + 提交 | 非空，长度 ≥ 20 | 请输入 API Key |
| Base URL | 失焦 + 提交 | URL 格式 | Base URL 格式不正确 |
| 文本模型 | 失焦 + 提交 | 非空 | 请输入文本模型名 |
| 视觉模型 | 失焦（如果填了） | 非空 | — |
| 汇率数值 | 失焦 + 提交 | > 0, ≤ 上限 | 汇率超出合理范围 |

### 6.2 异常路径

| 异常场景 | 触发条件 | 用户感知 | 系统处理 |
| --- | --- | --- | --- |
| 网络断开 | `navigator.onLine === false` | 顶部黄色 banner | 阻断保存 |
| 保存接口 500 | 后端错误 | 顶部红色 toast | 不自动重试 |
| AI 测试超时 | 30 秒无响应 | 红色 ✗ + 「连接超时」 | 立即停止 |
| 删除店铺 | 有未完成任务 | 二次确认弹窗 | 提示「有 N 个 SKU 正在分析」 |
| 多个 tab 同时编辑 | 前端检测 | 锁定非激活 tab | 防止覆盖 |

### 6.3 可访问性

- Tab 用 `role="tablist"` + `role="tab"` + `aria-selected`
- 表格行内操作按钮单独可达
- 「测试 AI 连接」loading 时 `aria-live="polite"` 通知结果

---

## 7. 异常与边界场景

### 7.1 数据边界

- **店铺列表为空**：显示「暂无数据」+ 引导到「新增店铺」
- **店铺数量上限**：根据套餐不同（1m: 1 个, 6m: 5 个, 12m: 20 个），超限提示「请先升级套餐」
- **API Key 长度极长**：maxLength 512，不报错
- **Base URL 含端口**：`https://api.example.com:8443/v1` 正常支持

### 7.2 业务边界

- **删除店铺前**：检查是否有未完成的 AI 任务
- **修改 AI 配置**：所有正在运行的 AI 任务使用旧配置，新任务用新配置
- **汇率并发修改**：乐观锁版本号，最后写入的覆盖前面的，提示「汇率已被他人更新」

---

## 8. 前端实现要点 (HTML → 组件映射)

### 8.1 组件拆分建议

| HTML 区块 | 推荐组件名 | 复用范围 | 备注 |
| --- | --- | --- | --- |
| 侧边栏 | `<AppSidebar>` | 全局 | register / settings / tenants |
| 顶部栏 | `<AppTopbar>` | 全局 | |
| Tabs | `<TabBar>` | 全局 | |
| 租户横幅 | `<TenantBanner>` | settings / subscribe | |
| 工具栏 | `<TableToolbar>` | 全局 | settings / tenants |
| 店铺表格 | `<StoreTable>` | settings | |
| 新增店铺弹窗 | `<StoreFormModal>` | settings | |
| AI 配置表单 | `<AIConfigForm>` | settings | |
| 汇率表单 | `<ExchangeRateForm>` | settings | |
| Switch 开关 | `<Switch>` | 全局 | settings / tenants |

### 8.2 状态管理

- **本地状态**：当前激活 tab、弹窗开关、表单字段
- **服务端状态**：店铺列表（推荐 SWR）、AI 配置、汇率
- **全局状态**：用户信息、租户到期状态

### 8.3 关键技术决策

| 决策点 | 推荐方案 | 理由 |
| --- | --- | --- |
| 表单库 | React Hook Form + Zod | 与 register 一致 |
| Switch 组件 | rc-switch 或自研 | Ant Design 已有 |
| AI 配置持久化 | localStorage + 后端 | 用户切换不会丢失 |
| 测试 AI 连接 | axios + timeout 30s | 简单可控 |

### 8.4 与现有代码的对接

- 复用 `<TenantBanner>`（与 subscribe 通用）
- 复用样式：`@color-danger: #ff4d4f`、`@color-warning: #faad14`
- 国际化：`settings.store.tab.*`、`settings.ai.provider.*`
- 与 `tenants.html` 联动：店铺列表只显示当前租户下的，与租户管理页对应

---

## 9. 验收标准 / Definition of Done

- [ ] **功能完整性**：
  - [ ] 4 个 tab 都能正常切换
  - [ ] 店铺增删改 + 启停用
  - [ ] AI 配置预设切换自动填入
  - [ ] 「测试 AI 连接」能正确显示成功/失败
  - [ ] 汇率修改立即生效
- [ ] **校验规则**：§6.1 所有字段校验均实现
- [ ] **异常路径**：§7 所有边界场景均有测试覆盖
- [ ] **性能**：首屏 < 1.5s，AI 测试 < 30s
- [ ] **浏览器兼容**：Chrome 100+ / Safari 15+ / Edge 100+
- [ ] **响应式**：≥ 1280px、1024px、768px 三个断点
- [ ] **a11y**：键盘可完全操作
- [ ] **埋点**：每个 tab 切换、每次保存、每次 AI 测试
- [ ] **设计走查**：UI 同事 review 通过
- [ ] **对接回归**：
  - [ ] 与 `tenants.html` 的租户状态联动
  - [ ] 与 `subscribe.html` 的「续期」链接正常

---

## 10. 未来迭代 (Out of Scope)

| 版本 | 计划内容 | 价值评估 |
| --- | --- | --- |
| v1.1 | OZON 跨境汇率高级配置 | 高 |
| v1.1 | AI 调用额度监控 | 高 |
| v2.0 | 自动汇率同步（央行） | 中 |
| v2.0 | 团队子账号 + 权限分配 | 高 |

---

## 11. 修订记录

| 版本 | 日期 | 修改人 | 修改内容 |
| --- | --- | --- | --- |
| v1.0 | 2026-09-03 | 平台产品组 | 初稿 |
