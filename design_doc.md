# Care Plan Generation System - Design Doc

---

## 1. 项目背景

本系统是一个面向 specialty pharmacy 的内部工具，用于基于患者临床信息自动生成标准化 Care Plan，以替代药剂师手工编写流程并提升效率与合规性。

---

## 2. 用户和使用场景

### 用户角色

- **Medical Assistant**
  - 录入患者、provider、订单及临床信息
  - 处理系统提示（warning / error）
  - 触发 care plan 生成

- **Pharmacist / Clinical Staff**
  - 查看、下载 care plan
  - 打印并交付给患者

> ⚠️ 病人不直接使用系统  
> ⚠️ Phase 1 不实现登录系统，所有 CVS 工作人员共享访问

---

### 使用场景（核心流程）

1. 医疗工作者在 Web 表单中输入患者与订单信息  
2. 前端执行基础格式校验  
3. 后端执行数据校验 + 重复检测  
4. ERROR 阻止 / WARNING 可继续  
5. 调用 LLM 生成 Care Plan  
6. 下载并打印  
7. 用于报表导出  

---

## 3. 功能需求

### ✅ 必须实现（MVP / Phase 1）

- Web 表单输入
- 双层校验（前端 + 后端）
- 重复检测（Patient / Order / Provider）
- Care Plan 自动生成
- Care Plan 下载
- 报表导出
- Patient Records（仅 string）

---

### 🟡 Phase 2

- 登录/权限
- PDF 上传解析
- Care Plan 编辑
- 历史记录
- EHR 集成

---

## 4. 数据模型

### 1. Patient（静态信息）

- id
- first_name
- last_name
- MRN
- DOB

---

### 2. Provider

- id
- name
- NPI (unique)

---

### 3. Order（本次开药相关）

- id
- patient_id (FK)
- provider_id (FK)
- medication_name
- primary_diagnosis
- additional_diagnoses
- medication_history
- patient_records (string)
- order_date

---

### 4. CarePlan

- id
- order_id (FK)
- problem_list
- goals
- interventions
- monitoring_plan
- generated_at

---

### 实体关系

Patient 1 --- N Order  
Provider 1 --- N Order  
Order 1 --- 1 CarePlan  

---

## 5. 关键业务规则

### Care Plan

- 一个 Order → 一个 Care Plan
- 必须包含：
  - Problem list
  - Goals
  - Pharmacist interventions
  - Monitoring plan

---

### 重复检测

**订单**

- 同一患者 + 同一药物 + 同一天 → ❌ ERROR  
- 同一患者 + 同一药物 + 不同天 → ⚠️ WARNING  

**患者**

- MRN 相同 + 名字或 DOB 不同 → ⚠️ WARNING  
- 名字 + DOB 相同 + MRN 不同 → ⚠️ WARNING  

**Provider**

- NPI 相同 + 名字不同 → ❌ ERROR  

---

### 验证规则

**前端**
- 必填
- 格式（NPI / MRN / ICD）

**后端**
- 所有格式验证再执行
- 业务规则验证（重复 / 一致性）

---

### 错误处理

- ERROR → 阻止
- WARNING → 可继续
- 所有错误必须清晰、可控

---

## 6. 技术栈

| 层 | 技术 |
|---|------|
| 前端 | React |
| 后端 | Python + Django + Django REST Framework |
| 数据库 | PostgreSQL |
| 异步任务 | Celery + Redis（本地） |
| AI / LLM | Claude API 或 OpenAI API |
| 容器化 | Docker |
| 测试 | pytest |

---

## 总结

系统核心：

1. 数据校验（前后端双层）
2. 业务规则引擎（重复检测）
3. LLM 生成模块

关键重点：

- 数据一致性
- 重复检测准确性
- LLM 输出结构稳定
