# 体检智能分析系统

提供体检报告解读和套餐推荐服务的API

**版本**: 1.0.0

## 在线访问

- **API文档**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html)
- **OpenAPI规范**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.json](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.json)

## 快速开始

### 基础URL
```
http://localhost:8000
```

### 工作流程

1. **上传PDF文件** (可选，体检报告解读需要)
2. **提交健康问卷** (获得任务UUID)
3. **查询任务结果**
4. **企业端生成套餐** (个性化推荐完成后)

## 主要接口

### 1. 文件上传
```bash
POST /uploads
Content-Type: multipart/form-data

# 表单字段:
# user_id: 用户ID
# file: PDF文件

# 响应:
{
  "message": "文件 report.pdf 上传成功（已覆盖之前的文件）",
  "file_path": "uploads/user001/user001_1704067200.pdf",
  "upload_time": "2025-01-01 12:00:00"
}
```

### 2. 健康检查
```bash
GET /health

# 响应:
{
  "status": "healthy",
  "service": "体检智能分析系统",
  "timestamp": "2025-01-01 12:00:00"
}
```

### 3. 提交健康问卷
```bash
POST /health_report
Content-Type: application/json

{
  "action": "interpret",  # 或 "recommend"
  "name": "张三",
  "title": "体检报告解读",
  "user_id": "user001",  # 体检报告解读时必需
  "company_code": "company_abc",  # 个性化推荐时必需
  "gender": "男",
  "age": 35,
  "pulse": 72,
  "waistline": 85,
  "sbp": 125,
  "dbp": 80,
  "symptoms": ["头痛", "失眠"],
  "medical_history": ["高血压"],
  "surgery_history": "阑尾切除术（2015年）",
  "previous_exam_abnormal": ["血压偏高", "血脂异常"],
  # ... 其他健康问卷字段
}

# 响应:
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "ai_results": "健康问卷已成功提交，智能体正在解读体检报告。",
  "ai_status": "PENDING"
}
```

### 4. 获取体检报告解读结果
```bash
GET /interpret_result/{uuid}

# 响应:
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "ai_status": "DONE",
  "ai_results": "# 体检报告解读结果\n\n基于您的体检报告和问卷信息..."
}
```

### 5. 获取个性化推荐结果
```bash
GET /recommend_result/{uuid}

# 响应:
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "ai_status": "WAITING",  # 或 "DONE"
  "ai_results": "个性化推荐已完成，等待企业管理员生成体检套餐。"
}
```

### 6. 企业端生成套餐
```bash
POST /companies/{company_code}/packages/generate

# 响应:
{
  "package_results": "套餐生成任务已启动，正在制定中…",
  "package_status": "PROCESSING"
}
```

### 7. 查询企业套餐状态
```bash
GET /companies/{company_code}/packages/status

# 响应:
{
  "package_status": "DONE",
  "package_results": "# 企业体检套餐方案\n\n## 男性套餐\n\n### 基础套餐\n- **适用人群：** 20人\n- **预估费用：** ¥800.00\n..."
}
```

### 8. 企业健康数据看板
```bash
GET /companies/{company_code}/dashboard

# 响应:
{
  "health_data": "# 企业员工健康分析报告（2025年度）\n\n## 数据概览\n- 在职员工总数：待统计\n..."
}
```

### 9. 企业套餐对比分析
```bash
GET /companies/{company_code}/packages/compare

# 响应:
{
  "compare_analyse": "# 体检套餐对比分析报告\n\n## 2025年与2024年对比概览\n- 整体套餐费用：待分析\n..."
}
```

### 10. 查询用户当前任务
```bash
GET /users/{user_id}/current_task

# 响应:
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "ai_status": "DONE",
  "ai_results": "# 体检报告解读结果...",
  "created_at": "2025-01-01 12:00:00",
  "action": "interpret"
}

# 如果无任务记录:
{
  "message": "用户暂无任务记录"
}
```

### 11. 查询用户上传状态
```bash
GET /users/{user_id}/upload_status

# 响应:
{
  "has_file": true,
  "filename": "user001_1704067200.pdf",
  "upload_time": "2025-01-01 12:00:00",
  "file_size": 2048576
}

# 如果无文件:
{
  "has_file": false,
  "message": "用户尚未上传PDF文件"
}
```

## 状态码说明

| 状态 | 说明 |
|------|------|
| `PENDING` | 任务已提交，等待处理 |
| `PROCESSING` | 正在处理中 |
| `WAITING` | 等待进一步操作（如等待企业管理员生成套餐） |
| `DONE` | 任务完成 |
| `ERROR` | 任务出错 |
| `CANCELLED` | 任务被取消（用户提交了新任务） |

## 完整使用示例

### 体检报告解读流程
```bash
# 1. 检查用户上传状态
curl http://localhost:8000/users/user001/upload_status

# 2. 上传PDF文件
curl -X POST http://localhost:8000/uploads \
  -F "user_id=user001" \
  -F "file=@report.pdf"

# 响应: {"message": "文件上传成功", "file_path": "...", "upload_time": "..."}

# 3. 提交解读问卷
curl -X POST http://localhost:8000/health_report \
  -H "Content-Type: application/json" \
  -d '{
    "action": "interpret",
    "name": "张三",
    "title": "体检报告解读",
    "user_id": "user001",
    "gender": "男",
    "age": 35
  }'

# 响应: {"uuid": "abc-123", "ai_status": "PENDING", "ai_results": "任务已提交..."}

# 4. 查询解读结果
curl http://localhost:8000/interpret_result/abc-123

# 响应: {"uuid": "abc-123", "ai_status": "DONE", "ai_results": "# 体检报告解读结果..."}

# 5. 或者查询用户当前任务
curl http://localhost:8000/users/user001/current_task
```

