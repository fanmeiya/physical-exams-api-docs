# 体检智能分析系统

提供体检报告解读和套餐推荐服务的API

**版本**: 1.0.0

## 在线访问

- **API文档（重要）**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html)
- **OpenAPI规范**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.json](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.json)

## 核心变更（V2.0）

1.  **新增企业上传官方套餐流程**：允许企业管理员上传包含多个套餐的Excel文件。
2.  **个人推荐流程分离**：员工提交问卷后，系统会根据企业是否上传了官方套餐，自动进入两条互斥的推荐流程之一：
    *   **流程A (匹配官方套餐)**: 如果企业已上传套餐，系统会为员工匹配最合适的官方套餐。
    *   **流程B (分配AI生成套餐)**: 如果企业未上传套餐，则沿用旧逻辑，等待管理员触发AI生成套餐并为员工分配。
3.  **新增专用接口**：为此增加了 `packages/upload`, `packages/upload/status`, 和 `match_result` 三个新接口。

## 工作流程详解

### 流程A：匹配官方套餐（企业已上传套餐）

1.  **企业管理员**：调用 `POST /companies/{company_code}/packages/upload` 提交套餐Excel文件，并用 `GET /companies/{company_code}/packages/upload/status/{task_id}` 查询处理状态，直至成功。
2.  **员工**：提交健康问卷 `POST /health_report`。
3.  **前端**：轮询 **`GET /match_result/{uuid}`** 接口。
4.  **后端**：
    *   `match_status` 变为 `PROCESSING`。
    *   匹配完成后，`match_status` 变为 `DONE`，并返回AI生成的匹配报告和该套餐的原始Excel内容。

### 流程B：分配AI生成套餐（企业未上传套餐）

1.  **员工**：提交健康问卷 `POST /health_report`。
2.  **前端**：此时轮询 `GET /match_result/{uuid}` 会得到 `match_status: "NOT_STARTED"`。
3.  **企业管理员**：在企业端点击“生成套餐”，触发 `POST /companies/{company_code}/packages/generate`。
4.  **后端**：为全公司员工分配AI生成的套餐。
5.  **前端**：此时轮询 **`GET /recommend_result/{uuid}`** 接口，会发现 `assignment_status` 变为 `DONE`，并得到分配结果。

**前端开发建议**：员工提交问卷后，可同时轮询 `GET /match_result/{uuid}` 和 `GET /recommend_result/{uuid}`。

## 主要接口

### 1. 提交健康问卷
- **接口**: `POST /health_report`
- **说明**: 保持不变。提交问卷，启动后台分析任务。

### 2. 获取体检报告解读结果
- **接口**: `GET /interpret_result/{uuid}`
- **说明**: 保持不变。获取由AI生成的体检报告文字解读。

### 3. 获取个性化项目推荐
- **接口**: `GET /personal_recommendation_result/{uuid}`
- **说明**: 保持不变。获取基于问卷生成的、与套餐无关的个人“应检项目”清单。

---
### 4. 【新增】上传企业官方套餐
```bash
POST /companies/{company_code}/packages/upload
Content-Type: application/json

{
  "excel_path": "/path/on/server/company_a_packages.xlsx",
  "task_id": "frontend-generated-uuid-for-this-upload"
}

# 成功响应 (202 Accepted):
{
  "message": "文件上传成功，正在后台进行解析和标准化"
}
```

### 5. 【新增】查询企业套餐上传状态
```bash
GET /companies/{company_code}/packages/upload/status/{task_id}

# 处理中响应:
{
  "code": 0,
  "upload_status": "PROCESSING",
  "upload_result": "正在进行标准化匹配..."
}

# 完成响应:
{
  "code": 0,
  "upload_status": "DONE",
  "upload_result": "套餐上传及处理完成"
}
```

### 6. 【新增】获取官方套餐匹配结果
```bash
GET /match_result/{uuid}

# 处理中响应:
{
  "code": 0,
  "match_result": "贵公司已上传官方套餐，正在为您匹配最合适的套餐...",
  "match_status": "PROCESSING",
  "package_name": null,
  "package_content": null
}

# 完成响应:
{
  "code": 0,
  "match_status": "DONE",
  "match_result": "# 套餐匹配报告\n\n## 推荐套餐：精英A套餐...",
  "package_name": "精英A套餐",
  "package_content": [
    { "项目分类": "基础检查", "项目名称": "身高体重", "说明": "必检" },
    { "项目分类": "实验室检查", "项目名称": "血常规", "说明": "必检" }
  ]
}
```

### 7. 【用途变更】获取AI生成套餐的分配结果
- **接口**: `GET /recommend_result/{uuid}`
- **说明**: 此接口现在**仅用于**获取由管理员触发AI生成套餐后的分配结果。
```bash
GET /recommend_result/{uuid}

# 等待管理员操作:
{
  "code": 0,
  "personal_package": "需等待企业管理员上传或生成套餐。",
  "assignment_status": "NOT_STARTED"
}

# 分配完成响应:
{
  "code": 0,
  "personal_package": "# AI套餐分配结果\n\n## 分配套餐：心脑血管专项...",
  "assignment_status": "DONE"
}
```

### 8. 企业端生成AI套餐
- **接口**: `POST /companies/{company_code}/packages/generate`
- **说明**: 保持不变。

### 9. 查询企业AI套餐生成状态
- **接口**: `GET /companies/{company_code}/packages/status`
- **说明**: 保持不变。

---
*最后更新时间: 2025-11-28 15:31:36*
