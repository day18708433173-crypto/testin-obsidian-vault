# Redis 存储总览

脚本服务 除 MySQL（db_file）外，还使用 Redis 承担**任务队列**与**缓存**两类职责。连接配置见 `redisconfig.xml` / `JedisPoolConf`，代码内以 `Constants.FILE_REDIS = "file"` 标识本模块的 Redis 实例。

## Key 清单

### 任务队列（List 结构）

| Key | 类型 | 生产者 → 消费者 | 说明 |
|-----|------|------------------|------|
| `Script.parseKey.queue` | List | 脚本上传/录制 → 解析线程（TestinScriptManageService.getParseJobTaskThread） | 待解析脚本 URL 队列，见 [脚本解析与导入](../../脚本服务/04-复杂功能细节/核心链路-脚本解析与导入.md) |
| `Script.importKey.queue` | List | 导入请求 → 导入线程（getImportJobTaskThread） | 待导入脚本 URL 队列 |
| `Script.importKey.exec.queue` | List | 导入调度 → 导入执行 | 导入执行队列 |
| `queue:action_log` | List | ActionLogService（导出/复制/导入）→ 平台基础功能服务 消费者 | 操作日志异步上报队列，落库 [script_action_log](../db_file/script_action_log.md)，见 [AOP与操作日志](../../脚本服务/04-复杂功能细节/横切-AOP与操作日志.md) |
| `parseAppQueue` | List | AppService → ParseAppStartThread | 待解析 App 队列，见 [后台线程全景](../../脚本服务/04-复杂功能细节/横切-后台线程全景.md) |
| `processParseApp` | List | ParseAppStartThread 内部 | App 解析"处理中"缓冲队列（brpoplpush 防丢失），成功才弹出 |

### 任务状态（String 结构，JSON 值，TTL 1 天）

| Key 模式 | 说明 |
|----------|------|
| `Script.parseKey.{scriptUrl}` | 单个脚本的解析状态（state/desc/details/dirs 等 JSON 字段），TTL 86400s |
| `Script.importKey.{scriptUrl}` | 单个脚本的导入状态（含 result/depends/errorMsg 等），TTL 86400s |
| `Script.importKey.req.{scriptUrl}` | 导入请求快照（requestId + actionLog JSON），TTL 86400s |

### 缓存（String 结构）

| Key 模式 | TTL | 说明 |
|----------|-----|------|
| `FILESYSTEM_SCRIPT_IMAGE_INFO_{appid}` | 3600s | App 图标 URL 缓存（CommonFileCacheService，回源 common_app 表） |
| `BATCH_RF_REPOSITORY_KEY` | — | 脚本批量刷新任务仓库（RedisConstants.BATCH_RF_REPOSITORY，ScriptService.getBatchRfJob） |

## 职责边界

- **Redis**：跨进程任务队列、任务状态快照、短 TTL 热点缓存。
- **JVM 内存队列**：数据管理（AI 资源）导入导出使用 `ResourceUtils` 的 `LinkedBlockingQueue`（exportQueue/importQueue），**不在 Redis**，重启即丢失，见 [导入导出异步队列](../../脚本服务/04-复杂功能细节/横切-脚本导入导出异步队列.md)。
- **MySQL**：最终持久化，所有队列任务的结果均回写 db_file 各表。
