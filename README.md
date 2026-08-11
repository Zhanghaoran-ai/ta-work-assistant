# TA 工作助手
> 支持飞书招聘、北森、Moka三大主流招聘系统的个人智能工作台账与任务管理工具
[![version](https://img.shields.io/badge/version-v3.0.0-blue)](https://github.com/Zhanghaoran-ai/ta-work-assistant)
[![license](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![systems](https://img.shields.io/badge/systems-飞书招聘%20%7C%20北森%20%7C%20Moka-orange)]()
## 📋 项目介绍
TA工作助手是面向招聘专员/HRBP的**个人智能工作台账**，采用**多系统适配器架构**，支持飞书招聘（Lark Hire）、北森（iTalent）、Moka三大主流招聘系统。从**待办任务视角**出发，帮你梳理任务优先级、时间分布和精力分配建议。
**核心定位**：不是数据报表工具，而是你的**个人任务管家**——告诉你"今天该做什么、先做什么、怎么做最高效"，并且每条任务都带对应招聘系统的直达链接，看完就能点进去处理。
**多系统支持**：
- ✅ 飞书招聘（Lark Hire）：飞书开放平台 hire/v1 API
- ✅ 北森（iTalent）：北森开放平台 RecruitV6 API
- ✅ Moka：Moka开放平台 api-platform/v1 API
- ✅ 多系统聚合：同时配置多个系统，统一管理待办
## ✨ 核心功能
### 1. 多系统适配架构 ⭐ 新增
- 统一数据模型，上层业务逻辑与底层系统解耦
- 适配器模式，新增招聘系统只需实现适配器
- 支持单系统和多系统聚合两种模式
- 两种聚合模式：merge（合并排序）/ group（按系统分组）
- 降级策略：某系统失败不影响其他系统
### 2. 智能待办清单
- 待办任务汇总（待评估简历、待面试、待写评估）
- 优先级智能排序（急招 > 内推 > 即将超时 > 高匹配）
- 超时预警（黄/红两级预警）
- 时间与精力分配建议
- 每条任务带来源系统标注和直达链接
### 3. 定时推送与主动提醒
- **早间每日看板**：工作日早上9点推送当日工作安排
- **午间提醒**：下午面试提醒 + 上午待办回顾
- **下班前收尾**：今日完成情况 + 未完成提醒
- **周度汇总**：本周工作汇总 + 下周待办预览
- **事件驱动提醒**：新急招简历、面试前提醒、超时预警等
- 多系统模式下自动聚合所有系统待办
### 4. 工作量与效率分析
- 工作量概览与分布统计
- 效率分析与趋势对比
- 多系统效率对比
- 候选人画像统计
### 5. 简历处理辅助
- 简历快速摘要
- 多候选人横向对比（支持跨系统对比）
- 智能初筛（匹配度评分 + 结论）
### 6. 面试支持
- 面试准备包（背景摘要 + 关注点 + 提问方向）
- 面试纪要整理
- 面试题库推荐
### 7. 流程追踪
- 候选人全流程追踪
- 时效分析与瓶颈识别
- 岗位招聘进度
### 8. 数据沉淀
- 个人面试经验库（飞书多维表格）
- 评估标准沉淀（越用越懂你的偏好）
## 🚀 快速开始
### 前置条件
- 企业已开通以下至少一个招聘系统：
  - 飞书招聘（Lark Hire）
  - 北森（iTalent）
  - Moka
- 拥有对应招聘系统的开放平台应用权限
- 应用已开通招聘相关API权限
### 三步快速上手（以飞书为例）
1. **创建飞书自建应用**
   - 登录飞书开放平台，创建自建应用
   - 开通招聘相关权限（`hire:application:readonly` 等）
   - 获取 App ID 和 App Secret
2. **配置TA工作助手**
   - 配置 App ID / App Secret
   - 选择认证模式（推荐用户身份模式）
   - 配置超时预警、优先级规则等参数
3. **开始使用**
   - 说"帮我看看今天有什么待办"即可查询
   - 说"帮我设置每天早上9点推送"开启定时推送
**北森/Moka配置**：参考 [快速上手指南](references/quick-start.md) 中的对应方案
**多系统配置**：分别配置各系统后，说"把飞书和Moka的待办合并展示"
详细配置说明请参考：[快速上手指南](references/quick-start.md)
## 📁 目录结构
```
ta-work-assistant/
├── SKILL.md                          # 技能主文件（核心说明）
├── README.md                         # 项目说明（本文件）
└── references/
    ├── quick-start.md                # 快速上手指南
    ├── adapter-design.md             # 多系统适配器设计文档
    ├── api-reference-feishu.md       # 飞书招聘API参考
    ├── api-reference-beisen.md       # 北森API参考
    ├── api-reference-moka.md         # MokaAPI参考
    ├── output-templates.md           # 输出模板参考
    ├── error-handling.md             # 异常处理与错误提示规范
    └── quality-checklist.md          # 质量校验清单
```
## 🔧 配置说明
### 飞书招聘配置项
| 配置项 | 说明 |
|--------|------|
| `feishu_app_id` | 飞书自建应用的App ID |
| `feishu_app_secret` | 飞书自建应用的App Secret |
| `feishu_auth_mode` | 认证模式：`user`（用户身份）/ `tenant`（应用身份） |
| `feishu_domain` | 域名：`feishu`（国内）/ `lark`（国际） |
### 北森配置项
| 配置项 | 说明 |
|--------|------|
| `beisen_tenant_id` | 北森租户ID |
| `beisen_app_key` | 北森应用Key |
| `beisen_app_secret` | 北森应用Secret |
| `beisen_web_domain` | 北森招聘系统Web域名 |
| `beisen_user_id` | 北森用户ID（用于筛选个人待办） |
### Moka配置项
| 配置项 | 说明 |
|--------|------|
| `moka_client_id` | Moka应用Client ID |
| `moka_client_secret` | Moka应用Client Secret |
| `moka_web_domain` | Moka招聘系统Web域名 |
| `moka_user_email` | Moka用户邮箱（用于筛选个人待办） |
### 通用可选配置项
| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `aggregation_mode` | 多系统聚合模式：`merge` / `group` | merge |
| `enabled_systems` | 启用的系统列表 | 配置的所有系统 |
| `timeout_warning_days` | 简历评估超时预警天数（黄） | 3 |
| `timeout_critical_days` | 简历评估严重超天天数（红） | 7 |
| `push_enabled` | 是否启用定时推送功能 | false |
| `push_morning_time` | 早间推送时间 | 09:00 |
| `event_reminder_enabled` | 是否启用事件驱动提醒 | false |
完整配置项请参考 [SKILL.md](SKILL.md)。
## 📊 输出示例
### 多系统聚合待办（merge模式）
```
📝 待评估简历（8份）
━━━━━━━━━━━━━━━━━━━━
[飞书] 4份 | [Moka] 2份 | [北森] 2份
[飞书] 🔥 1. 张三 - 前端开发（急招，已超时2天）
   5年经验 / 本科 / 字节跳动
   👉 [去评估](https://hire.feishu.cn/...)
[Moka] ⭐ 2. 李四 - 产品经理（内推）
   3年经验 / 硕士 / 腾讯
   👉 [去评估](https://app.mokahr.com/...)
[北森] 3. 王五 - 数据分析（已超时1天）
   4年经验 / 本科 / 美团
   👉 [去评估](https://hire.italent.cn/...)
```
更多模板请参考：[输出模板参考](references/output-templates.md)
## 🔌 数据来源
采用**多系统适配器架构**，支持三大主流招聘系统：
| 系统 | API | 认证方式 | 特点 |
|------|-----|---------|------|
| 飞书招聘 | 飞书开放平台 hire/v1 | App ID/Secret + OAuth | 接口规范，支持用户身份 |
| 北森 | 北森开放平台 RecruitV6 | Key/Secret + Bearer Token | 接口丰富，免登录链接 |
| Moka | Moka开放平台 api-platform/v1 | Basic Auth | 候选人信息丰富，外部链接 |
**核心思路**：三个系统都没有直接的"我的待办"API，通过"全量数据 + 按处理人/面试官筛选 + 按状态筛选"的方式间接实现个人待办视角。
详细接口说明请参考：
- [飞书招聘API参考](references/api-reference-feishu.md)
- [北森API参考](references/api-reference-beisen.md)
- [MokaAPI参考](references/api-reference-moka.md)
- [多系统适配器设计文档](references/adapter-design.md)
## ⚠️ 能力边界
### ✅ 可以做到
- 支持飞书招聘、北森、Moka三大系统
- 多系统聚合，统一管理待办
- 个人维度的工作量统计和分析
- 待办提醒和超时预警
- 简历摘要、对比、初筛辅助
- 面试准备和纪要整理
- 定时推送和事件驱动提醒
- 个人数据沉淀和经验积累
- 某系统失败时降级展示其他系统
### ❌ 做不到
- 直接在招聘系统中执行写操作（改状态、推进流程）
- 获取其他HR或团队的招聘数据（用户身份模式下）
- 获取全公司级别的招聘统计
- 自动执行需要人工确认的操作
- 跨系统执行写操作（本技能为只读模式）
> **原则**：AI辅助决策，人工确认执行
## 📝 版本历史
| 版本 | 日期 | 更新内容 |
|------|------|----------|
| v3.0.0 | 2026-08-11 | **多系统适配重大更新**：采用适配器架构，新增北森（iTalent）和Moka支持；统一数据模型，支持merge/group两种聚合模式；降级策略，某系统失败不影响其他；新增各系统API参考文档和适配器设计文档；向后兼容v2.x配置 |
| v2.1.0 | 2026-08-10 | 新增定时推送与主动提醒功能：支持早间/午间/下班前/周度多种推送节奏，事件驱动即时提醒，20+可配置项，自然语言对话式配置管理；从被动查询升级为主动助手 |
| v2.0.0 | 2026-08-10 | 通用版重大更新：数据源改为飞书开放平台标准API，支持所有飞书招聘企业；增加触发条件、异常处理、配置说明、质量校验、验收标准；明确数据沉淀实现方案 |
| v1.0.0 | 2026-08-09 | 初始版本：基于字节内部bytedance-hire，7大功能模块，6套输出模板 |
## 🤝 贡献
欢迎提交 Issue 和 Pull Request！
### 新增招聘系统支持
如需新增其他招聘系统支持，请参考 [适配器设计文档](references/adapter-design.md) 中的扩展指南。
## 📄 许可证
MIT License
---
**如果这个工具对你有帮助，欢迎给个 Star ⭐**
