# API 接口参考

本文件列出 TA 工作助手使用的飞书开放平台招聘 API 接口详情，供开发和排查问题参考。

---

## 接口总览

| 模块 | 接口 | 方法 | 用途 | 必需权限 |
|------|------|------|------|----------|
| 投递 | 获取投递列表 | GET | 拉取待评估/已评估简历 | `hire:application:readonly` |
| 投递 | 获取投递详情 | GET | 查看单个投递详情 | `hire:application:readonly` |
| 职位 | 获取职位列表 | GET | 岗位信息查询 | `hire:job:readonly` |
| 职位 | 获取职位详情 | GET | 单个岗位详情 | `hire:job.composite_info:readonly` |
| 人才 | 获取人才详情 | GET | 候选人基本信息 | `hire:talent:readonly` |
| 人才 | 获取简历信息 | GET | 简历详细内容 | `hire:talent:readonly` |
| 面试 | 获取面试列表 | GET | 面试安排查询 | `hire:interview:readonly` |
| 面试 | 获取面试详情 | GET | 单个面试详情 | `hire:interview:readonly` |
| 招聘需求 | 获取需求列表 | GET | 招聘需求查询 | `hire:job_requirement:readonly` |

---

## 1. 投递管理

### 1.1 获取投递列表

**接口地址**：`GET /open-apis/hire/v1/applications`

**用途**：拉取投递列表，用于待评估简历、已评估简历查询。

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `page_size` | int | 否 | 分页大小，默认20，最大100 |
| `page_token` | string | 否 | 分页标记，第一次请求不填 |
| `job_id` | string | 否 | 按职位ID筛选 |
| `stage_id` | string | 否 | 按阶段ID筛选 |
| `talent_id` | string | 否 | 按人才ID筛选 |
| `application_status` | int | 否 | 投递状态：1=待评估，2=已通过，3=已拒绝 |

