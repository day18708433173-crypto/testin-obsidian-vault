# real 云真机平台总览

> 分支无关的通用介绍。分支相关的接口明细、SQL、代码流程见 [分支文档](工程模块清单/real/branches/syy.release.z7.8.1.0/00-分支索引.md)。

## 模块组成

| 模块 | 职责 |
|------|------|
| 平台配置 | 平台授权、数据库环境管理、任务错误信息、项目配置的管理服务（增删查改） |
| 设备控制中心 | 与上位机长连接、设备管理、任务下发 |
| 任务调度服务 | 任务调度服务：任务基础信息初始化、任务调度、报告处理 |
| app处理服务 | 平台 App 设备任务对接：任务初始化、任务取消等，主要面向 App 模块 |
| real-portal | 门户（本文档库暂未覆盖，待补充） |

## 接口入口形态

1. `/*` 兜底：ApiServlet（real-common），按 action/op 反射调用 cn.testin.service.*
2. `/v3/*`：Spring MVC @RestController
3. IceRPC（如代码中存在）
4. 设备控制中心：与上位机的长连接协议

## 数据存储

MySQL（MyBatis）/ MongoDB / Elasticsearch / Redis，详见分支文档的 SQL 部分。

## 外部服务

本模块依赖的外部服务（跨模块调用 / 外部系统）统一见 [外部服务索引](../../外部服务/外部服务-索引.md)（工程模块清单/ 级共享，含 user-manager / notice-manager / script-service / file-service / 平台配置 / 任务调度服务 / app处理服务 / 设备控制中心 等）。

## 专题

见 [专题索引](专题-索引.md)（任务状态机、通知系统、后台线程、模块间通信、核心链路图）。