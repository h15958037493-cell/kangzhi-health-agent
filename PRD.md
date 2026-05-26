# 健康宣教系统数据大屏 产品需求文档

| 字段 | 内容 |
|------|------|
| 文档版本 | v1.1 |
| 状态 | 评审中 |
| 创建日期 | 2026-05-25 |
| 评审人 | - |

---

## 1. 背景与目标

### 1.1 问题背景

健康宣教系统已在多家医院上线运营，每个医院独立配置自身的病种科普内容（如屈光手术、糖尿病、消化性溃疡等）。目前缺乏统一的数据可视化平台，运营人员无法直观掌握各医院各病种的推送效果和用户查看情况，难以评估宣教成效和优化资源配置。

### 1.2 产品目标

- **用户目标**：为健康宣教运营人员提供多医院数据查看工具，支持切换医院查看对应数据，实时掌握推送与查看动态
- **业务目标**：通过数据大屏集中展示真实推送/查看数据，支持按医院隔离数据，辅助运营决策
- **本次范围**：开发健康宣教系统数据大屏 Web 页面，接入真实后端数据，支持多医院切换、病种按医院隔离、从规则库动态获取

### 1.3 成功指标（待定）

---

## 2. 用户分析

### 2.1 目标用户

**主要用户**：健康宣教运营人员、科室负责人

**用户画像**：
- 角色：医院宣教科室工作人员
- 使用场景：每日晨会数据查看、周报数据汇报、日常运营监控、多院区数据对比
- 技术熟练度：一般

### 2.2 用户痛点与解法

| 痛点 | 当前解决方式 | 我们的解法 |
|------|------------|----------|
| 数据分散，无法集中查看 | 登录多个后台 | 数据大屏一处汇总 |
| 推送效果无法直观对比 | 导出 Excel 手动分析 | 图表可视化对比 |
| 查看率等指标需要手动计算 | 人工统计 | 自动实时计算展示 |
| 无法快速了解详情 | 逐个进入后台 | 点击病种卡片弹出详情 |

---

## 3. 功能需求

### 3.1 功能概览

数据大屏以 Web 单页面形式呈现，通过 ECharts 图表库实现数据可视化。页面顶部支持医院切换器，切换后加载对应医院的病种数据。病种从后端规则库动态获取，不同医院可配置不同病种。每个图表数据均按选中医院隔离展示。

**核心业务流程**：
1. 用户进入大屏页面 → 系统加载医院列表 → 用户选择医院
2. 系统根据选中医院调用 `/api/dashboard/hospitals/{hospital_id}/diseases` 获取该医院病种列表
3. 系统调用 `/api/dashboard/hospitals/{hospital_id}/summary` 获取该医院汇总数据
4. 各图表组件根据病种数据渲染，每 30 秒自动刷新

---

### 3.2 功能模块：医院切换器

**描述**：页面顶部左侧医院切换下拉框，支持在多个医院之间切换

**主流程**：
1. 页面加载时，调用 `/api/hospitals` 获取当前用户有权限访问的医院列表
2. 默认选中第一个医院（或上次访问的医院）
3. 用户切换医院时，清空当前图表数据，加载新医院数据

**数据字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| hospital_id | String | 医院唯一标识 |
| hospital_name | String | 医院名称 |
| hospital_code | String | 医院编码 |

**交互规则**：
- 医院切换后，所有图表数据立即刷新，显示新医院数据
- 切换过程中显示 Loading 状态
- 医院列表包含"全部医院"选项，选择后展示所有医院的汇总数据（不含病种隔离）

---

### 3.3 功能模块：顶部统计卡片

**描述**：页面顶部 4 个统计卡片，实时展示从后端获取的当前选中医院的汇总数据

**主流程**：
1. 用户选择医院后，调用 `/api/dashboard/hospitals/{hospital_id}/summary` 获取汇总数据
2. 渲染累计推送次数、今日推送次数、累计查看次数、综合查看率
3. 每 30 秒自动刷新一次数据

**数据字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| total_push_count | Integer | 该医院累计推送次数（所有病种之和） |
| today_push_count | Integer | 该医院今日推送次数 |
| total_view_count | Integer | 该医院累计查看次数（所有病种之和） |
| overall_view_rate | Float | 该医院综合查看率（百分比，保留1位小数） |
| update_time | String | 数据更新时间（ISO 8601 格式） |

