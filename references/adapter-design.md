# 多系统适配器设计文档
## 概述
本文档描述 TA 工作助手技能的**多系统适配器架构**设计，包括统一数据模型、适配器接口规范、各系统适配方案、聚合模式设计等。
**设计目标**：
1. 上层业务逻辑与底层招聘系统解耦
2. 新增招聘系统只需实现适配器，无需修改业务逻辑
3. 支持单系统和多系统聚合两种模式
4. 统一的异常处理和降级策略
---
## 架构设计
### 整体架构
```
┌─────────────────────────────────────────────────────────────┐
│                      用户交互层                              │
│              （自然语言指令 / 定时推送触发）                  │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      业务逻辑层                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────┐ │
│  │待办整理  │ │优先级排序│ │超时预警  │ │定时推送/提醒   │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────────┘ │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                    │
│  │简历分析  │ │面试准备  │ │统计报表  │                    │
│  └──────────┘ └──────────┘ └──────────┘                    │
└─────────────────────────────┬───────────────────────────────┘
                              │ 统一数据模型接口
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│ 飞书招聘适配器 │     │  北森适配器   │     │  Moka适配器   │
│ feishu_hire   │     │    beisen     │     │     moka      │
└───────┬───────┘     └───────┬───────┘     └───────┬───────┘
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│ 飞书开放平台  │     │ 北森开放平台  │     │ Moka开放平台  │
│  hire/v1 API  │     │ RecruitV6 API │     │ api-platform  │
└───────────────┘     └───────────────┘     └───────────────┘
```
### 设计原则
1. **单一职责**：每个适配器只负责一个招聘系统的 API 调用和数据转换
2. **接口统一**：所有适配器实现相同的接口，上层无感知
3. **容错降级**：某系统失败不影响其他系统，整体可用
4. **可扩展**：新增系统只需实现适配器接口
5. **配置驱动**：通过配置决定使用哪些系统、聚合模式
---
## 统一数据模型
### 1. 待办任务模型（Task）
```typescript
interface Task {
  // 基础标识
  task_id: string;           // 系统内唯一ID（系统前缀+原始ID，避免冲突）
  task_type: 'resume_eval' | 'interview' | 'interview_eval';
  source_system: 'feishu_hire' | 'beisen' | 'moka';
  // 候选人信息
  candidate_name: string;    // 候选人姓名
  candidate_id: string;      // 候选人ID
  // 职位信息
  job_title: string;         // 职位名称
  job_id: string;            // 职位ID
  department?: string;       // 部门
  // 状态信息
  stage: string;             // 当前阶段
  status: 'pending' | 'in_progress' | 'overdue';
  priority: 'high' | 'medium' | 'low';
  // 时间信息
  created_at: string;        // 任务创建时间（ISO 8601）
  deadline?: string;         // 截止时间
  interview_time?: string;   // 面试时间（面试类任务）
  is_overdue: boolean;       // 是否超时
  overdue_days?: number;     // 超时时长（天）
  // 链接
  detail_url: string;        // 详情页直达链接
  // 扩展字段
  extra?: Record<string, any>; // 系统特有字段
}
```
### 2. 候选人模型（Candidate）
```typescript
interface Candidate {
  // 基础标识
  candidate_id: string;      // 候选人ID
  source_system: 'feishu_hire' | 'beisen' | 'moka';
  // 基本信息
  name: string;              // 姓名
  phone?: string;            // 电话
  email?: string;            // 邮箱
  gender?: string;           // 性别
  // 教育背景
  education?: string;        // 最高学历
  school?: string;           // 毕业院校
  major?: string;            // 专业
  // 工作经历
  experience_years?: number; // 工作年限
  last_company?: string;     // 最近公司
  last_title?: string;       // 最近职位
  // 其他
  source?: string;           // 来源渠道
  resume_url?: string;       // 简历详情链接
  application_id?: string;   // 申请ID
  job_title?: string;        // 应聘职位
  stage?: string;            // 当前阶段
  // 扩展字段
  extra?: Record<string, any>;
}
```
### 3. 职位模型（Job）
```typescript
interface Job {
  job_id: string;
  job_title: string;
  department?: string;
  headcount?: number;        // 招聘人数
  status?: string;           // 状态
  priority?: 'urgent' | 'normal';
  creator?: string;
  source_system: string;
  detail_url?: string;
  extra?: Record<string, any>;
}
```
### 4. 面试模型（Interview）
```typescript
interface Interview {
  interview_id: string;
  application_id: string;
  candidate_name: string;
  job_title: string;
  interview_time: string;
  interview_type?: string;
  interviewers: string[];    // 面试官列表
  round?: number;            // 面试轮次
  evaluation_status?: 'not_started' | 'in_progress' | 'completed';
  source_system: string;
  detail_url?: string;
  extra?: Record<string, any>;
}
```
---
## 适配器接口规范
所有适配器必须实现以下接口：
```typescript
interface HireSystemAdapter {
  // ========== 基础信息 ==========
  /** 适配器名称 */
  readonly name: string;
  /** 系统标识 */
  readonly systemCode: 'feishu_hire' | 'beisen' | 'moka';
  /** API 域名 */
  readonly apiDomain: string;
  // ========== 认证相关 ==========
  /** 初始化认证（获取token等） */
  authenticate(config: AdapterConfig): Promise<void>;
  /** 检查认证状态 */
  isAuthenticated(): boolean;
  /** 刷新认证 */
  refreshAuth(): Promise<void>;
  // ========== 待办任务 ==========
  /** 获取待评估简历列表 */
  getPendingResumes(options?: QueryOptions): Promise<Task[]>;
  /** 获取待面试列表 */
  getPendingInterviews(options?: QueryOptions): Promise<Task[]>;
  /** 获取待写评估列表 */
  getPendingEvaluations(options?: QueryOptions): Promise<Task[]>;
  /** 获取全部待办（合并以上三类） */
  getAllTasks(options?: QueryOptions): Promise<Task[]>;
  // ========== 候选人 ==========
  /** 获取候选人详情 */
  getCandidate(candidateId: string): Promise<Candidate>;
  /** 批量获取候选人 */
  getCandidates(candidateIds: string[]): Promise<Candidate[]>;
  // ========== 职位 ==========
  /** 获取职位列表 */
  getJobs(options?: QueryOptions): Promise<Job[]>;
  /** 获取职位详情 */
  getJob(jobId: string): Promise<Job>;
  // ========== 面试 ==========
  /** 获取面试详情 */
  getInterview(interviewId: string): Promise<Interview>;
  // ========== 链接生成 ==========
  /** 生成候选人详情链接 */
  getCandidateUrl(candidateId: string): string;
  /** 生成职位详情链接 */
  getJobUrl(jobId: string): string;
  /** 生成面试详情链接 */
  getInterviewUrl(interviewId: string): string;
  // ========== 健康检查 ==========
  /** 健康检查（API连通性） */
  healthCheck(): Promise<boolean>;
}
```
### 查询参数（QueryOptions）
```typescript
interface QueryOptions {
  startTime?: string;        // 开始时间
  endTime?: string;          // 结束时间
  status?: string[];         // 状态筛选
  jobId?: string;            // 职位筛选
  pageSize?: number;         // 每页数量
  maxResults?: number;       // 最大返回数量
  includeDetails?: boolean;  // 是否包含详情
}
```
### 适配器配置（AdapterConfig）
```typescript
interface AdapterConfig {
  // 通用
  systemCode: string;
  webDomain: string;         // Web端域名（用于生成链接）
  // 飞书招聘
  feishuAppId?: string;
  feishuAppSecret?: string;
  feishuAuthMode?: 'user' | 'tenant';
  feishuUserAccessToken?: string;
  feishuTenantAccessToken?: string;
  // 北森
  beisenTenantId?: string;
  beisenAppKey?: string;
  beisenAppSecret?: string;
  beisenUserId?: string;
  // Moka
  mokaClientId?: string;
  mokaClientSecret?: string;
  mokaUserEmail?: string;
  // 通用可选
  timeout?: number;          // 超时时间（毫秒）
  retryCount?: number;       // 重试次数
}
```
---
## 各系统适配方案
### 飞书招聘适配器（feishu_hire）
**认证方式**：App ID/Secret + OAuth（用户身份/应用身份）
**核心接口**：
- 待评估：`GET /hire/v1/applications` + 状态筛选 + 处理人筛选
- 待面试：`GET /hire/v1/interviews` + 面试官筛选 + 时间筛选
- 待写评估：`GET /hire/v1/interviews` + 状态筛选
- 候选人：`GET /hire/v1/talents/{id}`
**特点**：
- 接口规范，文档完善
- 支持用户身份 OAuth，个人视角数据精准
- 没有免登录链接，需拼接普通链接
**详细参考**：[飞书招聘API参考](api-reference-feishu.md)
### 北森适配器（beisen）
**认证方式**：Key/Secret + Bearer Token（企业级）
**核心接口**：
- 待评估：`POST /Apply/GetApplyListByDateAndStatus` + 状态筛选 + handlerId筛选
- 待面试：`GET /Interview/GetInterviewsByDate` + 面试官筛选
- 待写评估：`GET /Interview/GetInterviewsByDate` + 评价状态筛选
- 候选人：`POST /Applicant/BatchGetApplicantInfo`
- 免登录链接：`POST /Apply/GetApplyDetailUrl`
**特点**：
- 提供免登录详情链接，用户体验好
- 接口丰富，功能全面
- 主要是企业级 API，需通过 UserID 映射个人身份
- token 有效期2小时，需自动刷新
**详细参考**：[北森API参考](api-reference-beisen.md)
### Moka适配器（moka）
**认证方式**：Basic Auth（clientId:clientSecret）
**核心接口**：
- 待评估：`GET /data/ehrApplications?stage=preliminary_filter` + 负责人筛选
- 待面试：`GET /data/ehrApplications?stage=interview` + 面试官筛选
- 待写评估：`GET /data/ehrApplications` + 评估状态筛选
- 候选人：列表接口已包含大部分信息
- 免登录链接：`ehrCandidateExternalLink` 字段
**特点**：
- 候选人列表接口信息丰富，一次调用获取大部分字段
- 提供免登录外部链接
- 面试相关接口相对有限
- stage 无候选人时返回"服务器异常"，需特殊容错
- Basic Auth 简单，无需 token 刷新
**详细参考**：[MokaAPI参考](api-reference-moka.md)
---
## 多系统聚合模式
### 模式一：合并排序模式（merge）
将多个系统的待办合并后统一排序展示。
**适用场景**：用户希望看到一个统一的待办列表，按优先级处理。
**展示示例**：
```
📝 全部待办（12份）
━━━━━━━━━━━━━━━━━━━━
[飞书] 🔥 1. 张三 - 前端开发（急招，超时2天）
   👉 [去评估](https://hire.feishu.cn/...)
[Moka] ⭐ 2. 李四 - 产品经理（内推）
   👉 [去评估](https://app.mokahr.com/...)
[北森] 3. 王五 - 数据分析
   👉 [去评估](https://hire.italent.cn/...)
[飞书] 4. 赵六 - 后端开发
   👉 [去评估](https://hire.feishu.cn/...)
...
```
**排序规则**：
1. 优先级（high > medium > low）
2. 是否超时（超时优先）
3. 创建时间（早的优先）
4. 系统优先级（可配置，默认飞书 > 北森 > Moka）
### 模式二：按系统分组模式（group）
按系统分组展示，每组内独立排序。
**适用场景**：用户习惯按系统分别处理，或系统间差异较大。
**展示示例**：
```
📋 飞书招聘待办（5份）
━━━━━━━━━━━━━━━━━━━━
🔥 1. 张三 - 前端开发（急招，超时2天）
   👉 [去评估](https://hire.feishu.cn/...)
2. 赵六 - 后端开发
   👉 [去评估](https://hire.feishu.cn/...)
📋 Moka待办（4份）
━━━━━━━━━━━━━━━━━━━━
⭐ 1. 李四 - 产品经理（内推）
   👉 [去评估](https://app.mokahr.com/...)
📋 北森待办（3份）
━━━━━━━━━━━━━━━━━━━━
1. 王五 - 数据分析
   👉 [去评估](https://hire.italent.cn/...)
```
### 聚合配置
```typescript
interface AggregationConfig {
  mode: 'merge' | 'group';           // 聚合模式
  systems: string[];                 // 启用的系统列表
  systemPriority?: string[];         // 系统优先级（merge模式下同优先级时的排序）
  groupOrder?: string[];             // 分组顺序（group模式）
  failStrategy: 'partial' | 'all';   // 失败策略：partial=部分失败仍返回其他，all=全部失败才报错
}
```
---
## 异常处理与降级策略
### 单系统异常处理
每个适配器内部处理自身的异常：
1. **认证失败**：自动刷新 token，重试1次；仍失败则抛出 `AuthError`
2. **网络错误**：自动重试2次（指数退避）；仍失败则抛出 `NetworkError`
3. **接口限流**：等待后重试，最多3次；仍失败则抛出 `RateLimitError`
4. **数据为空**：返回空数组，不抛异常
5. **单条数据异常**：跳过该条，记录日志，不影响整体
### 多系统聚合异常处理
| 异常场景 | 处理策略 | 用户提示 |
|----------|----------|----------|
| 某系统认证失败 | 跳过该系统，其他系统正常返回 | "⚠️ 北森系统登录已过期，已为你展示飞书招聘和Moka的待办" |
| 某系统网络错误 | 跳过该系统，其他系统正常返回 | "⚠️ Moka系统暂时无法访问，已为你展示其他系统的待办" |
| 某系统接口限流 | 跳过该系统，提示稍后重试 | "⏳ 飞书招聘请求太频繁，该系统数据稍后再试" |
| 全部系统失败 | 整体报错，引导排查 | "⚠️ 所有招聘系统暂时无法访问，请检查网络和配置" |
| 某系统数据为空 | 正常展示，该系统显示"暂无" | 不额外提示 |
### 降级策略
1. **功能降级**：某系统的某类待办获取失败时，只展示能获取到的类型
2. **详情降级**：候选人详情获取失败时，用列表中的基础信息展示
3. **链接降级**：免登录链接获取失败时，用拼接的普通链接（需登录）
4. **字段降级**：某字段缺失时，用"-"或"未知"填充，不报错
---
## 数据缓存策略
### 缓存设计
```typescript
interface CacheEntry<T> {
  data: T;
  timestamp: number;       // 缓存时间
  ttl: number;             // 有效期（毫秒）
  sourceSystem: string;    // 来源系统
}
```
### 缓存规则
| 数据类型 | 缓存有效期 | 说明 |
|----------|-----------|------|
| 待办任务列表 | 5分钟 | 频繁变动，短缓存 |
| 候选人详情 | 1小时 | 相对稳定 |
| 职位列表 | 1小时 | 相对稳定 |
| 面试详情 | 30分钟 | 面试时间可能变动 |
| access_token | 有效期的80% | 提前刷新 |
### 缓存失效
- 用户主动刷新时清除对应缓存
- 定时推送时强制刷新（不使用缓存）
- 写操作后清除相关缓存（本技能为只读，暂不涉及）
---
## 性能优化
### 1. 并行调用
多系统模式下，并行调用各适配器：
```typescript
const results = await Promise.allSettled([
  feishuAdapter.getAllTasks(),
  beisenAdapter.getAllTasks(),
  mokaAdapter.getAllTasks()
]);
```
使用 `Promise.allSettled` 而非 `Promise.all`，确保某系统失败不影响其他。
### 2. 批量接口
优先使用批量接口减少调用次数：
- 北森：`BatchGetApplicantInfo`、`GetApplyDetailUrl`
- 飞书：分页拉取，pageSize 设为最大值
- Moka：limit 设为100，减少分页次数
### 3. 增量拉取
- Moka：使用 `moved_applications` 增量接口
- 北森：按时间范围增量拉取
- 飞书：按更新时间筛选
### 4. 数据预取
- 定时推送前预拉取数据，减少用户等待
- 高优候选人详情预加载
---
## 扩展性设计
### 新增招聘系统的步骤
1. 创建新的适配器类，实现 `HireSystemAdapter` 接口
2. 实现认证、API调用、数据映射、链接生成
3. 在适配器工厂中注册新系统
4. 在配置说明中增加新系统的配置项
5. 新增对应的 API 参考文档
6. 更新能力对比表和文档
**无需修改**：业务逻辑层、输出模板、定时推送、质量校验等
### 适配器工厂
```typescript
class AdapterFactory {
  private static adapters = new Map<string, HireSystemAdapter>();
  static register(systemCode: string, adapter: HireSystemAdapter) {
    this.adapters.set(systemCode, adapter);
  }
  static getAdapter(systemCode: string): HireSystemAdapter {
    const adapter = this.adapters.get(systemCode);
    if (!adapter) throw new Error(`不支持的招聘系统: ${systemCode}`);
    return adapter;
  }
  static getAdapters(systemCodes: string[]): HireSystemAdapter[] {
    return systemCodes.map(code => this.getAdapter(code));
  }
}
```
---
## 安全与合规
### 数据安全
1. **凭证加密存储**：App Secret、Client Secret 等敏感信息加密存储
2. **传输加密**：所有 API 调用使用 HTTPS
3. **最小权限**：只申请只读权限，不申请写权限
4. **数据隔离**：个人数据只存在用户个人空间，其他人不可见
5. **免登录链接**：不持久化，每次生成，避免泄露
### 隐私合规
1. 候选人个人信息（电话、邮箱等）仅在用户明确需要时展示
2. 数据沉淀需用户明确授权
3. 不将候选人数据用于除招聘工作外的其他用途
4. 支持数据删除（用户可清除沉淀的数据）
---
## 测试策略
### 单元测试
- 各适配器的字段映射逻辑
- 优先级排序算法
- 超时计算逻辑
- 异常处理逻辑
### 集成测试
- 各系统 API 连通性测试
- 多系统聚合测试
- 异常降级测试
- 缓存有效性测试
### 验收测试用例
| 测试场景 | 预期结果 |
|----------|----------|
| 单系统（飞书）查询待办 | 返回飞书待办，链接正确 |
| 单系统（北森）查询待办 | 返回北森待办，免登录链接可用 |
| 单系统（Moka）查询待办 | 返回Moka待办，外部链接可用 |
| 多系统 merge 模式 | 合并排序，来源标注正确 |
| 多系统 group 模式 | 按系统分组，每组独立排序 |
| 某系统认证失败 | 其他系统正常返回，失败系统提示 |
| 某系统网络错误 | 其他系统正常返回，失败系统提示 |
| 全部系统失败 | 整体报错，引导排查 |
| 某系统数据为空 | 正常展示，显示"暂无" |
| Moka阶段无候选人 | 友好提示，不报错 |
---
## 版本与兼容性
### 版本策略
- 主版本号：架构变更（如 v3.0.0 引入多系统）
- 次版本号：新增功能（如新增适配器、新增聚合模式）
- 修订号：Bug修复、文档更新
### 向后兼容
- v2.x 配置（仅飞书）在 v3.x 中自动兼容，source_system 默认 feishu_hire
- 旧的输出模板在多系统模式下自动增加来源标注
- 旧的定时推送配置自动兼容
---
## 相关文档
- [飞书招聘API参考](api-reference-feishu.md)
- [北森API参考](api-reference-beisen.md)
- [MokaAPI参考](api-reference-moka.md)
- [输出模板参考](output-templates.md)
- [异常处理与错误提示](error-handling.md)
- [质量校验清单](quality-checklist.md)
