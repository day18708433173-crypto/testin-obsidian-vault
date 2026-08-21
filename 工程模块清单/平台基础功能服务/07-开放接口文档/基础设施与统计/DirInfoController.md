# DirInfoController -- 目录管理（树/增删改移/任务数统计/根目录）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/dir/DirInfoController.java`
> 类级路由：`/v3/core/quartz_dir`
> Service 接口：`cn.testin.business.interfaces.dir.IDirInfoService`、`IDirQuartzJobService`
> 实现类：`cn.testin.business.impl.dir.DirInfoServiceImpl`、`DirQuartzJobServiceImpl`
> 业务：目录树管理（查询/新增/重命名/移动/删除），以及目录下任务模版数量统计、根目录获取。

## 接口列表总表

| 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|
| GET | `/v3/core/quartz_dir/get_dir_tree` | getDirTree | 获取目录树 | 无 |
| POST | `/v3/core/quartz_dir/add_dir` | addDir | 添加目录 | 无 |
| PUT | `/v3/core/quartz_dir/update_dir_name/{id}` | updateDirName | 更新目录名称 | 无 |
| PUT | `/v3/core/quartz_dir/move_dir/{id}` | moveDir | 移动目录 | 无 |
| DELETE | `/v3/core/quartz_dir/delete_dir/{id}` | deleteDir | 删除目录（级联子目录+任务模版） | 无 |
| GET | `/v3/core/quartz_dir/get_quartz_num` | getQuartzNum | 获取目录下任务模版数量 | 无 |
| GET | `/v3/core/quartz_dir/get_root_dir` | getRootDir | 获取根目录 | 无 |

统一响应包装：`ResponseResult<T>`；列表用 `BaseListResponseDTO`；写操作用 `BaseDataResultDTO { Long result }`；GET 查询接口带 `@UnderlineToCamel`。

---

## 1. GET /v3/core/quartz_dir/get_dir_tree -- 获取目录树

### 入口

`DirInfoController.getDirTree(DirRequestDTO dirRequestDTO)` -- DirInfoController.java（`@UnderlineToCamel`）

### 请求参数（DirRequestDTO，Query 绑定）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID |
| dirType | Integer | 是 | 目录类型（区分定时任务/计划等场景） |
| 其他 | -- | 否 | DirRequestDTO 内其余筛选字段 |

### 响应结构

`ResponseResult<BaseListResponseDTO<DirInfo>>`：目录树列表。

### 返回参数（DirInfo 元素结构）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array\<DirInfo\> | 目录树列表 |
| data.list[].id | Integer | 目录 ID |
| data.list[].projectId | Integer | 项目 ID |
| data.list[].parentDirId | Integer | 父目录 ID（根目录为空/0） |
| data.list[].dirName | String | 目录名称 |
| data.list[].createUserId | Integer | 创建人 ID |
| data.list[].updateUserId | Integer | 更新人 ID |
| data.list[].status | Integer | 状态 |
| data.list[].dirType | Integer | 目录类型 |
| data.list[].planInfoType | Integer | 计划类型 |
| data.list[].dirOrder | Integer | 目录排序 |
| data.list[].createTime | Date | 创建时间 |
| data.list[].updateTime | Date | 更新时间 |
| data.list[].children | Array\<DirInfo\> | 子目录（递归嵌套，同为 DirInfo 结构） |

### 实现意图

按项目+目录类型查出所有目录记录，在 Service 层组装为树形结构返回（children 嵌套）。

### 调用链

```
DirInfoController.getDirTree
└─ DirInfoServiceImpl.getDirTree
   → dir_info 表全量查询 + 树形组装
```

### 涉及表

| 表 | 操作 |
|---|---|
| dir_info | 读 |

---

## 2. POST /v3/core/quartz_dir/add_dir -- 添加目录

### 入口

`DirInfoController.addDir(@RequestBody @Valid DirRequestDTO dirRequestDTO)` -- DirInfoController.java

### 请求参数（DirRequestDTO，JSON Body，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| parentId | Integer | 否 | 父目录ID（为空则在根下创建） |
| dirName | String | 是 | 目录名称 |
| projectId | Integer | 是 | 项目ID |
| dirType | Integer | 是 | 目录类型 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 新建目录主键ID。

### 调用链

```
DirInfoController.addDir
└─ DirInfoServiceImpl.addDir
   → dir_info insert
```

---

## 3. PUT /v3/core/quartz_dir/update_dir_name/{id} -- 更新目录名称

### 入口

`DirInfoController.updateDirName(@PathVariable Integer id, @RequestBody @Valid DirRequestDTO dirRequestDTO)` -- DirInfoController.java

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|---|---|---|---|
| id | Path | 是 | 目录ID |
| dirName | Body | 是 | 新目录名称 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 调用链

```
DirInfoController.updateDirName
└─ DirInfoServiceImpl.updateDirName
   → dir_info update
```

---

## 4. PUT /v3/core/quartz_dir/move_dir/{id} -- 移动目录

### 入口

`DirInfoController.moveDir(@PathVariable Integer id, @RequestBody @Valid DirRequestDTO dirRequestDTO)` -- DirInfoController.java

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|---|---|---|---|
| id | Path | 是 | 被移动目录ID |
| parentId | Body | 是 | 目标父目录ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