**业务规则**：
- 综合查看率 = 累计查看次数 / 累计推送次数 × 100%
- 数据为空时显示 "--"

---

### 3.4 功能模块：病种推送与查看对比柱状图

**描述**：展示当前医院配置的病种的推送次数与查看次数对比柱状图，病种从该医院的规则库动态获取

**主流程**：
1. 用户选择医院后，调用 `/api/dashboard/hospitals/{hospital_id}/diseases` 获取该医院病种数据
2. 蓝色柱子表示推送次数，绿色柱子表示查看次数
3. X 轴为该医院配置的病种名称，Y 轴为次数
4. 每 30 秒自动刷新

**数据字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| disease_id | String | 病种唯一标识（与医院绑定） |
| disease_name | String | 病种名称 |
| push_count | Integer | 该医院该病种累计推送次数 |
| view_count | Integer | 该医院该病种累计查看次数 |
| view_rate | Float | 查看率（百分比，保留1位小数） |

---

### 3.5 功能模块：病种详情网格

**描述**：右侧病种卡片网格，支持点击查看单个病种详情。病种列表从当前医院的规则库动态获取。

**主流程**：
1. 用户选择医院后，调用 `/api/dashboard/hospitals/{hospital_id}/diseases` 获取该医院病种列表
2. 每个卡片显示病种名称、推送次数、查看次数
3. 点击任意卡片，调用 `/api/dashboard/hospitals/{hospital_id}/diseases/{disease_id}` 获取详情并弹窗

**异常流程**：

| 异常情况 | 系统行为 | 用户提示文案 |
|---------|---------|------------|
| 该医院无配置病种 | 显示 "该医院暂未配置病种" | "暂无数据" |
| 单病种详情加载失败 | 弹窗显示错误信息 | "数据加载失败，请稍后重试" |

---

### 3.6 功能模块：近 7 天推送趋势折线图

**描述**：展示最近 7 天的推送和查看趋势，数据来源于后端当前医院

**主流程**：
1. 用户选择医院后，调用 `/api/dashboard/hospitals/{hospital_id}/trend?days=7` 获取近 7 天趋势数据
2. 蓝色折线表示推送，绿色折线表示查看
3. X 轴为周一至周日，Y 轴为次数

**数据字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| date | String | 日期（YYYY-MM-DD） |
| push_count | Integer | 当日该医院推送次数 |
| view_count | Integer | 当日该医院查看次数 |

---

### 3.7 功能模块：推送分布饼图

**描述**：展示当前医院各病种在总推送中的占比分布

**主流程**：
1. 用户选择医院后，调用 `/api/dashboard/hospitals/{hospital_id}/distribution` 获取推送分布
2. 使用环形饼图（内半径 45%，外半径 75%）
3. 每个扇区显示病种名称和推送次数

**数据字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| disease_name | String | 病种名称 |
| push_count | Integer | 该医院该病种累计推送次数 |
| push_ratio | Float | 占比（百分比，保留1位小数） |

---

### 3.8 功能模块：查看率排行横向条形图

**描述**：按查看率从低到高排序展示当前医院的各病种

**主流程**：
1. 用户选择医院后，基于 `/api/dashboard/hospitals/{hospital_id}/diseases` 返回数据计算查看率
2. Y 轴为病种名称（倒序排列），X 轴为查看率百分比
3. 橙色柱状图，最大值 100%

---

### 3.9 功能模块：详情弹窗

**描述**：点击各模块"查看详情"或病种卡片时，弹出模态框展示详细信息

**弹窗类型**：
1. **病种推送与查看详情**：调用 `/api/dashboard/hospitals/{hospital_id}/summary`，展示该医院总推送/总查看/平均查看率
2. **所有病种详情**：调用 `/api/dashboard/hospitals/{hospital_id}/diseases`，展示该医院病种列表（可点击）
3. **近 7 天趋势详情**：调用 `/api/dashboard/hospitals/{hospital_id}/trend?days=7`，展示周汇总
4. **推送分布详情**：调用 `/api/dashboard/hospitals/{hospital_id}/distribution`，展示各病种明细
5. **查看率排行榜**：基于 `/api/dashboard/hospitals/{hospital_id}/diseases` 排序展示
6. **单个病种详情**：调用 `/api/dashboard/hospitals/{hospital_id}/diseases/{disease_id}`，展示该医院该病种基本信息 + 宣教数据

