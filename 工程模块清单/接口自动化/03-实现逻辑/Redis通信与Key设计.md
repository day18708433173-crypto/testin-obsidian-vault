---
tags: [实现逻辑]
---

# Redis通信与Key设计

## 概述

服务端与执行端之间**没有 RPC、没有 HTTP 回调主链路**，全部异步通信都走 Redis：任务下发走"每台执行机一个 list 队列"，结果回传走"服务端一个 list 队列"，任务本体（JMX）和执行状态作为普通 key-value 存放。所有 key 的命名集中在 `src/main/java/cn/testin/commons/utils/RedisKeyUtil.java`，统一前缀 `TESTIN_API_BACKEND:`（`RedisKeyUtil.java`）。

**排查"任务下发后没反应 / 报告卡住不动"类问题，先看本文的 key 表和文末排查指引。**

## 三套 Redis 客户端

注意仓库里有**三套相互独立**的 Redis 访问方式，混用会踩坑：

| 客户端 | 所在进程 | 实现 | 用途 |
|---|---|---|---|
| `RedisUtil` | 服务端 | Spring `StringRedisTemplate` 封装（`src/main/java/cn/testin/commons/utils/RedisUtil.java`），连接信息来自 `spring.redis.*` | 服务端绝大多数读写：JMX 写入、批次缓冲、停止列表、队列 lpush（`ServerProducer` 内部也是它） |
| `BackendJedisPoolManager` | 服务端 | 原生 Jedis 池（`src/main/java/cn/testin/config/redisConfig/BackendJedisPoolManager.java`），同样读 `spring.redis.*`，`@PostConstruct` 建池 | 只服务 `ServerConsumer` 的 brpop 长线程（`ServerConsumer.java`）。brpop(0) 会永久占住一个连接，所以不能用 RedisTemplate |
| `RunnerJedisPoolManager` / `RunnerJedisPoolUtils` / `RunnerProducer` / `RunnerConsumer` | 执行端 | 原生 Jedis（`src/main/java/cn/testin/runner/jedisConfig/`），连接信息来自 `RunnerApplication` 的**位置参数**而不是配置文件 | 执行端全部 Redis 访问 |

执行端不是 Spring 应用，读不到 `application.yml`，Redis 地址靠启动参数传入（`RunnerJedisPoolManager.initRedisConfig`，`RunnerJedisPoolManager.java`）。**两端必须连同一个 Redis 的同一个 db**，否则任务发出去执行机永远收不到。

## Redis key 总表

前缀 `TESTIN_API_BACKEND:` 下表中省略。生成方法都在 `RedisKeyUtil.java`。

### 执行链路核心 key

