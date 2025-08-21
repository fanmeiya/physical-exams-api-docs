# 体检智能分析系统

提供体检报告解读和套餐推荐服务的API

**版本**: 1.0.0

## 在线访问

- **API文档（重要）**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html)
- **OpenAPI规范**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.json](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.json)

## 快速开始

### 基础URL
```
http://localhost:8000
```

### 工作流程

1. **前端生成UUID** (任务唯一标识)
2. **上传PDF文件** (前端自行处理，体检报告解读需要)
3. **提交健康问卷** (传入UUID和file_path)
4. **查询任务结果** (使用UUID)
5. **企业端生成套餐** (个性化推荐完成后)

## 主要接口

### 1. 健康检查
```bash
GET /health

# 响应:
{
    "code": 0,
    "message": "服务正常",
    "data": {
        "status": "healthy",
        "service": "体检智能分析系统",
        "timestamp": "2025-08-21 14:30:00"
    }
}
```

### 2. 提交健康问卷
```bash
POST /health_report
Content-Type: application/json

{
  "action": "interpret",           # 必填: "interpret"(体检报告解读) 或 "recommend"(个性化推荐)
  "uuid": "用户生成的UUID",        # 必填: 前端生成的任务唯一标识
  "company_code": "company_abc",   # 必填: 企业代码
  
  # 基本信息 (可选)
  "name": "张三",
  "title": "体检报告解读",
  "user_id": "user001",
  
  # 体检报告解读专用 (action="interpret"时必填)
  "file_path": "/path/to/uploaded/report.pdf",  # PDF文件路径，前端上传后传入
  
  # 健康问卷数据
  "gender": "男",
  "age": 35,
  "pulse": 72,
  "waistline": 85,
  "sbp": 125,                      # 收缩压
  "dbp": 80,                       # 舒张压
  
  # 健康状况
  "symptoms": ["头痛", "失眠"],
  "medical_history": ["高血压"],
  "surgery_history": "阑尾切除术（2015年）",
  "previous_exam_abnormal": ["血压偏高", "血脂异常"],
  
  # 专项检查
  "lung_nodule_level": "1级",
  "hepatic_ct_done": "否",
  "renal_ct_done": "否",
  "anemia_types": ["缺铁性贫血"],
  "gastroscopy_done": "是",
  "gastroscopy_year": 2023,
  "gastroscopy_result": "浅表性胃炎",
  "colonoscopy_done": "否",
  "colonoscopy_year": 0,
  "colonoscopy_result": "",
  "hpv_screened": "是",
  "hpv_years": 2024,
  
  # 个人史和生活习惯
  "personal_history": ["经常喝酒"],
  "smoking_years": 0,
  "smoking_daily": 0,
  "smoking_quit": "否",
  "white_liquor_amount": 50,
  "beer_amount": 500,
  "occupation_traits": ["久坐", "高压"],
  "lifestyle": ["熬夜", "运动少"],
  "diet": ["高盐", "高油"],
  
  # 婚姻家庭
  "marriage_status": "是",
  "marriage_years": 10,
  "family_history": ["高血压", "糖尿病"],
  "family_cancer_types": ["肺癌"],
  
  # 用药情况
  "antihypertensive_names": "氨氯地平",
  "lipidlowering_names": "阿托伐他汀",
  "hypoglycemic_names": "",
  "longterm_meds": "维生素D"
}

# 成功响应:
{
  "code": 0,
  "ai_results": "健康问卷已成功提交，智能体正在解读体检报告",
  "ai_status": "PENDING"
}

# 错误响应:
{
  "code": 400,
  "ai_results": "体检报告解读需要提供file_path",
  "ai_status": "ERROR"
}
```

### 3. 获取体检报告解读结果
```bash
GET /interpret_result/{uuid}

# 处理中响应:
{
  "code": 0,
  "ai_results": "正在解读体检报告...",
  "ai_status": "PROCESSING"
}

# 完成响应:
{
  "code": 0,
  "ai_results": "# 体检报告解读结果\n\n## 主要发现\n\n基于您的体检报告和问卷信息，发现以下问题：\n\n### 1. 血压偏高\n- 收缩压125mmHg，略高于正常范围\n- 建议：低盐饮食，规律运动\n\n### 2. 血脂异常\n- 总胆固醇偏高\n- 建议：控制饮食，定期复查\n\n## 健康建议\n\n1. **饮食调整**：减少高盐、高油食物摄入\n2. **生活方式**：增加运动，改善睡眠\n3. **定期监测**：建议3个月后复查血压血脂\n\n*注：本解读仅供参考，具体诊疗请咨询专业医生*",
  "ai_status": "DONE"
}

# 错误响应:
{
  "code": 404,
  "ai_results": "任务不存在",
  "ai_status": "ERROR"
}
```

### 4. 获取个性化推荐结果
```bash
GET /recommend_result/{uuid}

# 等待响应:
{
  "code": 0,
  "ai_results": "个性化推荐已完成，等待企业管理员生成体检套餐。",
  "ai_status": "WAITING"
}

# 完成响应 (套餐生成后):
{
  "code": 0,
  "ai_results": "# 张三的体检套餐推荐\n\n## 推荐套餐：心脑血管专项套餐\n\n**匹配度：** 85%\n\n**套餐已包含项目：**\n- 心电图\n- 血压监测\n- 血脂全套\n- 颈动脉彩超\n\n**建议补充项目：**\n- 动态血压监测\n- 冠脉CTA\n\n**补充项目预估费用：** ¥350.00\n\n**说明：** 此套餐方案基于您的个人健康状况和公司整体情况制定，如有疑问请咨询HR或医务人员。",
  "ai_status": "DONE"
}
```

