---
branch: syy.release.z7.8.1.0
module: real-test
type: 接口文档
entry: ApiServlet
---

# Account (service/app)

账号设备匹配与释放的 ApiServlet。

类路径：`real-test/src/main/java/cn/testin/service/app/Account.java`，继承 `GenericBaseService`。

## 本类接口一览

| 接口 | op | 功能 |
| --- | --- | --- |
| match | Account.match | 匹配分配测试账号 |
| release | Account.release | 释放测试账号 |

## match (`Account.match`)

- **实现意图**：为测试任务匹配分配测试账号（如登录账号），从账号池中获取可用账号并锁定。

- **请求参数**：
| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| taskid | String | 否 | 任务 ID（代码未做空校验，直接透传服务层） |
| subtaskid | String | 否 | 子任务 ID |
| deviceid | String | 否 | 设备 ID |

- **返回参数**：
| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 返回码，0 成功，非 0 失败 |
| msg | String | 返回信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | 匹配到的账号信息（`PmrealAdaptAccount`，未匹配到为空对象） |
| data.objInfo.accountId | String | 账号 |
| data.objInfo.accountPwd | String | 密码 |
| data.objInfo.extension | String | 扩展信息 |
| data.objInfo.users | String | 使用者信息 |
| data.objInfo.useNum | Integer | 使用次数 |
| data.objInfo.matchStatus | Integer | 匹配状态 |

- **处理流程**：查询可用账号池 -> 锁定账号 -> 关联 taskId -> 返回账号信息。

- **调用链**：`Account` -> `IAccountService` -> [UserManager](../../../平台基础功能服务/00-首页.md)（账号管理）。

## release (`Account.release`)

- **实现意图**：测试完成后释放账号，解绑 taskId 关联，将账号归还账号池。

- **请求参数**：
| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| taskid | String | 否 | 任务 ID |
| subtaskid | String | 否 | 子任务 ID |
| deviceid | String | 否 | 设备 ID |
| accountId | String | 否 | 账号 ID |

- **返回参数**：
| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 返回码，0 成功，非 0 失败 |
| msg | String | 返回信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 释放结果：1 成功，0 失败 |

- **调用链**：[UserManager](../../../平台基础功能服务/00-首页.md)（账号释放）。
