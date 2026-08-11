# 北森（iTalent）招聘 API 参考文档
## 概述
本文档是 TA 工作助手技能中**北森（iTalent）招聘系统适配器**的 API 参考，基于北森开放平台 RecruitV6 接口。
**API 基础信息**：
- **API 域名**：`https://openapi.italent.cn`
- **开放平台地址**：https://open.italent.cn/
- **认证方式**：Key + Secret 获取 access_token，Bearer Token
- **接口限流**：100次/秒/企业，3000次/分钟/企业
- **开通方式**：联系北森项目经理或 CSM 开通接口权限，获取租户 ID 和授权回执单
---
## 认证方式
### 获取 access_token
**请求**：
```
POST /oauth/token
Content-Type: application/x-www-form-urlencoded
grant_type=client_credentials&client_id={app_key}&client_secret={app_secret}
```
**响应**：
```json
{
  "access_token": "xxx",
  "token_type": "Bearer",
  "expires_in": 7200
}
```
### 请求头格式
```
Authorization: Bearer {access_token}
Content-Type: application/json
```
> **注意**：Bearer 和 Token 之间有一个空格。
### Token 刷新策略
- access_token 有效期 2 小时
- 适配器应在 token 过期前自动刷新
- 刷新失败时提示用户检查 Key/Secret 配置
---
## 核心接口清单
### 一、申请（投递）相关
#### 1. 根据搜索条件查询应聘者申请信息
```
POST /RecruitV6/api/v1/Apply/GetApplyListByDateAndStatus
```
**用途**：获取待评估简历列表（核心接口）
**请求参数**：
```json
{
  "startTime": "2026-08-01 00:00:00",
  "endTime": "2026-08-11 23:59:59",
  "status": ["待评估"],
  "handlerId": "{当前用户UserID}",
  "pageIndex": 1,
  "pageSize": 100
}
```
**说明**：
- 每批次最多返回 1000 条数据
- 通过 `status` 筛选"待评估"阶段
- 通过 `handlerId` 筛选当前用户负责的申请
- 支持按时间范围增量拉取
**响应关键字段**：
```json
{
  "applyId": "申请ID",
  "applicantId": "应聘者ID",
  "applicantName": "应聘者姓名",
  "jobId": "职位ID",
  "jobName": "职位名称",
  "status": "当前状态",
  "stage": "当前阶段",
  "createTime": "投递时间",
  "handlerId": "处理人ID",
  "source": "来源渠道"
}
```
#### 2. 根据申请ID获取申请信息
```
GET /RecruitV6/api/v1/Apply/GetApplyById?applyId={applyId}
```
**用途**：获取单条申请的详细信息
#### 3. 获取应聘者申请对象信息
```
GET /RecruitV6/api/v1/Apply/GetApplyObject?applyId={applyId}
```
#### 4. 根据申请Id批量获取申请详情页免登录地址
```
POST /RecruitV6/api/v1/Apply/GetApplyDetailUrl
```
**用途**：生成免登录的详情页链接（核心接口，用于输出直达链接）
**请求参数**：
```json
{
  "applyIds": ["applyId1", "applyId2"]
}
```
**响应**：
```json
{
  "applyId": "申请ID",
  "detailUrl": "https://hire.xxx.com/...?token=xxx"
}
```
> **说明**：免登录链接有有效期，建议每次生成新的，不要缓存。
---
### 二、应聘者相关
#### 1. 批量获取应聘者个人信息
```
POST /RecruitV6/api/v1/Applicant/BatchGetApplicantInfo
```
**用途**：批量获取候选人基本信息
**请求参数**：
```json
{
  "applicantIds": ["id1", "id2", "id3"]
}
```
**响应关键字段**：
```json
{
  "applicantId": "应聘者ID",
  "name": "姓名",
  "gender": "性别",
  "phone": "电话",
  "email": "邮箱",
  "education": "最高学历",
  "school": "毕业院校",
  "major": "专业",
  "workYears": "工作年限"
}
```
#### 2. 批量获取简历模块信息
```
POST /RecruitV6/api/v1/Applicant/BatchGetResumeModules
```
**用途**：获取工作经历、教育经历、项目经历等简历模块
#### 3. 获取标准简历
```
GET /RecruitV6/api/v1/Applicant/GetStandardResume?applicantId={applicantId}
```
**用途**：获取结构化的标准简历内容
#### 4. 获取应聘者原始简历文件
```
GET /RecruitV6/api/v1/Applicant/GetOriginalResumeFile?applicantId={applicantId}
```
**用途**：下载原始简历文件（PDF/Word）
---
### 三、面试相关
#### 1. 根据面试时间获取面试
```
GET /RecruitV6/api/v1/Interview/GetInterviewsByDate?startDate={start}&endDate={end}
```
**用途**：获取指定时间范围内的面试列表（核心接口，用于待面试和待写评估）
**筛选逻辑**：
- 待面试：面试时间 > 当前时间 + 面试官包含当前用户
- 待写评估：面试时间 < 当前时间 + 评价状态 = 未评价 + 面试官包含当前用户
**响应关键字段**：
```json
{
  "interviewId": "面试ID",
  "applyId": "申请ID",
  "applicantName": "应聘者姓名",
  "jobName": "职位名称",
  "interviewTime": "面试时间",
  "interviewType": "面试类型",
  "interviewers": ["面试官ID列表"],
  "evaluationStatus": "评价状态（未评价/已评价）",
  "round": "面试轮次"
}
```
#### 2. 根据面试ID获取面试信息
```
GET /RecruitV6/api/v1/Interview/GetInterviewById?interviewId={interviewId}
```
#### 3. 根据申请ID获取面试信息
```
GET /RecruitV6/api/v1/Interview/GetInterviewsByApplyId?applyId={applyId}
```
#### 4. 根据面试ID获取评价详情
```
GET /RecruitV6/api/v1/Interview/GetEvaluationDetailById?interviewId={interviewId}
```
**用途**：获取已填写的面试评价内容
---
### 四、职位相关
#### 1. 根据条件获取职位列表
```
POST /RecruitV6/api/v1/Job/GetJobListByCondition
```
**用途**：获取职位列表，用于岗位维度统计
**请求参数**：
```json
{
  "status": ["招聘中"],
  "pageIndex": 1,
  "pageSize": 100
}
```
**响应关键字段**：
```json
{
  "jobId": "职位ID",
  "jobName": "职位名称",
  "department": "部门",
  "headcount": "招聘人数",
  "status": "状态",
  "priority": "优先级（急招/普通）",
  "creator": "创建人"
}
```
#### 2. 根据职位Id获取职位列表
```
POST /RecruitV6/api/v1/Job/GetJobListByIds
```
---
### 五、其他接口
#### 招聘需求
```
POST /RecruitV6/api/v1/JobRequirement/GetListByCondition
```
#### Offer信息
```
POST /RecruitV6/api/v1/Offer/GetOfferInfo
```
#### 待入职人员
```
POST /RecruitV6/api/v1/Onboarding/GetPendingCheckinList
```
#### 操作记录历史
```
POST /RecruitV6/api/v1/OperationRecord/GetAllHistory
```
**用途**：获取候选人的全部操作记录，用于流程追踪和时效分析
---
## 待办任务实现逻辑
### 待评估简历
```
1. 调用 GetApplyListByDateAndStatus
   - status = ["待评估"]
   - handlerId = 当前用户UserID
   - 时间范围：最近30天（可配置）
2. 分页拉取全部数据
3. 按创建时间排序
4. 调用 GetApplyDetailUrl 批量生成免登录链接
5. 映射到统一待办模型
```
### 待面试
```
1. 调用 GetInterviewsByDate
   - startDate = 今天
   - endDate = 今天+7天（可配置）
2. 筛选 interviewers 包含当前用户UserID
3. 筛选面试时间 > 当前时间
4. 关联申请信息获取候选人姓名、职位
5. 生成面试详情链接
6. 映射到统一待办模型
```
### 待写评估
```
1. 调用 GetInterviewsByDate
   - startDate = 今天-30天（可配置）
   - endDate = 今天
2. 筛选 interviewers 包含当前用户UserID
3. 筛选 evaluationStatus = "未评价"
4. 关联申请信息获取候选人姓名、职位
5. 计算超时时长（面试结束时间到现在）
6. 生成评估提交页链接
7. 映射到统一待办模型
```
---
## 字段映射表（北森 → 统一模型）
### 待办任务映射
| 统一模型字段 | 北森字段 | 说明 |
|-------------|---------|------|
| task_id | applyId / interviewId | 申请ID或面试ID |
| task_type | 固定值 | resume_eval / interview / interview_eval |
| candidate_name | applicantName | 应聘者姓名 |
| job_title | jobName | 职位名称 |
| job_id | jobId | 职位ID |
| stage | status / stage | 当前状态/阶段 |
| priority | 派生 | 根据职位优先级、是否超时等计算 |
| created_at | createTime / interviewTime | 创建时间/面试时间 |
| deadline | 派生 | 创建时间+超时阈值 |
| is_overdue | 派生 | 当前时间 > deadline |
| overdue_days | 派生 | (当前时间 - deadline) 的天数 |
| detail_url | GetApplyDetailUrl返回 | 免登录详情链接 |
| source_system | 固定值 "beisen" | 来源系统 |
### 候选人映射
| 统一模型字段 | 北森字段 | 说明 |
|-------------|---------|------|
| candidate_id | applicantId | 应聘者ID |
| name | name | 姓名 |
| phone | phone | 电话 |
| email | email | 邮箱 |
| gender | gender | 性别 |
| education | education | 最高学历 |
| school | school | 毕业院校 |
| major | major | 专业 |
| experience_years | workYears | 工作年限 |
| last_company | 工作经历模块 | 最近公司 |
| last_title | 工作经历模块 | 最近职位 |
| source | source | 来源渠道 |
| resume_url | 派生 | 简历详情页链接 |
| source_system | 固定值 "beisen" | 来源系统 |
---
## 链接生成规则
### 申请详情链接
优先使用 `GetApplyDetailUrl` 接口返回的免登录链接。
如果接口不可用，使用拼接方式：
```
https://{beisen_web_domain}/recruit/apply/detail/{applyId}
```
> `beisen_web_domain` 为企业实际的北森招聘系统访问域名，需在配置中填写。
### 面试详情链接
```
https://{beisen_web_domain}/recruit/interview/detail/{interviewId}
```
### 评估提交链接
```
https://{beisen_web_domain}/recruit/interview/evaluation/{interviewId}
```
### 候选人详情链接
```
https://{beisen_web_domain}/recruit/applicant/detail/{applicantId}
```
### 职位详情链接
```
https://{beisen_web_domain}/recruit/job/detail/{jobId}
```
---
## 错误码说明
| 错误码 | 说明 | 处理方式 |
|--------|------|----------|
| 401 | access_token 无效或过期 | 自动刷新 token，重试 |
| 403 | 权限不足 | 提示联系北森管理员开通接口权限 |
| 404 | 资源不存在 | 跳过该条数据，提示用户 |
| 429 | 请求限流 | 等待后重试，降低请求频率 |
| 500 | 服务器内部错误 | 重试2次，仍失败则提示 |
| 1001 | 参数错误 | 检查请求参数格式 |
| 2001 | 应用未授权 | 检查 Key/Secret 配置 |
---
## 最佳实践
### 1. 数据拉取策略
- 待评估简历：增量拉取，按时间范围，每次拉取最近30天
- 待面试：拉取未来7天的面试
- 待写评估：拉取最近30天已完成但未评价的面试
- 候选人详情：批量获取，减少接口调用次数
### 2. 错误处理策略
- token 过期自动刷新，不打扰用户
- 单条数据获取失败时跳过，不影响整体
- 接口限流时自动退避重试
- 免登录链接每次重新生成，不缓存
### 3. 性能优化
- 批量接口优先（BatchGetApplicantInfo、GetApplyDetailUrl）
- 合理设置分页大小，减少请求次数
- 候选人信息缓存，有效期1小时
### 4. 数据安全
- access_token 加密存储
- 免登录链接不持久化，每次生成
- 个人数据只在用户个人空间存储
---
## 开通流程
1. 联系北森项目经理或 CSM，说明需要开通 OpenAPI 接口权限
2. 获取授权回执单，包含：租户 ID、应用 Key、应用 Secret
3. 登录北森管理后台 → 账户中心 → 应用开发 >> OpenAPI连接器 → 创建连接器
4. 配置 IP 白名单（如有需要）
5. 在 TA 工作助手配置中填写：beisen_tenant_id、beisen_app_key、beisen_app_secret、beisen_web_domain、beisen_user_id
---
## 相关文档
- [北森开放平台官网](https://open.italent.cn/)
- [北森招聘API文档](https://open.italent.cn/doc)
- [多系统适配器设计文档](adapter-design.md)
- [飞书招聘API参考](api-reference-feishu.md)
- [MokaAPI参考](api-reference-moka.md)