**交互规则**：
- 点击遮罩层可关闭弹窗
- 点击关闭按钮可关闭弹窗
- 弹窗宽度最大 800px，高度最大 80vh，超出可滚动

---

## 4. 数据埋点需求

### 4.1 埋点概述

数据大屏的数据来源于健康宣教系统后端数据库，埋点数据由用户行为触发后端记录，大屏通过 API 拉取并展示。埋点分为两类：**前端事件埋点**（用户行为）和**后端数据埋点**（推送/查看记录）。

---

### 4.2 后端数据埋点（推送与查看记录）

健康宣教系统后端需记录每次推送和查看行为，作为大屏数据的源头。病种与医院绑定，推送和查看记录均需关联医院 ID。

#### 4.2.1 医院信息表（hospital）

| 字段名 | 数据类型 | 说明 | 示例 |
|--------|---------|------|------|
| id | BigInt | 主键 | 1 |
| hospital_id | String | 医院唯一标识 | H001 |
| hospital_name | String | 医院名称 | 衢州市人民医院 |
| hospital_code | String | 医院编码 | QZRMYY |
| created_at | DateTime | 创建时间 | 2026-01-01 00:00:00 |

#### 4.2.2 病种规则配置表（disease_rule）

每个医院独立配置自己的病种规则，病种与医院绑定：

| 字段名 | 数据类型 | 说明 | 示例 |
|--------|---------|------|------|
| id | BigInt | 主键 | 1 |
| hospital_id | String | 所属医院 ID | H001 |
| disease_id | String | 病种 ID | diabetes |
| disease_name | String | 病种名称 | 糖尿病 |
| description | String | 病种描述 | 糖尿病健康宣教内容 |
| target_age | String | 目标年龄段 | 35-70岁 |
| risk_level | String | 风险等级 | 高/中/低 |
| push_rule_count | Int | 推送规则数量 | 5 |
| department | String | 所属科室 | 内分泌科 |
| status | String | 状态 | enabled/disabled |
| created_at | DateTime | 创建时间 | 2026-01-01 00:00:00 |
| updated_at | DateTime | 更新时间 | 2026-05-25 10:00:00 |

#### 4.2.3 推送记录表（push_log）

每次向用户推送宣教内容时，记录一条推送日志，按医院隔离：

| 字段名 | 数据类型 | 说明 | 示例 |
|--------|---------|------|------|
| id | BigInt | 主键 | 100001 |
| hospital_id | String | 医院 ID | H001 |
| disease_id | String | 病种 ID（关联 disease_rule） | diabetes |
| disease_name | String | 病种名称 | 糖尿病 |
| user_id | String | 推送目标用户 ID | U123456 |
| push_time | DateTime | 推送时间 | 2026-05-25 10:30:00 |
| push_channel | String | 推送渠道 | wechat/短信/APP |
| push_content_id | String | 推送内容 ID | C001 |
| push_status | String | 推送状态 | success/failed |
| created_at | DateTime | 记录创建时间 | 2026-05-25 10:30:01 |

#### 4.2.4 查看记录表（view_log）

每次用户点击查看宣教内容时，记录一条查看日志，按医院隔离：

| 字段名 | 数据类型 | 说明 | 示例 |
|--------|---------|------|------|
| id | BigInt | 主键 | 200001 |
| hospital_id | String | 医院 ID | H001 |
| disease_id | String | 病种 ID（关联 disease_rule） | diabetes |
| disease_name | String | 病种名称 | 糖尿病 |
| user_id | String | 查看用户 ID | U123456 |
| view_time | DateTime | 查看时间 | 2026-05-25 10:35:22 |
| view_duration | Int | 查看时长（秒） | 45 |
| view_source | String | 来源 | push_click/history_browse |
| push_log_id | BigInt | 关联的推送记录 ID（可空） | 100001 |
| created_at | DateTime | 记录创建时间 | 2026-05-25 10:35:23 |

---

### 4.3 后端数据聚合 API

大屏前端通过调用后端 API 获取聚合数据，后端基于 push_log 和 view_log 表进行统计计算。所有 API 均按医院隔离数据。

