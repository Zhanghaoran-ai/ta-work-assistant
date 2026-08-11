# Moka 招聘 API 参考文档
## 概述
本文档是 TA 工作助手技能中 **Moka 招聘系统适配器**的 API 参考，基于 Moka 开放平台 api-platform/v1 接口。
**API 基础信息**：
- **API 域名**：`https://api.mokahr.com/api-platform/v1`
- **测试环境**：`https://api-staging-3.mokahr.com/api-platform/v1`
- **开放平台文档**：https://www.mokahr.com/docs/api/
- **认证方式**：Basic Auth（clientId:clientSecret）
- **分页方式**：使用 `next` 参数分页，每次最多返回 20-100 条
- **开通方式**：在 Moka 开放平台申请应用，获取 Client ID 和 Client Secret
---
## 认证方式
### Basic Auth 认证
将 `clientId:clientSecret` 进行 Base64 编码，放在 Authorization 请求头中。
```
Authorization: Basic base64(clientId:clientSecret)
```
**示例**：
```python
import base64
auth = base64.b64encode(f"{client_id}:{client_secret}".encode()).decode()
headers = {"Authorization": f"Basic {auth}"}
```
### 部分开放平台接口（Bearer + sign）
部分接口使用 Bearer Token + sign 签名方式：
```
sign = sha1(clientId + clientSecret + appId + time)
```
> 本技能主要使用 v1 数据接口，采用 Basic Auth 方式。
---
## 核心接口清单
### 一、候选人/投递相关
#### 1. 获取候选人信息（核心接口）
```
GET /api-platform/v1/data/ehrApplications
```
**用途**：获取候选人列表，用于待评估简历、待面试、待写评估等所有待办场景
**请求参数**：
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| applicationId | string | 条件必填 | 候选人申请ID，逗号分隔多个 |
| stage | string | 条件必填 | 阶段筛选：all/preliminary_filter/offer/pending_checkin/filter/interview/exam/none/shigong |
| email | string | ❌ | 邮箱筛选 |
| phone | string | ❌ | 电话筛选 |
| movedAtStartTime | string | ❌ | 阶段移动开始时间 |
| movedAtEndTime | string | ❌ | 阶段移动结束时间 |
| limit | int | ❌ | 每页数量，默认20，最大100 |
| order | string | ❌ | 排序方式 |
| next | string | ❌ | 分页游标 |
| invitationUpdateStatus | string | ❌ | 邀请更新状态 |
> **注意**：`applicationId` 和 `stage` 参数必须传其一；若查询 stage 阶段没有候选人会提示服务器异常（需友好处理）。
**响应关键字段**：
```json
{
  "applicationId": 47151744,
  "candidateId": 28739723,
  "name": "张三",
  "phone": "138xxxx",
  "email": "xxx@163.com",
  "gender": "男",
  "source": "智联",
  "sourceType": 1,
  "academicDegree": "本科",
  "lastSpeciality": "人力资源管理",
  "lastCompany": "郑州xxx公司",
  "stageName": "沟通offer",
  "stageType": "101",
  "job": {
    "title": "test",
    "department": "xxx",
    "jobId": "ea932057-xxx"
  },
  "jobManager": {
    "name": "安涛",
    "email": "antao@mokahr.com",
    "employeeId": "01"
  },
  "resumeUrl": "",
  "educationInfo": [...],
  "experienceInfo": [...],
  "ehrCandidateExternalLink": "https://cdn.mokahr.com/forward/candidate/info?access_token=xxx"
}
```
**stageType 说明**：
| stageType | 含义 |
|-----------|------|
| 100 | 初筛 |
| 101 | Offer型 |
| 102 | 待入职 |
| 200 | 筛选型 |
| 201 | 面试型 |
| 202 | 测试型 |
| 205 | 无类型 |
| 206 | 试工型 |
#### 2. 获取移动到某阶段的候选人（增量拉取）
```
GET /api-platform/v1/data/moved_applications
```
**用途**：增量获取阶段变动的候选人，适合定时推送场景
**请求参数**：
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| stage | string | ✅ | 阶段 |
| fromTime | string | ✅ | 起始时间 |
| next | string | ❌ | 分页游标 |
| reqType | string | ❌ | all全量/update增量 |
| limit | int | ❌ | 每页数量 |
**说明**：首次调用返回指定阶段所有候选人并标记，后续只返回新增/变动的。
#### 3. 获取候选人信息 v2
```
POST /api-platform/v2/data/ehrApplications
```
**用途**：v2版本候选人接口，支持更多筛选条件
**请求参数**：applicationIds、email、phone、limit、next 等
---
### 二、面试相关
#### 1. 取消面试
```
POST /api-platform/v1/open-platform/interview/cancel
```
**请求参数**：appId、typeId、hireMode、groupId、interviewIds、applicationIds
> 本技能为只读模式，不调用写操作接口，仅作参考。
#### 2. 面试安排与面试官管理
面试安排、面试官管理等接口在 Moka 开放平台模块中，需额外申请权限。
> **说明**：Moka 的面试相关接口相对有限，待面试和待写评估主要通过候选人阶段 + 面试官字段间接筛选。
---
### 三、其他接口
#### 1. 结果回传
```
POST /api-platform/v1/open-platform/survey/result
```
#### 2. 回传候选人信息
```
POST /api-platform/v1/open-platform/survey/getCandidateInfo
```
#### 3. 同步人事信息
```
POST /api-platform/v2/users/syncInfo
```
---
## 待办任务实现逻辑
### 待评估简历
```
1. 调用 GET /data/ehrApplications
   - stage = preliminary_filter（初筛阶段）
   - limit = 100
2. 分页拉取全部数据（使用 next 游标）
3. 筛选 jobManager.email = 当前用户邮箱（或 employeeId 匹配）
4. 按阶段移动时间排序
5. 使用 ehrCandidateExternalLink 字段作为详情链接
6. 映射到统一待办模型
```
> **说明**：Moka 没有单独的"待评估"状态，通过 stage=preliminary_filter + 负责人筛选来实现。
### 待面试
```
1. 调用 GET /data/ehrApplications
   - stage = interview（面试阶段）
   - limit = 100
2. 分页拉取全部数据
3. 筛选面试官包含当前用户（需通过面试相关接口或候选人字段判断）
4. 筛选面试时间 > 当前时间（如接口返回面试时间）
5. 使用 ehrCandidateExternalLink 作为详情链接
6. 映射到统一待办模型
```
> **说明**：Moka 的面试时间信息需要通过额外接口获取，如不可用则按阶段筛选并提示用户。
### 待写评估
```
1. 调用 GET /data/ehrApplications
   - stage = interview（面试阶段）或已完成面试的阶段
   - limit = 100
2. 分页拉取全部数据
3. 筛选面试官包含当前用户
4. 筛选评估状态 = 未写（如接口返回评估状态）
5. 计算超时时长（面试结束时间到现在）
6. 使用 ehrCandidateExternalLink 作为详情链接
7. 映射到统一待办模型
```
> **说明**：Moka 的评估状态字段可能需要通过面试详情接口获取，如不可用则按阶段筛选并提示用户补充。
---
## 字段映射表（Moka → 统一模型）
### 待办任务映射
| 统一模型字段 | Moka字段 | 说明 |
|-------------|---------|------|
| task_id | applicationId | 申请ID |
| task_type | 派生 | 根据stage判断 |
| candidate_name | name | 候选人姓名 |
| job_title | job.title | 职位名称 |
| job_id | job.jobId | 职位ID |
| stage | stageName / stageType | 当前阶段名称/类型 |
| priority | 派生 | 根据职位、是否超时等计算 |
| created_at | movedAt / 投递时间 | 阶段移动时间 |
| deadline | 派生 | 创建时间+超时阈值 |
| is_overdue | 派生 | 当前时间 > deadline |
| overdue_days | 派生 | (当前时间 - deadline) 的天数 |
| detail_url | ehrCandidateExternalLink | 候选人外部链接（免登录） |
| source_system | 固定值 "moka" | 来源系统 |
### 候选人映射
| 统一模型字段 | Moka字段 | 说明 |
|-------------|---------|------|
| candidate_id | candidateId | 候选人ID |
| name | name | 姓名 |
| phone | phone | 电话 |
| email | email | 邮箱 |
| gender | gender | 性别 |
| education | academicDegree | 最高学历 |
| school | educationInfo[0].school | 毕业院校（取最高学历） |
| major | lastSpeciality | 专业 |
| experience_years | 派生 | 根据工作经历计算 |
| last_company | lastCompany | 最近公司 |
| last_title | experienceInfo[0].title | 最近职位 |
| source | source | 来源渠道 |
| resume_url | resumeUrl / ehrCandidateExternalLink | 简历链接 |
| source_system | 固定值 "moka" | 来源系统 |
---
## 链接生成规则
### 候选人详情链接
优先使用接口返回的 `ehrCandidateExternalLink` 字段（免登录外部链接）。
如果该字段为空，使用拼接方式：
```
https://{moka_web_domain}/candidate/detail/{candidateId}
```
> `moka_web_domain` 为企业实际的 Moka 招聘系统访问域名，需在配置中填写。
### 申请/投递详情链接
```
https://{moka_web_domain}/application/detail/{applicationId}
```
### 面试详情链接
```
https://{moka_web_domain}/interview/detail/{interviewId}
```
### 职位详情链接
```
https://{moka_web_domain}/job/detail/{jobId}
```
---
## 错误码说明
| HTTP状态码 | 说明 | 处理方式 |
|-----------|------|----------|
| 401 | 认证失败 | 检查 Client ID/Secret 是否正确 |
| 403 | 权限不足 | 提示联系 Moka 管理员开通接口权限 |
| 404 | 资源不存在 | 跳过该条数据 |
| 429 | 请求限流 | 等待后重试，降低请求频率 |
| 500 | 服务器内部错误 | 重试2次，仍失败则提示 |
| 500（阶段无候选人） | 查询的stage阶段没有候选人 | 友好提示"该阶段暂无候选人"，不报错 |
### 常见业务错误
| 错误信息 | 说明 | 处理方式 |
|----------|------|----------|
| "服务器异常" | 查询阶段无候选人时可能返回 | 捕获后返回空列表，不报错 |
| "invalid client" | Client ID 无效 | 检查配置 |
| "invalid signature" | 签名错误（Bearer+sign方式） | 检查签名算法 |
---
## 最佳实践
### 1. 数据拉取策略
- 候选人列表：按 stage 分阶段拉取，每个阶段单独调用
- 增量拉取：使用 moved_applications 接口，只拉取变动数据
- 分页：使用 next 游标，直到返回为空
- 候选人详情：列表接口已包含大部分信息，尽量减少额外调用
### 2. 错误处理策略
- "服务器异常"错误需特殊处理，可能是阶段无候选人，返回空列表即可
- Basic Auth 认证失败时，提示检查 Client ID/Secret
- 单条数据字段缺失时，用空值填充，不影响整体
- 外部链接为空时，用拼接方式生成普通链接（需要登录）
### 3. 性能优化
- 按 stage 并行拉取不同阶段的候选人
- limit 设为最大值 100，减少请求次数
- 候选人信息缓存，有效期1小时
- 增量拉取优先，减少全量拉取频率
### 4. 数据安全
- Client Secret 加密存储
- 免登录外部链接（ehrCandidateExternalLink）不持久化
- 个人数据只在用户个人空间存储
---
## 开通流程
1. 访问 Moka 开放平台：https://www.mokahr.com/docs/api/
2. 申请开发者账号，创建应用
3. 获取 Client ID 和 Client Secret
4. 申请数据接口权限（候选人、面试等）
5. 配置 IP 白名单（如有需要）
6. 在 TA 工作助手配置中填写：moka_client_id、moka_client_secret、moka_web_domain、moka_user_email
---
## Moka 系统特点与适配注意事项
### 优势
1. 候选人列表接口信息丰富，一次调用可获取大部分字段
2. 提供免登录外部链接（ehrCandidateExternalLink），用户体验好
3. 支持增量拉取（moved_applications），适合定时推送
4. Basic Auth 认证简单，无需维护 token 刷新
### 限制
1. 没有直接的"我的待办"接口，需通过阶段+负责人筛选
2. 面试相关接口相对有限，面试时间和评估状态可能需要额外处理
3. stage 阶段无候选人时返回"服务器异常"，需要特殊容错
4. 主要是企业级 API，用户身份需通过邮箱/employeeId 映射
### 适配建议
1. 待办筛选以 stage + jobManager.email 为主
2. 面试时间和评估状态如接口不提供，在输出中说明"需登录系统查看详情"
3. 对"服务器异常"错误做特殊处理，返回空列表
4. 优先使用 ehrCandidateExternalLink 作为详情链接
---
## 相关文档
- [Moka开放平台文档](https://www.mokahr.com/docs/api/)
- [多系统适配器设计文档](adapter-design.md)
- [飞书招聘API参考](api-reference-feishu.md)
- [北森API参考](api-reference-beisen.md)
