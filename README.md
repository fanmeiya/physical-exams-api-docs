# 体检智能分析系统

提供体检报告解读和套餐推荐服务的API

**版本**: 1.0.0
**最后更新**: 2025-11-28 16:27:32

## 在线访问

- **API文档（交互式）**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.html)
- **OpenAPI规范**: [https://fanmeiya.github.io/physical-exams-api-docs/api_docs.json](https://fanmeiya.github.io/physical-exams-api-docs/api_docs.json)

## 核心变更说明（V2.0）

### 1. 个人推荐流程优化（重要）
员工提交问卷后，若`action`字段为`recommend`，则会进行个性化项目推荐(**原逻辑**），在原逻辑基础上增加了：系统会立即检测企业是否已上传官方套餐，并在响应中告知前端下一步操作。这避免了前端盲目轮询。

*   **场景 A：企业已上传官方套餐**
    *   `POST /health_report` 返回 `has_official_package: true`。
    *   前端应轮询 **`GET /match_result/{uuid}`** 获取匹配结果。
*   **场景 B：企业未上传官方套餐**
    *   `POST /health_report` 返回 `has_official_package: false`。
    *   前端**不应轮询 **`GET /match_result/{uuid}`。

### 2. 新增企业套餐上传功能
企业管理员现在可以上传官方 Excel 套餐文件，系统会自动解析并用于员工匹配。

### 3. 重要区分说明

现在系统总共有两类套餐匹配逻辑：1. 若企业上传了官方套餐，则员工端访问`GET /match_result/{uuid}` 可匹配官方套餐；2. 适用于企业方暂未上传官方套餐，则员工端不用访问`GET /match_result/{uuid}`，可以等待企业方上传，也可以由企业管理员基于群体自己制定套餐然后分配给该员工(即接口：`GET /recommend_result/{uuid}`,之前的系统逻辑保持不变)。

---

## 接口调用指南

### 1. 提交健康问卷（已更新）
- **接口**: `POST /health_report`
- **变更**: 响应体新增 `has_official_package` 字段。
- **前端逻辑**:
    1.  调用此接口提交问卷。
    2.  检查响应中的 `has_official_package`。
    3.  若为 `true`，则可以轮询 `/match_result/{uuid}`；否则前端不用展示该接口。

```json
// 响应示例
{
  "message": "问卷提交成功",
  "has_official_package": true  // true: 轮询 match_result; false:不访问 match_result
}
```
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
  "task_id": "11111"
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

**`upload_status`状态含义**
`PENDING` 任务已提交,准备下载文件
`PROCESSING` 正在解析Excel或进行标准化
`DONE` 处理完成
`ERROR` 处理失败

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
  "package_content": "[
    { "项目分类": "基础检查", "项目名称": "身高体重", "说明": "必检" },
    { "项目分类": "实验室检查", "项目名称": "血常规", "说明": "必检" }
  ]"
}
```

其中`match_result`是套餐匹配情况说明（包含了推荐的套餐名，理由，待补充的体检项目以及理由）

`package_name`是套餐名

`package_content`是该套餐的原始表格内容的JSON字符串，前端需要把这部分内容渲染为表格形式，让员工可以查看套餐具体内容

**响应字段说明**

| 字段              | 类型   | 说明                                                         |
| ----------------- | ------ | ------------------------------------------------------------ |
| `match_status`    | string | 匹配状态: NOT_STARTED(未开始)/PROCESSING(处理中)/DONE(已完成)/ERROR(出错了) |
| `match_result`    | string | Markdown格式的匹配报告                                       |
| `package_name`    | string | 匹配到的套餐名称(Excel中的sheet名)                           |
| `package_content` | array  | 该套餐包含的所有项目详情                                     |

### 7. 获取AI生成套餐的分配结果

- **接口**: `GET /recommend_result/{uuid}`
- **说明**: 保持不变。
### 8. 企业端生成AI套餐
- **接口**: `POST /companies/{company_code}/packages/generate`
- **说明**: 保持不变。

### 9. 查询企业AI套餐生成状态
- **接口**: `GET /companies/{company_code}/packages/status`
- **说明**: 保持不变。

### 10. 查询企业员工整体的健康数据

- **接口**：`GET /companies/{company_code}/dashboard`

- **说明**：保持不变

### 11.  查询企业端与之前体检套餐的对比分析

- **接口**：`GET /companies/{company_code}/packages/compare`

- **说明**：保持不变

---
*最后更新时间: 2025-11-28 16:27:32*