| key 模式 | 类型 | 写方 → 读方 | 用途 | TTL |
|---|---|---|---|---|
| `JMETER_JMX_ID:{machineId}:{reportId}` | String | 服务端 `TestExecuteService.java` / poller → 执行端 `RunnerTestExecuteService.java` | 任务本体 JMX 字符串。执行端取出执行后主动删除 | 24h |
| `TESTIN_PRO_RUNNER_LINTERER:{machineId}` | List | 服务端 `ServerProducer.java` lpush → 执行端 `RunnerConsumer.java` brpop | **每台执行机专属的任务队列**，元素是 `JmeterRunRequestDTO` 的 fastjson JSON | 无 |
| `TESTIN_PRO_SERVER_LINTERER` | List | 执行端 `RunnerProducer.java` lpush → 服务端 `ServerConsumer.java` brpop | **结果回传队列**（全执行机共用），元素是 `JmeterRunResponseDTO` 的 Jackson JSON | 无 |
| `TASK_STOP_REPORT_MACHINEIDLIST:{reportId}` | List | 服务端 `TestAutomationService.java` 写入 → 停止时 `taskStopper`读取 | 记录该报告被哪些执行机执行，停止时逐台发 HTTP `/stopTask` | 24h |
| `TASK_EXECUTED:BUFFER:{reportId}` | List | `TestExecuteService.java` lpush → `TestTaskBatchPoller.java` rpop | 批次缓冲队列，元素是批次序号 | 无 |
| `TASK_EXECUTED:JMX:{reportId}:{batchIndex}` | String | `TestExecuteService.java` → poller 取出后删除 | 单批 JMX 缓存 | 24h |
| `TASK_EXECUTED:DTO:{reportId}:{batchIndex}` | String | `TestExecuteService.java` → poller 取出后删除 | 单批 DTO 缓存 | 24h |
| `TASK_EXECUTED:ACTIVE_SET` | Set | `TestExecuteService.java` sadd → poller 每 3s 扫描 | 有待下发批次的 reportId 集合 | 无 |
| `TASK_EXECUTED:RUNNING:{reportId}` | String | poller 下发时设置→ 执行端 listener 执行完删除（`TestinResultBackendListener.java`） | "当前批次执行中"标记，存在期间 poller 跳过 | 2h（兜底） |
| `RUNNER_VARIABLES:{reportId}` | String | 服务端本地执行时由 overlay 类 `org.apache.jmeter.threads.JMeterThread` 写入（`src/main/java/org/apache/jmeter/threads/JMeterThread.java`）→ `TestCallApiRunService.java` 读取 | CALLAPI（UI 自动化调接口）执行结束后的变量回传 | 5 分钟 |
| `TESTIN_PRO_RUNNER_BLOCKING_JMETER_JMX_ID:JMETER_JMX_ID:{machineId}` | String | 预留（`RunnerTestExecuteService.checkUnfinishedTasks`） | "执行端重启后优先执行的阻塞任务"，相关代码已注释，**当前未启用** | - |

### 业务缓存 key（与执行链路弱相关）

| key 模式 | 用途 |
|---|---|
| `MODULE:{projectId}:{moduleId}` | 模块树子目录 id 列表缓存（`TestModuleService.java`），TTL 12 分钟 |
| `MODULEID_TO_COUNT:{projectId}:{moduleId}` | 目录下元素个数缓存（`TestCommonService.java`），TTL 1 天 |
| `SKEY_REPORT_ID:{skey}` | 预留：skey→reportId 映射，当前无写入方 |
| `PLAN_EXECUTE:` / `FOLLOW_UP:` / `TASK` / `CASE` | 常量声明在 `RedisKeyUtil.java`，部分已无实际使用方，读代码时以"有没有 get 方法被调用"为准 |

## 通信协议

```
服务端                                                   执行端
  │                                                        │
  │  ① SET  JMETER_JMX_ID:{mid}:{rid} = JMX XML 字符串      │
  │  ② LPUSH RUNNER_LINTERER:{mid} = JmeterRunRequestDTO   │
  │      (fastjson 序列化; 不含 HashTree)                    │
  │ ──────────────────────────────────────────────────────►│
  │                        ③ GET JMETER_JMX_ID:{mid}:{rid} │
  │                        ④ SaveService 反序列化 → 执行     │
  │                                                        │
  │  ⑤ LPUSH SERVER_LINTERER = JmeterRunResponseDTO        │
  │      (Jackson 序列化; realTimeReport=true 按用例实时,    │
  │       =false 整批最终结果)                               │
  │ ◄──────────────────────────────────────────────────────│
```

要点：

- **JMX 与 DTO 分离**：队列消息里只有 DTO（几十 KB 级），JMX（可能几 MB）走独立 key。这避免队列消息过大，但引入"DTO 到了、JMX 没了"的丢失场景（TTL 过期、key 被误删）。
- **DTO 序列化不对称**：下发用 fastjson（`TestExecuteService.java` 的 `JSONObject.toJSONString`），回传用 Jackson（`TestinResultBackendListener.convertToXML`，——方法名叫 XML 但产出是 JSON）。改 DTO 字段时两套序列化对 `null`、枚举的处理差异要留意。
- 队列元素**永远不会被主动清理失败残留**：runner 宕机期间 lpush 进来的 DTO 会堆在 `RUNNER_LINTERER:{machineId}` 里，runner 恢复后会被逐条消费——但此时 JMX key 可能已过期，表现为"消费了但直接 return"（`RunnerTestExecuteService.java`）。

