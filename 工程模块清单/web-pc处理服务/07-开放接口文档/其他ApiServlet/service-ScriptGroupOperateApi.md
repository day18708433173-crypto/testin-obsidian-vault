# service-ScriptGroupOperateApi — 脚本组查询与保存代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/script/ScriptGroupOperateApi.java`（@Component，继承 `AbstractApi`）
> 类型：远端代理（→ Script 服务）
> 转发方式：V1 ApiServlet 前缀 `Script`，`action=script`，op 为 `ScriptGroup.get/add`
> 说明：异常仅 `print()` 记日志不抛出，失败时返回 null/0

## 方法列表

### 1. getScriptGroupById — 按组 id 查询脚本组

```java
public cn.testin.realweb.pojo.taskDetail.ScriptGroup getScriptGroupById(int groupId, int projectId)
```

**转发目标**：`op=ScriptGroup.get`，data 含 `groupId/projectId`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| groupId | int | 是 | 脚本组 id |
| projectId | int | 是 | 项目 id |

**返回参数**：`pojo.taskDetail.ScriptGroup`（注意与 file.script.ScriptGroupApi 返回的 `api.bean.file.ScriptGroup` 非同类型）

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | ScriptGroup | 脚本组对象（字段见 Script 服务，代码未确认） |

**调用者**：
- `WebQuartz.java / 733` — Web 定时任务取脚本组
- `McPcQuartz.java / 785` — PC 定时任务取脚本组

### 2. saveScriptGroup — 保存脚本组

```java
public int saveScriptGroup(file.ScriptGroup scriptGroup, Integer scriptType)
```

**转发目标**：`op=ScriptGroup.add`；提交前改写对象：`status=1`、`projectid←projectId`、`groupScriptidArray←scriptidArray`、`groupScriptnoArray←scriptnoArray`、附加 `scriptType`、移除 `groupId`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| scriptGroup | file.ScriptGroup | 是 | 脚本组对象（projectId/scriptidArray/scriptnoArray 等） |
| scriptType | Integer | 否 | 脚本类型 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | int | 新增组 id（`result` 强转 int），失败返回 0 |

**调用者**：
- `WebQuartz.java` / `McPcQuartz.java` — 定时任务按脚本快照重建脚本组

## 相关文档

- [00-分支索引](00-分支索引.md)
- [Script](../../../脚本服务/00-首页.md)
- [service-ScriptGroupApi](service-ScriptGroupApi.md)（同 op 的另一封装）
- [WebQuartz](WebQuartz.md) / [McPcQuartz](McPcQuartz.md)
