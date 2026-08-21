# TemplateService
- **职责**：模板管理服务，按目录类型（APP/用例/WEB/PC）删除对应模板及执行记录
- **调用方**：平台基础功能服务（`TemplateV1Api`）
  - deleteTemplateByCondition（APP 远程停 Job / 用例删除任务模板+执行记录 / WEB-PC 批量删除 Quartz Job）
  - 内部依赖 `TemplateV3Api`（getTemplateHaveTypeByCaseTemplateIds / getTaskTemplateResponseByCondition）
- **被以下文档引用**：
  - PlanDeviceController.md（取模板覆盖的套件类型/OS 类型）
  - PlanInfoController.md（复制计划补偿删除模板）
- **状态**：stub（待对应仓库代码补全）