## 为什么是 list + brpop 而不是 pub/sub

`RunnerConsumer` 虽然 `extends JedisPubSub`（`RunnerConsumer.java`，历史遗留），实际通信用的是 **list + brpop(0)**：

1. **不丢消息**：pub/sub 在执行端离线期间的消息直接丢弃；list 会把任务攒在队列里，执行机恢复后继续消费（配合 JMX 24h TTL，接受"延迟执行"而不是"丢失"）。
2. **天然点对点**：每台执行机一个队列 key（`RUNNER_LINTERER:{machineId}`），服务端按 machineId 精确投递，不需要订阅过滤。
3. **背压简单**：执行端一次只 brpop 一条、跑完再取下一条，配合服务端 `TestTaskBatchPoller` 的 RUNNING 标记，天然实现"一台执行机同一时刻只跑一个批次"。
4. brpop(0) 永久阻塞，所以两端的消费线程都持有**独立长连接**（服务端用 `BackendJedisPoolManager` 单独建池、执行端用 `RunnerJedisPoolManager.getJedis()` 新建裸连接），避免占满业务连接池。

## 任务丢失排查指引

按下发顺序逐段检查（redis-cli 直连平台 Redis，注意选对 db）：

1. `LLEN TESTIN_API_BACKEND:TESTIN_PRO_RUNNER_LINTERER:{machineId}` —— 有积压说明执行端没消费（runner 没初始化 machineId、brpop 线程死了、或 runner 连错 Redis）；
2. `GET TESTIN_API_BACKEND:JMETER_JMX_ID:{machineId}:{reportId}` —— 队列空了但 JMX 还在，说明 DTO 被消费但执行没发生（或执行端取 JMX 时 key 已过期，此时执行端日志无报错、任务静默丢弃）；
3. `SMEMBERS TESTIN_API_BACKEND:TASK_EXECUTED:ACTIVE_SET` / `LLEN ...BUFFER:{reportId}` —— 多批任务卡在中间批次：看 `GET ...TASK_EXECUTED:RUNNING:{reportId}` 是否残留（2h TTL 兜底，listener 没删成功时 poller 会一直跳过）；
4. `LLEN TESTIN_API_BACKEND:TESTIN_PRO_SERVER_LINTERER` —— 有积压说明服务端 `ServerConsumer` 的 brpop 线程挂了（看服务端日志有没有持续打"报告结果落库失败"）；
5. 执行端拿不到任务先确认两端连的是**同一个 Redis 实例和 db**（runner 的第 4 个位置参数）。

## 注意事项与坑

1. **连接池配置异味**：`BackendJedisPoolManager` 是 `MAX_TOTAL=20, MAX_IDLE=0, MIN_IDLE=20`（`BackendJedisPoolManager.java`），`maxIdle < minIdle` 在 commons-pool2 里 minIdle 实际被压到 maxIdle；runner 侧同款配置（`RunnerJedisPoolManager.java`）外加 `setLifo(false)`。改池参数前先弄清这三个值的实际生效关系。
2. **runner 的 brpop 用的是裸连接**：`RunnerConsumer.java` 调 `RunnerJedisPoolManager.getJedis()` 每次 new 一个 Jedis（不走池），Redis 重启后这条连接永久失效，`while(true)` 里只会循环打错误日志且不会重建连接——runner 需要重启才能恢复消费。
3. **BUFFER 队列 pop 后数据缺失不会重试**：`TestTaskBatchPoller.java` 发现 JMX/DTO 缓存缺失只打 error 日志，该批次直接丢失，任务会卡在"执行中"直到人工干预。
4. **`SKEY_REPORT_ID` / `RUNNER_BLOCKING` 等 key 是半成品**：有常量与工具方法但写入方被注释或不存在，不要基于它们开发新功能。
5. 所有 key 的 TTL 都是"防 Redis 撑爆"的兜底，**业务正确性不依赖 TTL**；清理逻辑分散在 listener、poller、stopRun 里，新增 key 时记得把三处清理路径都考虑到（特别是 `stopRun` 里的批次缓存清理，`TestAutomationService.java`）。