#### 4.3.1 医院列表接口

**接口路径**：`GET /api/hospitals`

**请求参数**：无

**响应字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| hospital_id | String | 医院唯一标识 |
| hospital_name | String | 医院名称 |

#### 4.3.2 医院汇总数据接口

**接口路径**：`GET /api/dashboard/hospitals/{hospital_id}/summary`

**路径参数**：`hospital_id` - 医院 ID

**响应字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| hospital_id | String | 医院 ID |
| hospital_name | String | 医院名称 |
| total_push_count | Integer | 该医院累计推送次数 |
| today_push_count | Integer | 该医院今日推送次数 |
| total_view_count | Integer | 该医院累计查看次数 |
| overall_view_rate | Float | 该医院综合查看率 |
| disease_count | Integer | 该医院配置的病种数量 |
| update_time | String | 数据更新时间 |

**SQL 计算逻辑**：
```sql
SELECT
  p.hospital_id,
  COUNT(DISTINCT p.id) as total_push_count,
  COUNT(DISTINCT CASE WHEN DATE(p.push_time) = CURDATE() THEN p.id END) as today_push_count,
  COUNT(DISTINCT v.id) as total_view_count
FROM push_log p
LEFT JOIN view_log v ON p.hospital_id = v.hospital_id
WHERE p.hospital_id = :hospital_id AND p.push_status = 'success'
GROUP BY p.hospital_id;
```

#### 4.3.3 医院病种列表接口

**接口路径**：`GET /api/dashboard/hospitals/{hospital_id}/diseases`

**路径参数**：`hospital_id` - 医院 ID

**响应字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| disease_id | String | 病种 ID |
| disease_name | String | 病种名称 |
| push_count | Integer | 该医院该病种累计推送次数 |
| view_count | Integer | 该医院该病种累计查看次数 |
| view_rate | Float | 查看率（百分比） |

**SQL 计算逻辑**：
```sql
SELECT
  p.disease_id,
  p.disease_name,
  COUNT(DISTINCT p.id) as push_count,
  COUNT(DISTINCT v.id) as view_count,
  ROUND(COUNT(DISTINCT v.id) / COUNT(DISTINCT p.id) * 100, 1) as view_rate
FROM push_log p
LEFT JOIN view_log v ON p.disease_id = v.disease_id AND p.hospital_id = v.hospital_id
WHERE p.hospital_id = :hospital_id
GROUP BY p.disease_id, p.disease_name
ORDER BY push_count DESC;
```

#### 4.3.4 医院病种详情接口

**接口路径**：`GET /api/dashboard/hospitals/{hospital_id}/diseases/{disease_id}`

**路径参数**：
- `hospital_id` - 医院 ID
- `disease_id` - 病种 ID

**响应字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| disease_id | String | 病种 ID |
| disease_name | String | 病种名称 |
| description | String | 病种描述 |
| target_age | String | 目标年龄段 |
| risk_level | String | 风险等级（高/中/低） |
| push_count | Integer | 该医院该病种累计推送次数 |
| view_count | Integer | 该医院该病种累计查看次数 |
| view_rate | Float | 查看率 |
| push_rule_count | Integer | 推送规则数量 |
| department | String | 所属科室 |

#### 4.3.5 医院近 7 天趋势接口

**接口路径**：`GET /api/dashboard/hospitals/{hospital_id}/trend`

**路径参数**：`hospital_id` - 医院 ID

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| days | Integer | 否 | 查询天数，默认 7 |

**响应字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| date | String | 日期（YYYY-MM-DD） |
| push_count | Integer | 当日该医院推送次数 |
| view_count | Integer | 当日该医院查看次数 |

**SQL 计算逻辑**：
```sql
SELECT
  DATE(p.push_time) as date,
  COUNT(DISTINCT p.id) as push_count,
  COUNT(DISTINCT v.id) as view_count
FROM push_log p
LEFT JOIN view_log v ON DATE(p.push_time) = DATE(v.view_time) AND p.disease_id = v.disease_id AND p.hospital_id = v.hospital_id
WHERE p.hospital_id = :hospital_id AND p.push_time >= DATE_SUB(CURDATE(), INTERVAL 7 DAY)
GROUP BY DATE(p.push_time)
ORDER BY date ASC;
```

#### 4.3.6 医院推送分布接口

