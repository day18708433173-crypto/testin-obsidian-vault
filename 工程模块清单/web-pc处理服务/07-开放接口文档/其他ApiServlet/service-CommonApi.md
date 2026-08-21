# service-CommonApi — 任务模版保存（本地封装）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/common/CommonApi.java`（@Service）
> 类型：本地业务封装类（非远端转发；内部走本地 `TemplateService` → quartz_job 表）
> 说明：类名带 Api 且被 ApiServlet 以 `action=common, op=CommonApi.saveTemplate` 反射路由调用，但实现落在本模块

## 方法列表

### 1. saveTemplate — 保存任务模版

```java
public Integer saveTemplate(String templateStr) throws GeneralException
```

**用途**：保存任务模版 JSON，返回生成的 jobId；失败或结果 <=0 抛 `noneData`（"保存任务模版失败"）。

**流程**：
1. `templateService.saveTemplate(templateStr)` → 本地 TemplateService（`ServiceImpl<QuartzJobMapper, QuartzJob>`，写 MySQL quartz_job）
2. 结果校验，异常统一包装为 GeneralException

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| templateStr | String | 是 | 任务模版完整 JSON 字符串（含 projectid，透传 `TemplateService.saveTemplate`） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 新生成的 jobId（模板ID） |

**涉及表**：

| 存储 | 表 | 操作 |
|------|-----|------|
| MySQL db_common | quartz_job | 写（→TemplateService） |

**调用者**：
- `TaskServiceImpl.java`（`commonApi.saveTemplate(jsonObj.toString())`，随后 `dirQuartApi.addDirQuart` 绑定目录）
- ApiServlet 路由入口（action=common）

## 相关文档

- [00-分支索引](00-分支索引.md)
- [外部服务索引](../../../外部服务/外部服务-索引.md)
- [service-DirQuartzApi](service-DirQuartzApi.md)、[service-McPcTaskApi](service-McPcTaskApi.md)