修改目录的 `parentId` 实现目录位置变更；需校验目标父目录存在且不会造成循环引用。

### 调用链

```
DirInfoController.moveDir
└─ DirInfoServiceImpl.moveDir
   → dir_info update parentId
```

---

## 5. DELETE /v3/core/quartz_dir/delete_dir/{id} -- 删除目录

### 入口

`DirInfoController.deleteDir(@PathVariable Integer id, DirDeleteRequestDTO dirDeleteRequestDTO)` -- DirInfoController.java（`@UnderlineToCamel`）

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|---|---|---|---|
| id | Path | 是 | 要删除的目录ID |
| projectId / 其他 | Query (DirDeleteRequestDTO) | 是 | 项目ID等校验字段 |

### 响应结构

- 成功：`ResponseResult.success(DirDeleteResponseDTO)`，含删除结果信息。
- 失败：`ResponseResult.error(FAIL_CODE, DirDeleteResponseDTO)`，`data.result == ERROR_CODE` 时返回错误。

### 返回参数（DirDeleteResponseDTO）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 删除结果（= ERROR_CODE 时整体为 error 响应） |
| data.taskInfoWithPlanInfoResponseDTOS | Array\<TaskInfoWithPlanInfoResponseDTO\> | 关联任务/计划信息列表 |
| data.taskInfoWithPlanInfoResponseDTOS[].taskId | Integer | 任务 ID |
| data.taskInfoWithPlanInfoResponseDTOS[].planInfos | Array\<PlanInfoResponseDTO\> | 关联计划列表（字段见 PlanInfoController） |

### 实现意图

删除目录并级联删除其下所有子目录及关联的任务模版。Controller 层对 Service 返回的 `result == ERROR_CODE` 做判断，当 result 等于 `Constants.ERROR_CODE` 时包装为 error 响应，否则 normal success。

### 调用链

```
DirInfoController.deleteDir
└─ DirInfoServiceImpl.deleteDir
   → dir_info 级联软删 + dir_quartz_job 关联清理
```

### 涉及表

| 表 | 操作 |
|---|---|
| dir_info | 写（级联软删/物理删） |
| dir_quartz_job | 写（清理关联） |

---

## 6. GET /v3/core/quartz_dir/get_quartz_num -- 获取目录下任务模版数量

### 入口

`DirInfoController.getQuartzNum(@RequestParam Integer projectId, @RequestParam Integer dirType, @RequestParam(required=false) Integer planInfoType)` -- DirInfoController.java

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目ID |
| dir_type | Integer | 是 | 目录类型 |
| plan_info_type | Integer | 否 | 计划类型（过滤特定计划类型的模版数） |

### 响应结构

`ResponseResult<DirWithCountResponseDTO>`：各目录节点及其下的任务模版数量。

### 返回参数（DirWithCountResponseDTO）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.quartzNum | Map\<Integer, Integer\> | 目录 ID → 任务模版数量的映射（HashMap） |

### 实现意图

统计每个目录下关联的任务模版（quartz job）数量，按目录树结构返回带计数的 DTO。实际委托给 `IDirQuartzJobService.getQuartzNum`。

### 调用链

```
DirInfoController.getQuartzNum
└─ DirQuartzJobServiceImpl.getQuartzNum
   → dir_info + dir_quartz_job 联查计数
```

### 涉及表

| 表 | 操作 |
|---|---|
| dir_info | 读 |
| dir_quartz_job | 读（计数） |

---

## 7. GET /v3/core/quartz_dir/get_root_dir -- 获取根目录

### 入口

`DirInfoController.getRootDir(DirRequestDTO dirRequestDTO)` -- DirInfoController.java（`@UnderlineToCamel`）

### 请求参数（DirRequestDTO，Query 绑定）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID |
| dirType | Integer | 是 | 目录类型 |

### 响应结构

`ResponseResult<DirInfo>`：根目录信息（单条记录，非列表）。若项目下无根目录则可能返回 null 或自动创建。`data` 字段结构同「get_dir_tree」的 `DirInfo`（含 id/projectId/parentDirId/dirName/dirType/children 等）。

### 实现意图

按项目和目录类型查找根目录（`parentId` 为 null/0 的顶层节点）；为上层业务提供起始节点。

### 调用链

```
DirInfoController.getRootDir
└─ DirInfoServiceImpl.getRootDirInfo
   → dir_info 查 parentId 为空的记录
```

### 涉及表

| 表 | 操作 |
|---|---|
| dir_info | 读 |

---

## 备注

- 本 Controller 同时注入了 `IDirInfoService` 和 `IDirQuartzJobService`，目录结构相关委托前者，模版关联相关委托后者。
- `deleteDir` 在 Controller 层对 result 做了 ERROR_CODE 判断，将部分 Service 返回结果转为 error 响应（而非抛异常）。
- 目录树为通用抽象，同时服务于定时任务模板（quartz job）和测试计划两个场景，通过 `dirType` 区分。
- DTO 定义见 `cn.testin.dto.request.dir` 和 `cn.testin.dto.response.dir` 包。

相关文档：[00-分支索引](00-分支索引.md) · [DirQuartzJobController](DirQuartzJobController.md)