**接口路径**：`GET /api/dashboard/hospitals/{hospital_id}/distribution`

**路径参数**：`hospital_id` - 医院 ID

**响应字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| disease_id | String | 病种 ID |
| disease_name | String | 病种名称 |
| push_count | Integer | 该医院该病种累计推送次数 |
| push_ratio | Float | 占比（百分比） |

---

### 4.4 前端行为埋点

大屏页面本身需记录用户行为，用于分析使用情况。

#### 4.4.1 埋点事件清单

| 事件名 | 触发时机 | 上报参数 | 目的 |
|--------|---------|---------|------|
| page_view | 页面加载完成 | page_name, refer, device, browser | 统计页面访问量 |
| chart_hover | 鼠标悬停图表 | chart_type, disease_id, hover_duration | 分析用户关注点 |
| detail_click | 点击"查看详情" | modal_type, click_source | 统计详情查看率 |
| disease_card_click | 点击病种卡片 | disease_id, disease_name | 分析病种关注度 |
| refresh_click | 点击手动刷新 | data_type, refresh_time | 统计手动刷新频率 |
| modal_open | 弹窗打开 | modal_type, open_source | 分析弹窗使用情况 |
| modal_close | 弹窗关闭 | modal_type, stay_duration | 分析弹窗停留时长 |

#### 4.4.2 埋点数据格式

```json
{
  "event": "detail_click",
  "event_time": "2026-05-25T10:30:00+08:00",
  "page_name": "健康宣教数据大屏",
  "user_id": "admin_001",
  "device": "Windows 10",
  "browser": "Chrome 120",
  "params": {
    "modal_type": "disease_detail",
    "click_source": "disease_card"
  }
}
```

#### 4.4.3 埋点上报方式

- 使用.gif 1x1 像素埋点（无阻塞，无性能影响）
- 数据上报至后端 `/api/analytics/event` 接口
- 离线用户访问时，埋点数据本地缓存，恢复网络后重试上报

---

### 4.5 数据更新机制

#### 4.5.1 实时数据刷新

- 大屏页面加载时调用 API 获取最新数据
- 每 30 秒自动调用 `/api/dashboard/summary` 和 `/api/dashboard/diseases` 刷新数据
- 用户可手动点击刷新按钮强制刷新
- 数据更新时图表平滑过渡，不闪烁

#### 4.5.2 数据延迟处理

- 后端每次 API 请求时返回 `update_time` 时间戳
- 前端比对时间，若超过 5 分钟未更新，显示警告提示"数据更新可能存在延迟"

---

## 5. 非功能性需求

### 5.1 性能要求

| 指标 | 要求 | 场景 |
|------|------|------|
| 页面首次加载 | < 3s | 4G 网络，ECharts CDN 加载完成 |
| API 接口响应 | P99 < 500ms | 正常负载 |
| 图表数据刷新 | < 1s | 用户无感知 |
| 前端埋点上报 | 异步，无阻塞 | 不影响页面性能 |

### 5.2 安全要求

- [ ] 所有 API 接口需 Token 鉴权，防止未授权访问
- [ ] 埋点数据需脱敏处理，不包含用户敏感信息
- [ ] 后端 API 需防止 SQL 注入 / XSS 攻击
- [ ] 大屏页面部署需 HTTPS

### 5.3 兼容性

| 维度 | 要求 |
|------|------|
| 浏览器 | Chrome 90+，Edge 90+，Firefox 88+，Safari 14+ |
| 屏幕分辨率 | 最佳展示 1920px 及以上 |
| 操作系统 | Windows / macOS / Linux |

### 5.4 设计规范

- **设计风格**：浅色系，白色背景，蓝色主色调
- **主色**：#2563eb（蓝色）
- **辅助色**：#16a34a（绿色）、#ca8a04（黄色）、#f97316（橙色）
- **背景色**：#f8fafc（米白）
- **卡片背景**：白色，1px 边框 #e2e8f0
- **圆角**：12px（卡片）、8px（小元素）

---

## 6. 交互与设计说明

### 6.1 页面布局