### 个性化推荐流程
```bash
# 1. 提交推荐问卷
curl -X POST http://localhost:8000/health_report \
  -H "Content-Type: application/json" \
  -d '{
    "action": "recommend",
    "name": "李四",
    "title": "个性化推荐",
    "company_code": "company_abc",
    "gender": "女",
    "age": 28
  }'

# 响应: {"uuid": "def-456", "ai_status": "PENDING", "ai_results": "任务已提交..."}

# 2. 查询推荐状态
curl http://localhost:8000/recommend_result/def-456

# 响应: {"uuid": "def-456", "ai_status": "WAITING", "ai_results": "等待企业管理员生成套餐"}

# 3. 企业端生成套餐
curl -X POST http://localhost:8000/companies/company_abc/packages/generate

# 响应: {"package_status": "PROCESSING", "package_results": "正在制定中..."}

# 4. 查询套餐状态
curl http://localhost:8000/companies/company_abc/packages/status

# 响应: {"package_status": "DONE", "package_results": "# 企业体检套餐方案..."}

# 5. 再次查询员工推荐结果（状态变为DONE）
curl http://localhost:8000/recommend_result/def-456

# 响应: {"uuid": "def-456", "ai_status": "DONE", "ai_results": "# 李四的体检套餐推荐..."}
```

## 用户覆盖机制

系统实现了智能覆盖机制，确保每个用户只有一份有效数据：

### 文件覆盖
- 用户上传新PDF时，自动删除之前的所有文件
- 每个用户目录只保留最新的一份PDF文件
- 清理相关的上传记录文件

### 任务覆盖
- 用户提交新问卷时，自动取消之前进行中的任务
- 旧任务状态变为 `CANCELLED`
- 确保不会出现重复处理

## 健康问卷字段说明

### 基础信息
- `action`: 操作类型 ("interpret" | "recommend")
- `name`: 姓名
- `title`: 报告标题
- `user_id`: 用户ID (体检报告解读必需)
- `company_code`: 企业代码 (个性化推荐必需)

### 基本健康数据
- `gender`: 性别 ("男" | "女")
- `age`: 年龄
- `pulse`: 脉搏
- `waistline`: 腹围
- `sbp`: 收缩压/高压
- `dbp`: 舒张压/低压

### 健康状况
- `symptoms`: 症状列表
- `medical_history`: 既往病史
- `surgery_history`: 手术史
- `previous_exam_abnormal`: 既往体检异常指标

### 专项检查
- `lung_nodule_level`: 肺结节分级
- `hepatic_ct_done`: 是否已行上腹部增强CT检查
- `renal_ct_done`: 是否已行肾错构瘤CT检查
- `anemia_types`: 贫血原因筛查
- `gastroscopy_done`: 既往是否行胃镜检查 ("是" | "否")
- `gastroscopy_year`: 上次胃镜检查年份
- `gastroscopy_result`: 胃镜检查结果
- `colonoscopy_done`: 既往是否行肠镜检查 ("是" | "否")
- `colonoscopy_year`: 上次肠镜检查年份
- `colonoscopy_result`: 肠镜检查结果
- `hpv_screened`: 既往是否行HPV筛查 ("是" | "否")
- `hpv_years`: 既往HPV筛查时间年份

### 生活习惯
- `smoking`: 抽烟 ("是" | "否")
- `smoking_years`: 抽烟年数
- `smoking_daily`: 每天抽烟支数
- `smoking_quit`: 是否已戒烟 ("是" | "否")
- `drinking`: 经常饮酒 ("是" | "否")
- `white_liquor_amount`: 白酒饮用量
- `beer_amount`: 啤酒饮用量
- `asbestos_exposure`: 长期接触石棉、粉尘或重金属 ("是" | "否")
- `occupation_traits`: 职业特点列表
- `lifestyle`: 生活习惯列表
- `diet`: 饮食习惯列表

### 婚姻家庭
- `marriage_status`: 是否已婚 ("是" | "否")
- `marriage_years`: 已婚年数
- `family_history`: 直系亲属病史列表
- `family_cancer_types`: 直系亲属恶性肿瘤类型列表

### 用药情况
- `antihypertensive_names`: 降压药物名称
- `lipidlowering_names`: 降脂药物名称
- `hypoglycemic_names`: 降糖药物名称
- `longterm_meds`: 长期服用的其他药物

### 企业管理
- `POST /companies/{company_code}/packages/generate` - 生成企业套餐
- `GET /companies/{company_code}/packages/status` - 查询套餐生成状态
- `GET /companies/{company_code}/dashboard` - 企业健康数据看板
- `GET /companies/{company_code}/packages/compare` - 套餐对比分析

## 技术栈

- **后端框架**: FastAPI
- **API文档**: OpenAPI 3.0 + Swagger UI
- **文件处理**: 支持PDF上传和管理
- **异步处理**: BackgroundTasks 后台任务处理
- **数据存储**: 文件系统 + 内存缓存
- **AI模型**: 集成深度学习模型进行智能分析

---

*最后更新时间: 2025-08-11*
