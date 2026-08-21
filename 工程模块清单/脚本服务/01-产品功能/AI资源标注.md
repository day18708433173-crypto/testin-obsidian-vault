---
branch: syy.release.z7.8.1.0
module: filesystem
type: 产品功能
---

# AI 资源标注

## 功能是什么

管理 **AI 标注资源**（`ai_resource`）：用于 AI 识别/校验的标注素材（如图片标注、模型资源）的增删查改与批量导入导出。

## 核心能力

- AI 标注资源的 CRUD；
- AI 资源的批量导入 / 导出（走 `ResourceUtils` 的 JVM 内队列 + `ExportDispatchThread` / `ImportDispatchThread`，见 [脚本导入导出异步队列](../04-复杂功能细节/横切-脚本导入导出异步队列.md)）。

## 入口文档

- [ResourcesMarkController](../07-开放接口文档/AI资源标注/ResourcesMarkController.md)（V3）
- [Resources](../07-开放接口文档/AI资源标注/Resources.md)（V1，`action=ai`）

## 延伸阅读

- [脚本导入导出异步队列](../04-复杂功能细节/横切-脚本导入导出异步队列.md)
- [功能总览](功能总览.md)