**响应示例**：
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "items": [
      {
        "id": "application_id_123",
        "talent_id": "talent_id_456",
        "job_id": "job_id_789",
        "job_name": "高级前端开发工程师",
        "stage": {
          "id": "stage_id",
          "name": "简历评估"
        },
        "application_status": 1,
        "create_time": "1620000000000",
        "update_time": "1620000000000"
      }
    ],
    "page_token": "next_page_token",
    "has_more": true
  }
}
```

**个人待办筛选逻辑**：
- 用户身份模式：API自动按用户权限过滤，返回用户有权限的投递
- 应用身份模式：需要额外按"面试官/评估人ID = 当前用户"筛选

---

### 1.2 获取投递详情

**接口地址**：`GET /open-apis/hire/v1/applications/{application_id}`

**用途**：获取单个投递的详细信息。

**路径参数**：
- `application_id`：投递ID

**查询参数**：
- `include_detail`：是否包含详细信息，默认true

---

## 2. 职位管理

### 2.1 获取职位列表

**接口地址**：`GET /open-apis/hire/v1/jobs`

**用途**：获取职位列表，用于岗位分布统计、岗位筛选。

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `page_size` | int | 否 | 分页大小 |
| `page_token` | string | 否 | 分页标记 |
| `job_status` | int | 否 | 职位状态：1=招聘中，2=已关闭 |
| `keyword` | string | 否 | 关键词搜索 |

---

### 2.2 获取职位详情

**接口地址**：`GET /open-apis/hire/v1/jobs/{job_id}`

**用途**：获取单个职位的详细信息，包括JD、招聘要求等。

**用于**：
- 自动从JD提取筛选标准
- 岗位招聘进度统计
- 急招岗位识别

---

## 3. 人才管理

### 3.1 获取人才详情

**接口地址**：`GET /open-apis/hire/v1/talents/{talent_id}`

**用途**：获取候选人基本信息。

**返回字段**：
- 姓名、性别、联系方式
- 学历、工作年限
- 当前公司、当前职位
- 期望薪资、期望城市
- 人才标签、来源渠道

---

### 3.2 获取简历信息

**接口地址**：`GET /open-apis/hire/v1/talents/{talent_id}/resumes`

**用途**：获取候选人的简历详细内容。

**返回字段**：
- 教育经历
- 工作经历
- 项目经历
- 技能标签
- 自我评价

---

## 4. 面试管理

### 4.1 获取面试列表

**接口地址**：`GET /open-apis/hire/v1/interviews`

**用途**：获取面试列表，用于待面试、已面试查询。

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `page_size` | int | 否 | 分页大小 |
| `page_token` | string | 否 | 分页标记 |
| `interviewer_id` | string | 否 | 按面试官ID筛选 |
| `start_time_from` | string | 否 | 面试开始时间（起始） |
| `start_time_to` | string | 否 | 面试开始时间（结束） |
| `interview_status` | int | 否 | 面试状态：1=未开始，2=进行中，3=已结束 |

**个人待办筛选逻辑**：
- 传入 `interviewer_id = 当前用户ID`
- 按状态筛选：未开始=待面试，已结束未评价=待写评估

---

### 4.2 获取面试详情

**接口地址**：`GET /open-apis/hire/v1/interviews/{interview_id}`

**用途**：获取单个面试的详细信息。

**返回字段**：
- 面试时间、地点、方式
- 面试官列表
- 候选人信息
- 面试轮次
- 面试评价（如有）

---

## 5. 招聘需求

### 5.1 获取招聘需求列表

**接口地址**：`GET /open-apis/hire/v1/job_requirements`

**用途**：获取招聘需求列表，用于招聘进度统计。

---

## 6. 认证方式

### 6.1 用户身份（推荐个人待办使用）

**OAuth 2.0 授权码模式**：

1. 用户点击授权链接，跳转到飞书授权页
2. 用户确认授权，回调获取 authorization_code
3. 用 code 换取 user_access_token
4. 用 user_access_token 调用 API

**特点**：
- 数据范围：跟用户本人权限一致
- 安全性：只能看用户自己能看的数据
- 适合：个人待办、个人视角

### 6.2 应用身份（适合企业级场景）

**tenant_access_token**：

1. 用 app_id + app_secret 换取 tenant_access_token
2. 用 tenant_access_token 调用 API

**特点**：
- 数据范围：应用权限范围内的全租户数据
- 适合：管理员视角、企业级报表
- 注意：权限较大，需要谨慎使用

---

## 7. 错误码说明

| 错误码 | 说明 | 处理方式 |
|--------|------|----------|
| 0 | 成功 | - |
| 99991663 | 权限不足 | 提示用户申请对应scope权限 |
| 99991661 | token无效 | 重新获取token |
| 99991668 | token过期 | 刷新token |
| 99991001 | 参数错误 | 检查请求参数 |
| 99991400 | 请求太频繁 | 限流，稍后重试 |
| 99991500 | 系统错误 | 稍后重试，持续失败联系管理员 |

---

## 8. 限流说明

| 接口 | 限流策略 |
|------|----------|
| 所有接口 | 100次/分钟（应用维度） |
| 单用户维度 | 20次/分钟 |

**超限处理**：
- 返回错误码 99991400
- 自动退避重试（等待1秒、2秒、4秒...）
- 提示用户"请求太频繁，请稍后再试"

---

## 9. 链接生成规则

根据投递/面试/人才ID，生成对应的飞书招聘页面链接：

| 页面类型 | URL 格式 |
|----------|----------|
| 人才详情 | `{domain}/talent/{talent_id}?application_id={application_id}` |
| 职位详情 | `{domain}/job/{job_id}` |
| 面试详情 | `{domain}/interview/{interview_id}` |
| 投递详情 | `{domain}/application/{application_id}` |

其中 `{domain}` 根据企业配置：
- 国内版：`https://hire.feishu.cn`
- 国际版：`https://hire.larksuite.com`

---

## 10. 最佳实践

### 数据拉取策略

1. **增量拉取**：优先用 update_time 做增量，减少全量拉取
2. **分页处理**：大列表必须分页处理，每页不超过50条
3. **并发控制**：API调用并发不超过3个，避免触发限流
4. **缓存机制**：
   - 职位信息、人才基本信息：缓存1小时
   - 待办列表：缓存5分钟
   - 面试详情：缓存30分钟

### 错误处理策略

1. **可重试错误**（网络错误、限流、5xx）：自动重试2次，指数退避
2. **不可重试错误**（参数错误、权限不足）：直接提示用户
3. **部分失败**：列表中个别项失败不影响整体，跳过失败项继续处理

### 数据安全

1. 不在日志中打印完整的候选人敏感信息
2. token 加密存储，不明文保存
3. 遵循最小权限原则，只申请必要的scope
4. 用户数据只在用户会话内使用，不跨用户共享
