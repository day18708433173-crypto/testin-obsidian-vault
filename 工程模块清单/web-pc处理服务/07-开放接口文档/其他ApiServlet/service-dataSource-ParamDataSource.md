# service-dataSource-ParamDataSource — 参数化数据源数据查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/dataSource/ParamDataSource.java`（继承 `GenericBaseService`）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=dataSource, op=ParamDataSource.<方法>
> 本地注入：`IParamDataSourceService`（另持有一个默认配置的 `JedisPool`，当前方法未使用）

## 方法列表

### 1. getParamTableData — 获取参数表数据

```java
public String getParamTableData(ApiRequest apirequest) throws Exception
```

**用途**：提测/任务详情页查询参数化数据源在当前脚本、标签过滤下的可用数据行。

**流程**：
1. 校验 sourceId、projectId、scriptNo 必填
2. 可选参数：tagList / skipTagList（Gson 反序列化为 `List<Integer>`）、oldTaskId、scriptStatus 数组
3. `IParamDataSourceService.getParamTableData(projectId, sourceId, scriptNo, tagList, skipTagList, oldTaskId, scriptStatus)` 查询返回数据行列表

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sourceId | String | 是 | 数据源ID |
| projectId | Integer | 是 | 项目组ID |
| scriptNo | Integer | 是 | 脚本编号 |
| tagList | JSONArray | 否 | 标签ID列表（List&lt;Integer&gt;） |
| skipTagList | JSONArray | 否 | 排除标签ID列表（List&lt;Integer&gt;） |
| oldTaskId | String | 否 | 旧任务ID |
| scriptStatus | JSONArray | 否 | 脚本状态数组（List&lt;Integer&gt;） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | JSONArray | 参数表数据行列表 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| Remote DataSource（经 IParamDataSourceService） | 参数表数据 | 读 |

### 2. getDefaultDataSourceParam — 获取默认数据源参数分配

```java
public String getDefaultDataSourceParam(ApiRequest apirequest) throws Exception
```

**用途**：按策略（paramStrategy）为脚本×设备组合预分配默认数据源参数。

**流程**：
1. 校验 sourceId（转 parentId）、projectId、eid、paramStrategy、scriptNos、devices 均必填
2. 可选 tagList（Gson 反序列化）
3. `IParamDataSourceService.getDefaultDataSourceParam(eid, projectId, parentId, scriptNos, devices, paramStrategy, tagList)` 计算分配结果
4. 返回时 dataMap 经 fastjson `toJSON` 二次包装后放入 RES_DATA

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sourceId | Integer | 是 | 数据源ID（转为 parentId） |
| projectId | Integer | 是 | 项目组ID |
| eid | Integer | 是 | 企业ID |
| paramStrategy | Integer | 是 | 分配策略 |
| scriptNos | JSONArray | 是 | 脚本编号数组 |
| devices | JSONArray | 是 | 设备数组 |
| tagList | JSONArray | 否 | 标签ID列表（List&lt;Integer&gt;） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | JSONArray | 默认数据源参数分配结果 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| Remote DataSource（经 IParamDataSourceService） | 参数表数据 | 读 |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-task-Task](service-task-Task.md)（detail 中补充数据源标签名称）
- [外部服务索引](../../../外部服务/外部服务-索引.md)
