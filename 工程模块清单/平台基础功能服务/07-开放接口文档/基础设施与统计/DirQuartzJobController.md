# DirQuartzJobController -- 目录-任务模版关联管理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/dir/DirQuartzJobController.java`
> 类级路由：`/v3/core/dir_quartz_job`
> Service 接口：`cn.testin.business.interfaces.dir.IDirQuartzJobService`
> 实现类：`cn.testin.business.impl.dir.DirQuartzJobServiceImpl`
> 业务：管理目录（dir_info）与任务模版（quartz job）之间的多对多关联关系，支持添加关联、批量移位、批量删除、双向查询。

## 接口列表总表

| 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|
| POST | `/v3/core/dir_quartz_job/add_dir_quartz_job` | addDirQuartzJob | 添加任务模版与目录关联 | 无 |
| POST | `/v3/core/dir_quartz_job/update_job_position` | updateJobPosition | 批量修改任务模版关联目录位置 | 无 |
| POST | `/v3/core/dir_quartz_job/remove_dir_quartz` | removeDirQuartz | 批量删除任务模版与目录关联 | 无 |
| GET | `/v3/core/dir_quartz_job/get_job_ids` | getJobIds | 按目录查询关联的任务模版ID列表 | 无 |
| GET | `/v3/core/dir_quartz_job/get_dir_ids` | getDirIds | 按任务模版查询关联的目录ID列表 | 无 |

统一响应包装：`ResponseResult<T>`；写操作返回 `BaseDataResultDTO { Long result }`；GET 查询接口带 `@UnderlineToCamel`。

---

## 1. POST /v3/core/dir_quartz_job/add_dir_quartz_job -- 添加关联

### 入口

`DirQuartzJobController.addDirQuartzJob(@RequestBody DirQuartzJob dirQuartzJob)` -- DirQuartzJobController.java

### 请求参数（DirQuartzJob，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| dirId | Integer | 是 | 目录ID |
| jobId | Integer | 是 | 任务模版（定时Job）ID |
| projectId | Integer | 是 | 项目ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 新增关联主键ID。

### 调用链

```
DirQuartzJobController.addDirQuartzJob
└─ DirQuartzJobServiceImpl.addDirQuartzJob
   → dir_quartz_job insert
```

### 涉及表

| 表 | 操作 |
|---|---|
| dir_quartz_job | 写（insert） |

---

## 2. POST /v3/core/dir_quartz_job/update_job_position -- 批量修改关联位置

### 入口

`DirQuartzJobController.updateJobPosition(@RequestBody DirQuartzJobRequestDTO dirQuartzJobRequestDTO)` -- DirQuartzJobController.java

### 请求参数（DirQuartzJobRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | List\<Integer\> | 是 | 要移动的任务模版ID列表 |
| targetDirId | Integer | 是 | 目标目录ID |
| projectId | Integer | 是 | 项目ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数（批量 update 行数）。

### 实现意图

将一批任务模版从当前目录移动到目标目录：按 jobIds 批量更新 `dir_quartz_job.dir_id` 为目标目录ID。Service 层调用 `batchUpdateJobPosition`。

### 调用链

```
DirQuartzJobController.updateJobPosition
└─ DirQuartzJobServiceImpl.batchUpdateJobPosition
   → dir_quartz_job 批量 update dirId
```

---

## 3. POST /v3/core/dir_quartz_job/remove_dir_quartz -- 批量删除关联

### 入口

`DirQuartzJobController.removeDirQuartz(@RequestBody DirQuartzUpdateRequestDTO dirQuartzUpdateRequestDTO)` -- DirQuartzJobController.java

### 请求参数（DirQuartzUpdateRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| ids | List\<Integer\> | 是 | 要移除关联的任务模版ID列表（`getIds()` 判空，空则抛异常） |
| projectId | Integer | 否 | 项目ID |
| dirType | Integer | 否 | 目录类型 |
| planInfoType | Integer | 否 | 计划类型 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 删除行数。

### 实现意图

批量解除指定任务模版与目录的关联关系，按 jobIds 删除 `dir_quartz_job` 记录。注意：仅删除关联关系，不删除目录也不删除任务模版本身。

### 调用链

```
DirQuartzJobController.removeDirQuartz
└─ DirQuartzJobServiceImpl.batchRemoveDirQuartzByJobIds
   → dir_quartz_job 批量 delete
```

---

## 4. GET /v3/core/dir_quartz_job/get_job_ids -- 按目录查任务模版ID

### 入口

`DirQuartzJobController.getJobIds(DirInfo dirInfo)` -- DirQuartzJobController.java（`@UnderlineToCamel`）

### 请求参数（DirInfo，Query 绑定）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 是 | 目录ID |
| projectId | Integer | 是 | 项目ID |

### 响应结构

`ResponseResult<QuartzJobIdResponseDTO>`，`data.result` = 关联的 jobId 列表。

### 返回参数（QuartzJobIdResponseDTO）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Array\<Integer\> | 关联的任务模版 ID 列表 |

### 实现意图

根据目录信息查询该目录下所有关联的任务模版ID，用于展示目录内容或在删除/移动目录前做引用检查。

### 调用链

```
DirQuartzJobController.getJobIds
└─ DirQuartzJobServiceImpl.getDirQuartzJobIds
   → dir_quartz_job 按 dirId 查询
```

---

## 5. GET /v3/core/dir_quartz_job/get_dir_ids -- 按任务模版查目录ID

### 入口

`DirQuartzJobController.getDirIds(DirQuartzJobRequestDTO dirQuartzJobRequestDTO)` -- DirQuartzJobController.java（`@UnderlineToCamel`）

### 请求参数（DirQuartzJobRequestDTO，Query 绑定）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | List\<Integer\> | 是 | 任务模版ID列表 |
| projectId | Integer | 是 | 项目ID |

### 响应结构

`ResponseResult<QuartzJobIdResponseDTO>`，`data.result` = 关联的 dirId 列表。

### 返回参数（QuartzJobIdResponseDTO）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Array\<Integer\> | 关联的目录 ID 列表 |

### 实现意图

反向查询：给定一批任务模版ID，查出它们分别属于哪些目录。用于模板管理侧展示模板所在目录。

### 调用链

```
DirQuartzJobController.getDirIds
└─ DirQuartzJobServiceImpl.getDirByJobId
   → dir_quartz_job 按 jobIds 查询
```

---

## 备注

- 目录-任务模版关系为多对多：一个目录可含多个模版，一个模版可被放入多个目录（实际使用中通常一对一）。
- 所有写操作返回 `BaseDataResultDTO`，`result` 为影响行数（Long 类型）。
- `getJobIds` 入参直接使用 `DirInfo` 实体作为 Query 绑定（不单独定义 DTO），字段通过 `@UnderlineToCamel` 转换。
- 本 Controller 的批量操作（移位/删除）同时服务于目录管理（`DirInfoController.deleteDir` 级联删除时调用）和测试计划复制（`PlanInfoController.copyTestPlan` 复制目录关系时调用）。

相关文档：[00-分支索引](00-分支索引.md) · [DirInfoController](DirInfoController.md) · [QuartzController](QuartzController.md)