```
┌────────────────────────────────────────────────────────────┐
│  衢州市人民医院 ▼  │    健康宣教系统数据大屏    │  最后更新: 10:30:00  🔄  │
├──────────┬──────────┬──────────┬──────────────────────────┤
│ 累计推送  │ 今日推送  │ 累计查看  │        综合查看率          │
│  2,748   │    56    │  1,688   │         61.4%            │
├─────────────────────────┬────────────┬────────────────────┤
│  病种推送与查看对比      │  病种详情   │     数据概览         │
│      (柱状图)           │  (网格)    │  病种数量: 9         │
│                        │            │  平均查看率: 61.4%    │
├─────────────────────────┴────────────┴────────────────────┤
│   近7天推送趋势    │    推送分布     │     查看率排行        │
│    (折线图)       │    (饼图)      │    (横向条形图)       │
└────────────────────────────────────────────────────────────┘
```

### 6.2 交互规范

- **页面加载**：显示骨架屏，API 返回后渐显数据
- **数据刷新**：右上角显示"最后更新时间"，刷新时图标旋转动画
- **鼠标悬停**：图表显示 Tooltip，展示具体数值
- **点击"查看详情"**：弹出模态框，加载时显示 Loading 状态
- **点击病种卡片**：弹出该病种详情弹窗
- **点击遮罩/关闭按钮**：关闭弹窗

---

## 7. 明确不做（Out of Scope）

> 本期不包含：
> - 用户登录认证（大屏页面公开访问）
> - 数据导出功能（Excel/PDF）
> - 移动端适配优化
> - 多语言切换
> - 实时消息推送（大屏被动接收）

---

## 8. 迭代规划

### MVP（本期 v1.0）
- [ ] 医院切换功能
- [ ] 顶部 4 个统计卡片（实时数据，按医院隔离）
- [ ] 病种推送与查看对比柱状图（病种从规则库动态获取）
- [ ] 病种详情网格（可点击）
- [ ] 近 7 天推送趋势折线图（按医院隔离）
- [ ] 推送分布饼图（按医院隔离）
- [ ] 查看率排行横向条形图（按医院隔离）
- [ ] 详情弹窗功能
- [ ] 浅色系 UI 设计
- [ ] 前端行为埋点
- [ ] 后端 API 接口开发（按医院隔离）
- [ ] 数据库表设计（hospital、disease_rule、push_log、view_log）

### 后续迭代

| 版本 | 功能方向 | 优先级 | 备注 |
|------|---------|--------|------|
| v1.1 | 用户维度统计 | P1 | 按用户查看偏好推荐 |
| v2.0 | 数据导出 | P2 | Excel/PDF 周报生成 |
| v2.1 | 实时大屏模式 | P1 | 电视大屏自动轮播 |

---

## 9. 风险与假设

| 类型 | 描述 | 影响 | 应对方案 |
|------|------|------|---------|
| 风险 | 后端 API 性能不足 | 高 | 添加 Redis 缓存，API 结果缓存 30s |
| 风险 | 埋点数据量大影响数据库 | 中 | 分表分区，定期归档历史数据 |
| 假设 | 后端已具备 push_log 和 view_log 数据基础 | - | 需后端确认现有表结构 |
| 假设 | 大屏页面无需登录即可访问 | - | 需安全评估确认 |

---

## 10. 附录

### 10.1 后端接口文档清单

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/hospitals` | GET | 医院列表 |
| `/api/dashboard/hospitals/{hospital_id}/summary` | GET | 医院汇总数据 |
| `/api/dashboard/hospitals/{hospital_id}/diseases` | GET | 医院病种列表（从规则库获取） |
| `/api/dashboard/hospitals/{hospital_id}/diseases/{disease_id}` | GET | 医院病种详情 |
| `/api/dashboard/hospitals/{hospital_id}/trend` | GET | 医院趋势数据 |
| `/api/dashboard/hospitals/{hospital_id}/distribution` | GET | 医院推送分布 |
| `/api/analytics/event` | POST | 埋点上报 |

### 10.2 数据库表清单

| 表名 | 说明 |
|------|------|
| hospital | 医院信息表 |
| disease_rule | 病种规则配置表（病种与医院绑定） |
| push_log | 推送记录表（按医院隔离） |
| view_log | 查看记录表（按医院隔离） |

### 10.3 参考资料

- 仓库地址：https://github.com/h15958037493-cell/kangzhi-health-agent
- 页面入口文件：index.html（前端大屏页面）
- 依赖库：ECharts 5.4.3（CDN）