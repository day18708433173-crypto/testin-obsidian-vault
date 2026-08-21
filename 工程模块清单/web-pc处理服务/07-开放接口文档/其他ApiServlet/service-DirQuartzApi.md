# service-DirQuartzApi — 目录与任务模版关联管理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/dir/DirQuartzApi.java`（@Service）
> 类型：远端代理（→ RealLogfile 服务）
> 转发方式：V3 REST 路径（`ServiceRemoteV3Api.remotePost/remote/remoteGet`，RestTemplate，基址 `Config.LOG_FILE_URL`，路径常量见 `api/constant/ApiUrl.java`）

## 方法列表

### 1. removeDirQuartz — 删除目录和任务模版的关联关系

```java
public Integer removeDirQuartz(DirQuartzRequestDTO dirQuartzRequestDTO) throws GeneralException
```

**用途**：删除目录下任务模版关联（projectId<=0 或 ids 为空直接返回 0）。

**转发目标**：`POST /v3/core/dir_quartz_job/remove_dir_quartz`

**请求参数**（body `DirQuartzRequestDTO`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 是 | 项目 id，<=0 直接返回 0 |
| ids | List&lt;Integer&gt; | 是 | 关联 id 列表，空直接返回 0 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 删除影响行数 |

**调用者**：`WebQuartz.java`、`McPcQuartz.java`

### 2. addDirQuart — 添加目录和任务模版的关联关系

```java
public Integer addDirQuart(DirQuartzJob dirQuartzJob) throws GeneralException
```

**用途**：新建定时任务/任务模版时绑定到目录；返回 null 抛 `apiInvalid`（"创建任务关联信息失败"）。

**转发目标**：`POST /v3/core/dir_quartz_job/add_dir_quartz_job`

**请求参数**（body `DirQuartzJob`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| dirQuartzJob | DirQuartzJob | 是 | 目录-任务模版关联对象（projectId/dirId/jobId 等） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 新增关联结果 |

**调用者**：`WebQuartz.java`、`McPcQuartz.java`、`TaskServiceImpl.java`

### 3. getDirQuartzJobIds — 获取目录关联的任务模版 ID

```java
public List<Integer> getDirQuartzJobIds(DirInfoDTO dirInfoDTO) throws GeneralException
```

**用途**：按 project_id / id / dir_type 查询目录关联的任务模版 id 列表。

**转发目标**：`GET /v3/core/dir_quartz_job/get_job_ids?project_id=&id=&dir_type=`

**请求参数**（query `DirInfoDTO`，均为可选）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 否 | 项目 id |
| id | Integer | 否 | 目录 id |
| dirType | Integer | 否 | 目录类型 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | List&lt;Integer&gt; | 目录关联的任务模版 id 列表 |

**调用者**：`BaseQuartz.java`

### 4. getRootDirId — 获取根目录 id

```java
public Integer getRootDirId(DirInfoDTO dirInfoDTO) throws GeneralException
```

**用途**：按 project_id + dir_type 查询项目根目录 id，新建任务模版未指定目录时挂到根目录。

**转发目标**：`GET /v3/core/quartz_dir/get_root_dir?project_id=&dir_type=`

**请求参数**（query `DirInfoDTO`，均为可选）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 否 | 项目 id |
| dirType | Integer | 否 | 目录类型 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | Integer | 根目录 id（DirInfoDTO.id） |

**调用者**：`WebQuartz.java`、`McPcQuartz.java`、`TaskServiceImpl.java`

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealLogfile 服务](../../../平台基础功能服务/00-首页.md)
- [service-TestPlanApi](service-TestPlanApi.md)、[service-McPcTaskApi](service-McPcTaskApi.md)
