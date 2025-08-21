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

1. **上传PDF文件** (可选，体检报告解读需要)
2. **提交健康问卷** (传入任务UUID)
3. **查询任务结果**
4. **企业端生成套餐** (个性化推荐完成后)

## 主要接口


### 1. 健康检查
```bash
GET /health

# 响应:
{      "code": 0,
        "message": "服务正常",
        "data": {
            "status": "healthy",
            "service": "体检智能分析系统",
            "timestamp": time.strftime("%Y-%m-%d %H:%M:%S")
        }
}
```

### 2. 提交健康问卷
```bash
POST /health_report
Content-Type: application/json

{
  "action": "interpret",  # 或 "recommend"，必填
  "uuid": "任务UUID",  必填
  "name": "张三",
  "title": "体检报告解读",
  "user_id": "user001",  # 体检报告解读时必需
  "company_code": "company_abc",  # 个性化推荐时必需，必填
  "file_path": "uploads/user001/user001_1704067200.pdf", #用户的体检报告地址，体检报告解读需要

  # 基本健康数据
  "gender": "男",
  "age": 35,
  "pulse": 72,
  "waistline": 85,
  "sbp": 125,  # 收缩压
  "dbp": 80,   # 舒张压
  
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
  
  # 个人史
  "personal_history": ['经常喝酒'],

  # 生活习惯
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

# 响应:
{
  "code": 0,
  "ai_results": "健康问卷已成功提交，智能体正在解读体检报告。",
  "ai_status": "PENDING"
}
```

### 3. 获取体检报告解读结果
```bash
GET /interpret_result/{uuid}

# 响应:
{
  "code":0,
  "ai_status": "DONE",
  "ai_results": "# 体检报告解读结果\n\n基于您的体检报告和问卷信息..."
}
```

### 4. 获取个性化推荐结果
```bash
GET /recommend_result/{uuid}

# 响应:
{
  "code":0,
  "ai_status": "WAITING",  # 或 "DONE"
  "ai_results": "个性化推荐已完成，等待企业管理员生成体检套餐。"
}
```

### 5. 企业端生成套餐
```bash
POST /companies/{company_code}/packages/generate

# 响应:
{
  "code":0,
  "package_results": "套餐生成任务已启动，正在制定中…",
  "package_status": "PROCESSING"
}
```

### 6. 查询企业套餐状态
```bash
GET /companies/{company_code}/packages/status

# 响应:
{
  "code":0,
  "package_status": "DONE",
  "package_results": "# 企业体检套餐方案\n\n## 男性套餐\n\n### 基础套餐\n- **适用人群：** 20人\n- **预估费用：** ¥800.00\n..."
}
```

### 7. 企业健康数据看板
```bash
GET /companies/{company_code}/dashboard

# 响应:
{
  "code":0,
  "health_data": "# 企业员工健康分析报告（2025年度）\n\n## 数据概览\n- 在职员工总数：待统计\n..."
}
```

### 8. 企业套餐对比分析
```bash
GET /companies/{company_code}/packages/compare

# 响应:
{
  "code":0,
  "compare_analyse": "# 体检套餐对比分析报告\n\n## 2025年与2024年对比概览\n- 整体套餐费用：待分析\n..."
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

## 技术栈

- **后端框架**: FastAPI
- **API文档**: OpenAPI 3.0 + Swagger UI
- **文件处理**: 支持PDF上传和管理
- **异步处理**: BackgroundTasks 后台任务处理
- **数据存储**: 文件系统 + 内存缓存
- **AI模型**: 集成深度学习模型进行智能分析
---

*最后更新时间: 2025-08-11*
