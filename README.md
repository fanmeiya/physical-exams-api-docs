# 体检智能分析系统

提供体检报告解读和套餐推荐服务的API

**版本**: 1.0.0

## 🌐 在线访问

- **主页**: [https://fanmeiya.github.io/physical-exams-api-docs/](https://fanmeiya.github.io/physical-exams-api-docs/)
- **API文档**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html)
- **备用文档**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs_fallback.html](https://fanmeiya.github.io/physical-exams-api-docs/api_docs_fallback.html)

## 📁 文件说明

- `index.html` - 项目主页和快速导航
- `api_docs.html` - 完整的交互式API文档
- `api_docs_fallback.html` - 兼容性备用文档
- `api_docs.json` - OpenAPI 3.0 规范文件

## 🚀 快速开始

### 基础URL
```
http://localhost:8000
```

### 主要接口

#### 1. 健康检查
```bash
GET /health
```

#### 2. 提交健康问卷
```bash
POST /health_report
Content-Type: application/json

{
  "action": "interpret", // 或 "recommend"
  "name": "用户姓名",
  "title": "报告标题",
  "gender": "男", // 或 "女"
  "age": 30
  // ... 其他健康问卷字段
}
```

#### 3. 查询任务状态
```bash
GET /task_status/{task_id}
```

#### 4. 获取结果
```bash
GET /interpret_result/{task_id}    # 体检报告解读结果
GET /recommend_result/{task_id}    # 个性化推荐结果
```

#### 5. 企业套餐相关
```bash
POST /companies/{company_code}/packages/generate  # 生成企业套餐
GET /companies/{company_code}/packages/status     # 查询套餐状态
GET /companies/{company_code}/dashboard           # 健康数据看板
GET /companies/{company_code}/packages/compare    # 套餐对比分析
```

## 📊 状态码说明

| 状态 | 说明 |
|------|------|
| PENDING | 任务已提交，等待处理 |
| PROCESSING | 正在处理中 |
| WAITING | 等待进一步操作（如等待企业管理员生成套餐） |
| DONE | 任务完成 |
| ERROR | 任务出错 |

## 🔧 技术栈

- **后端框架**: FastAPI
- **API文档**: OpenAPI 3.0 + Swagger UI
- **部署**: GitHub Pages

## 📧 联系方式

如有问题，请联系开发团队。