### 5. 企业端生成套餐
```bash
POST /companies/{company_code}/packages/generate

# 成功响应:
{
  "code": 0,
  "package_results": "套餐生成任务已启动，正在制定中…",
  "package_status": "PROCESSING"
}

# 错误响应:
{
  "code": 404,
  "package_results": "未找到企业 company_abc 的数据",
  "package_status": "ERROR"
}
```

### 6. 查询企业套餐状态
```bash
GET /companies/{company_code}/packages/status

# 完成响应:
{
  "code": 0,
  "package_status": "DONE",
  "package_results": "# 企业体检套餐方案\n\n**员工总数：** 50人\n\n## 男性套餐\n\n### 基础套餐\n- **适用人群：** 20人\n- **预估费用：** ¥800.00\n- **包含项目：** 血常规, 尿常规, 心电图, 胸部X光\n\n### 心脑血管套餐\n- **适用人群：** 15人\n- **预估费用：** ¥1200.00\n- **包含项目：** 血常规, 血脂全套, 心电图, 颈动脉彩超\n\n## 女性套餐\n\n### 基础套餐\n- **适用人群：** 10人\n- **预估费用：** ¥850.00\n- **包含项目：** 血常规, 尿常规, 妇科检查, 乳腺彩超\n\n### 妇科专项套餐\n- **适用人群：** 5人\n- **预估费用：** ¥1300.00\n- **包含项目：** 血常规, 妇科检查, 乳腺彩超, HPV检测\n\n## 套餐汇总\n\n- **男性套餐数量：** 2个\n- **女性套餐数量：** 2个\n- **总套餐数量：** 4个"
}

# 等待状态:
{
  "code": 0,
  "package_status": "PENDING",
  "package_results": "有员工推荐数据，可以开始生成套餐"
}
```

### 7. 企业健康数据看板
```bash
GET /companies/{company_code}/dashboard

# 响应:
{
  "code": 0,
  "health_data": "# 企业员工健康分析报告（2025年度）\n\n## 数据概览\n- 在职员工总数：待统计\n- 已提交问卷人数：待统计\n- 心脑血管高风险人数：待分析\n..."
}
```

### 8. 企业套餐对比分析
```bash
GET /companies/{company_code}/packages/compare

# 响应:
{
  "code": 0,
  "compare_analyse": "# 体检套餐对比分析报告\n\n## 2025年与2024年对比概览\n- 整体套餐费用：待分析\n- 套餐数量变化：待统计\n..."
}
```

## 响应码说明

| 响应码 | 说明 | 
|--------|------|
| `0` | 成功 |
| `400` | 请求参数错误 |
| `404` | 资源不存在 |
| `500` | 服务器内部错误 |

## 任务状态说明

| 状态 | 说明 |
|------|------|
| `PENDING` | 任务已提交，等待处理 |
| `PROCESSING` | 正在处理中 |
| `WAITING` | 等待进一步操作（如等待企业管理员生成套餐） |
| `DONE` | 任务完成 |
| `ERROR` | 任务出错 |
| `CANCELLED` | 任务被取消（用户提交了新任务） |

## 工作流程详解

### 体检报告解读流程
1. **前端**：生成UUID，用户上传PDF文件
2. **前端**：调用 `POST /health_report` 传入 `action="interpret"`, `uuid`, `file_path` 和问卷数据
3. **后端**：返回 `ai_status="PENDING"`，开始后台解读
4. **前端**：轮询 `GET /interpret_result/{uuid}` 查询结果
5. **后端**：解读完成后返回 `ai_status="DONE"` 和解读结果

### 个性化推荐流程
1. **前端**：生成UUID，用户填写健康问卷
2. **前端**：调用 `POST /health_report` 传入 `action="recommend"`, `uuid`, `company_code` 和问卷数据
3. **后端**：返回 `ai_status="PENDING"`，开始生成个人推荐
4. **前端**：轮询 `GET /recommend_result/{uuid}` 查询结果
5. **后端**：个人推荐完成后返回 `ai_status="WAITING"`，等待企业生成套餐
6. **企业管理员**：调用 `POST /companies/{company_code}/packages/generate` 生成套餐
7. **后端**：套餐生成完成后，所有员工的状态变为 `ai_status="DONE"`

## 技术栈

- **后端框架**: FastAPI
- **API文档**: OpenAPI 3.0 + Swagger UI  
- **异步处理**: BackgroundTasks 后台任务处理
- **数据存储**: 文件系统 + 内存缓存
- **AI模型**: 集成深度学习模型进行智能分析
- **模型管理**: 全局模型加载，避免重复初始化

## 注意事项

1. **UUID管理**：前端需要自行生成和管理UUID，建议使用标准的UUID4格式
2. **文件路径**：file_path需要是服务器可访问的绝对路径
4. **响应格式**：所有接口都返回统一的响应格式，通过code字段判断成功/失败
5. **轮询建议**：前端轮询间隔建议2-5秒，避免过频请求

---

*最后更新时间: 2025-08-21*
