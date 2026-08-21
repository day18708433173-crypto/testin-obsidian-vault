# service-Enterprise — 企业接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/Enterprise.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口，通过 `action=user` + `op=Enterprise.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "Enterprise.方法名", "action": "user", "data": {...}}`
> 业务：企业信息 CRUD、企业成员管理、企业审核、企业用户关系维护等。

## op 列表总表

| #   | op                                             | 方法名                                 | 说明                 |
| --- | ---------------------------------------------- | ----------------------------------- | ------------------ |
| 1   | Enterprise.summary                             | summary                             | 企业摘要统计             |
| 2   | Enterprise.modifyEnterprise                    | modifyEnterprise                    | 修改企业信息和扩展信息        |
| 3   | Enterprise.getViewEnterpriseUser               | getViewEnterpriseUser               | 查询企业管理员账户          |
| 4   | Enterprise.getEnterprise                       | getEnterprise                       | 查询单个企业详情           |
| 5   | Enterprise.getUserEnterpriseList               | getUserEnterpriseList               | 查询用户归属的企业列表（分页）    |
| 6   | Enterprise.modifyAccessTime                    | modifyAccessTime                    | 更新企业访问时间（切企业触发）    |
| 7   | Enterprise.listByCondition                     | listByCondition                     | 后台按条件分页查企业列表       |
| 8   | Enterprise.getViewEnterpriseByEid              | getViewEnterpriseByEid              | 查企业视图详情            |
| 9   | Enterprise.enterpriseAuth                      | enterpriseAuth                      | 企业审核               |
| 10  | Enterprise.insertEnterpriseUser                | insertEnterpriseUser                | 添加企业用户（用户+关系一并写入）  |
| 11  | Enterprise.initUser                            | initUser                            | 争锋专用新建用户           |
| 12  | Enterprise.insertEnterpriseUserRelation        | insertEnterpriseUserRelation        | 添加企业用户关系           |
| 13  | Enterprise.updateEnterpriseUser                | updateEnterpriseUser                | 根据企业关系更新用户信息       |
| 14  | Enterprise.getEnterpriseUserRelationList       | getEnterpriseUserRelationList       | 按条件查询企业用户关系列表      |
| 15  | Enterprise.lastRelation                        | lastRelation                        | 按访问时间获取最后一条企业用户关系  |
| 16  | Enterprise.activeMember                        | activeMember                        | 激活企业成员             |
| 17  | Enterprise.modifyEnterpriseUserInfo            | modifyEnterpriseUserInfo            | 修改企业成员信息           |
| 18  | Enterprise.modifyEnterpriseUserInfoNew         | modifyEnterpriseUserInfoNew         | 修改企业成员信息（含项目列表）    |
| 19  | Enterprise.deleteEnterpriseUserInfo            | deleteEnterpriseUserInfo            | 删除未确认的企业成员         |
| 20  | Enterprise.deleteEnterpriseUserProjectRelation | deleteEnterpriseUserProjectRelation | 删除企业-项目-角色关联       |
| 21  | Enterprise.getprojectManager                   | getprojectManager                   | 获取项目组管理员           |
| 22  | Enterprise.nextEnterpriseInfo                  | nextEnterpriseInfo                  | 获取下一条企业信息（按更新时间游标） |
| 23  | Enterprise.changeEnterpriseAdmin               | changeEnterpriseAdmin               | 变更企业管理员            |
| 24  | Enterprise.initUserInfo                        | initUserInfo                        | 新建用户（含项目关联列表）      |
| 25  | Enterprise.roleList                            | roleList                            | 查询企业角色列表           |
| 26  | Enterprise.add                                 | add                                 | 新建企业（注册）           |
| 27  | Enterprise.getByThirdPartyEid                  | getByThirdPartyEid                  | 根据第三方企业 ID 查企业信息   |

---

## 返回结构约定

所有方法返回 V1 信封：`{ error_code, msg, data }`。`data` 内按方法不同含以下键之一：

| 键 | 类型 | 说明 |
|---|---|---|
| result | Integer/Boolean/String | 结果值（影响行数、主键、布尔结果等），具体见各方法「响应」 |
| object | Object | 单个对象 |
| list | Array | 列表 |
| page / pageSize / totalRow / totalPage | Integer | 分页信息（list 伴随） |

各方法 `object`/`list` 内部字段见对应「响应」小节。

---

## 1. op=Enterprise.summary — 企业摘要统计

### 请求格式
{"op": "Enterprise.summary", "action": "user", "data": {"eid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID |

### 核心逻辑

查询企业下的项目数、成员数等摘要统计。

### 响应

`{ error_code, msg, data: { object: { ...统计Map } } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 企业摘要统计 Map（字段代码未确认） |

---

## 2. op=Enterprise.modifyEnterprise — 修改企业

### 请求格式
{"op": "Enterprise.modifyEnterprise", "action": "user", "data": {"enterpriseInfo": {...}, "enterpriseExpand": {...}, "userInfo": {...}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| enterpriseInfo | JSONObject | 是 | 企业基本信息 `DbEnterpriseInfo` |
| enterpriseExpand | JSONObject | 否 | 企业扩展信息 `DbEnterpriseExpand` |
| userInfo | JSONObject | 否 | 用户信息（含 `newEmail` 可更新邮箱）；修改后发通知清除缓存 |

### 核心逻辑

修改企业基本信息和扩展信息，可选同步更新管理员用户信息。唯一键冲突时返回 `duplicateKey`。

### 响应

`{ error_code, msg, data: { result } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 影响行数 |

---

## 3. op=Enterprise.getViewEnterpriseUser — 企业管理员账户

### 请求格式
{"op": "Enterprise.getViewEnterpriseUser", "action": "user", "data": {"eid": ..., "fullName": "...", "shortName": "..."}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 三选一 | 企业 ID |
| fullName + shortName | String | 三选一 | 企业全称+简称（必须成对传入） |

### 核心逻辑

查询企业的管理员用户视图数据。fullName/shortName 必须配对传入。

### 响应

`{ error_code, msg, data: { object: { eid, enterpriseShortname, enterpriseDisplayname, enterpriseCreatetime, enterpriseFullname, userEmail, userMobile, userName, userid } } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 企业管理员账户对象 |
| data.object.eid | Integer | 企业 ID |
| data.object.enterpriseShortname | String | 企业简称 |
| data.object.enterpriseDisplayname | String | 企业显示名 |
| data.object.enterpriseCreatetime | Long | 企业创建时间 |
| data.object.enterpriseFullname | String | 企业全称 |
| data.object.userEmail | String | 管理员邮箱 |
| data.object.userMobile | String | 管理员手机 |
| data.object.userName | String | 管理员姓名 |
| data.object.userid | Integer | 管理员用户 ID |

---

## 4. op=Enterprise.getEnterprise — 企业详情

### 请求格式
{"op": "Enterprise.getEnterprise", "action": "user", "data": {"eid": ..., "fullName": "...", "shortName": "..."}}

### 请求参数

同 `getViewEnterpriseUser`。

### 核心逻辑

查询 `db_enterprise` 基本信息，含 `webSite`、`descr`。

### 响应

`{ error_code, msg, data: { object: { eid, enterpriseShortname, enterpriseFullname, enterpriseDisplayname, enterpriseCreatetime, webSite, descr } } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 企业详情对象 |
| data.object.eid | Integer | 企业 ID |
| data.object.enterpriseShortname | String | 企业简称 |
| data.object.enterpriseFullname | String | 企业全称 |
| data.object.enterpriseDisplayname | String | 企业显示名 |
| data.object.enterpriseCreatetime | Long | 企业创建时间 |
| data.object.webSite | String | 企业网站 |
| data.object.descr | String | 企业描述 |

---

## 5. op=Enterprise.getUserEnterpriseList — 用户的企业列表

### 请求格式
{"op": "Enterprise.getUserEnterpriseList", "action": "user", "data": {"userid": ..., "page": ..., "pageSize": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userid | Integer | 否 | 用户 ID |
| page / pageSize | Integer | 否 | 分页 |

### 核心逻辑

分页查询用户归属的全部企业。

### 响应

`{ error_code, msg, data: { list: [...], page, pageSize, totalRow, totalPage } }` — 每条含 `eid`、`enterpriseShortname`、`enterpriseFullname`、`enterpriseDisplayname`、`enterpriseCreatetime`、`enterpriseAccesstime`、`estatus`、`echannel`。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 企业列表 |
| data.list[].eid | Integer | 企业 ID |
| data.list[].enterpriseShortname | String | 企业简称 |
| data.list[].enterpriseFullname | String | 企业全称 |
| data.list[].enterpriseDisplayname | String | 企业显示名 |
| data.list[].enterpriseCreatetime | Long | 企业创建时间 |
| data.list[].enterpriseAccesstime | Long | 企业访问时间 |
| data.list[].estatus | Integer | 企业状态 |
| data.list[].echannel | String | 企业渠道 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

---

## 6. op=Enterprise.modifyAccessTime — 更新企业访问时间

### 请求格式
{"op": "Enterprise.modifyAccessTime", "action": "user", "data": {"eid": ..., "userid": ..., "sid": "..."}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| userid | Integer | 否 | 用户 ID |
| sid | String | 否 | 令牌 |

### 核心逻辑

更新用户对企业的最新访问时间到 `db_enterprise_user_relation.e_access_time` 和 `db_online.eid`。

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否更新成功 |

---

## 7. op=Enterprise.listByCondition — 后台企业列表

### 请求格式
{"op": "Enterprise.listByCondition", "action": "user", "data": {"fullName": "...", "page": ..., "pageSize": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| fullName | String | 否 | 企业全称 |
| userEmail | String | 否 | 用户邮箱 |
| userMobile | String | 否 | 用户手机 |
| loginMobile | String | 否 | 登录手机 |
| status | Integer | 否 | 状态 |
| channel | String | 否 | 渠道 |
| updateTimeStart / updateTimeEnd | Long | 否 | 更新时间范围 |
| page / pageSize | Integer | 否 | 分页 |

### 核心逻辑

后台管理端按多个条件组合查询企业视图列表。

### 响应

`{ error_code, msg, data: { list: [...], page, pageSize, totalRow, totalPage } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 企业视图列表（字段代码未确认） |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

---

## 8. op=Enterprise.getViewEnterpriseByEid — 企业视图详情

### 请求格式
{"op": "Enterprise.getViewEnterpriseByEid", "action": "user", "data": {"eid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID |

### 响应

`{ error_code, msg, data: { object: ViewEnterpriseInfo } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 企业视图对象（ViewEnterpriseInfo，字段代码未确认） |

---

## 9. op=Enterprise.enterpriseAuth — 企业审核

### 请求格式
{"op": "Enterprise.enterpriseAuth", "action": "user", "data": {"expand": {...}, "record": {...}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| expand | JSONObject | 是 | `DbEnterpriseExpand` |
| record | JSONObject | 是 | `DbEnterpriseAuthRecord` |

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 审核是否成功 |

---

## 10. op=Enterprise.insertEnterpriseUser — 添加企业用户

### 请求格式
{"op": "Enterprise.insertEnterpriseUser", "action": "user", "data": {"userInfo": {...}, "relationList": [...]}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userInfo | JSONObject | 是 | `DbUserInfo`，email 必填 |
| relationList | JSONArray | 是 | `DbEnterpriseUserRelation` 数组 |
| ip | String | 否 | IP 地址 |
| userId | Integer | 否 | 用户 ID |

### 核心逻辑

同时写入用户信息和多条企业-项目-用户关联关系。0=成功，1=用户已存在， -1=其他异常。

### 响应

`{ error_code, msg, data: { result: Integer } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 0 成功 / 1 用户已存在 / -1 其他异常 |

---

## 11. op=Enterprise.initUser — 争锋专用新建用户

### 请求格式
{"op": "Enterprise.initUser", "action": "user", "data": {"email": "...", "name": "...", "mobile": "...", "eid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| email | String | 是 | 邮箱 |
| name | String | 否 | 姓名（缺省用 email） |
| mobile | String | 否 | 手机号 |
| eid | Integer | 是 | 企业 ID |

### 核心逻辑

使用 OEM 默认密码创建用户，关联到企业下 projectid=0 的 CREW 角色。同时更新默认角色。

### 响应

`{ error_code, msg, data: { result: userId(成功) / 0/1/-1 } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 成功返回 userId；0/1/-1 表示已存在或异常 |

---

## 12. op=Enterprise.insertEnterpriseUserRelation — 添加企业用户关系

### 请求格式
{"op": "Enterprise.insertEnterpriseUserRelation", "action": "user", "data": {"relation": {...}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| relation | JSONObject | 是 | `DbEnterpriseUserRelation` |

### 核心逻辑

单独在企业-用户-项目关联表中插入一条关系记录。

### 响应

`{ error_code, msg, data: { result } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 影响行数 |

---

## 13. op=Enterprise.updateEnterpriseUser — 根据关联更新用户

### 请求格式
{"op": "Enterprise.updateEnterpriseUser", "action": "user", "data": {"relation": {...}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| relation | JSONObject | 是 | `DbEnterpriseUserRelation` |

### 核心逻辑

通过企业用户关系中的字段同步更新 `db_user` 中的用户信息。

### 响应

`{ error_code, msg }`（无 data 包装）

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |

---

## 14. op=Enterprise.getEnterpriseUserRelationList — 企业用户关系列表

### 请求格式
{"op": "Enterprise.getEnterpriseUserRelationList", "action": "user", "data": {"eid": ..., "projectid": ..., "userid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| projectid | Integer | 否 | 项目 ID |
| userid | Integer | 否 | 用户 ID |
| userStatus | Integer | 否 | 用户状态 |

### 响应

`{ error_code, msg, data: { list: [DbEnterpriseUserRelation, ...] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 企业用户关系列表（DbEnterpriseUserRelation） |
| data.list[].eid | Integer | 企业 ID |
| data.list[].projectid | Integer | 项目 ID |
| data.list[].userid | Integer | 用户 ID |
| data.list[].roleid | Integer | 角色 ID |
| data.list[].userName | String | 用户姓名 |
| data.list[].userMobile | String | 用户手机 |
| data.list[].userJob | String | 用户职位 |
| data.list[].userStatus | Integer | 用户状态 |
| data.list[].eAccessTime | Long | 企业访问时间 |
| data.list[].pAccessTime | Long | 项目访问时间 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].updateTime | Long | 更新时间 |

---

## 15. op=Enterprise.lastRelation — 最后一条企业用户关系

### 请求格式
{"op": "Enterprise.lastRelation", "action": "user", "data": {"eid": ..., "projectid": ..., "userid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| projectid | Integer | 否 | 项目 ID |
| userid | Integer | 否 | 用户 ID |

### 核心逻辑

按访问时间降序取最后一条关联记录。

### 响应

`{ error_code, msg, data: { object: DbEnterpriseUserRelation } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 关联对象（DbEnterpriseUserRelation，字段同方法 14 的 list 元素） |

---

## 16. op=Enterprise.activeMember — 激活企业成员

### 请求格式
{"op": "Enterprise.activeMember", "action": "user", "data": {"eid": ..., "userid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| userid | Integer | 否 | 用户 ID |

### 核心逻辑

激活企业下的成员状态，成功后发缓存清除通知。

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否激活成功 |

---

## 17. op=Enterprise.modifyEnterpriseUserInfo — 修改企业成员信息

### 请求格式
{"op": "Enterprise.modifyEnterpriseUserInfo", "action": "user", "data": {"relation": {...}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| relation | JSONObject | 是 | `DbEnterpriseUserRelation`（含更新字段） |

### 核心逻辑

修改企业-用户关联中的成员信息（姓名、手机号、角色等），发缓存通知。

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否修改成功 |

---

## 18. op=Enterprise.modifyEnterpriseUserInfoNew — 修改企业成员信息（新）

### 请求格式
{"op": "Enterprise.modifyEnterpriseUserInfoNew", "action": "user", "data": {"relation": {...}, "projectIdList": [...]}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| relation | JSONObject | 是 | `DbEnterpriseUserRelation` |
| projectIdList | JSONArray | 否 | 项目 ID 列表（批量更新多项目关联） |

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否修改成功 |

---

## 19. op=Enterprise.deleteEnterpriseUserInfo — 删除企业成员

### 请求格式
{"op": "Enterprise.deleteEnterpriseUserInfo", "action": "user", "data": {"eid": ..., "userid": ..., "projectid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| userid | Integer | 否 | 用户 ID |
| projectid | Integer | 否 | 项目 ID |

### 核心逻辑

删除企业用户关联记录，发缓存清除通知。

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否删除成功 |

---

## 20. op=Enterprise.deleteEnterpriseUserProjectRelation — 删除企业项目角色关联

### 请求格式
{"op": "Enterprise.deleteEnterpriseUserProjectRelation", "action": "user", "data": {"eid": ..., "userid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| userid | Integer | 否 | 用户 ID |

### 核心逻辑

清除该用户在企业下所有项目角色关联。

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否删除成功 |

---

## 21. op=Enterprise.getprojectManager — 项目组管理员

### 请求格式
{"op": "Enterprise.getprojectManager", "action": "user", "data": {"eid": ..., "projectid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID |
| projectid | Integer | 是 | 项目 ID |

### 响应

`{ error_code, msg, data: { result: [ViewEnterpriseUser, ...] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Array | 项目组管理员列表（ViewEnterpriseUser） |
| data.result[].eid | Integer | 企业 ID |
| data.result[].userid | Integer | 用户 ID |
| data.result[].userEmail | String | 邮箱 |
| data.result[].userName | String | 姓名 |
| data.result[].userMobile | String | 手机 |

---

## 22. op=Enterprise.nextEnterpriseInfo — 下一条企业信息

### 请求格式
{"op": "Enterprise.nextEnterpriseInfo", "action": "user", "data": {"updateTime": ..., "status": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| updateTime | Long | 是 | 游标时间戳 |
| status | Integer | 否 | 状态筛选 |

### 核心逻辑

按更新时间游标获取下一条企业信息（用于增量同步）。

### 响应

`{ error_code, msg, data: { object: ViewEnterpriseInfo } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 企业视图对象（ViewEnterpriseInfo，字段代码未确认） |

---

## 23. op=Enterprise.changeEnterpriseAdmin — 变更企业管理员

### 请求格式
{"op": "Enterprise.changeEnterpriseAdmin", "action": "user", "data": {"eid": ..., "submitter": ..., "userid": ..., "roleid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID |
| submitter | Integer | 是 | 当前管理员 ID（将被降级为 CREW） |
| userid | Integer | 是 | 新任管理员 ID |
| roleid | Integer | 是 | 角色 ID |

### 核心逻辑

将 submitter 所有企业下角色改为 CREW；将 userid 在企业下所有关联改为管理员角色，并补全缺失的项目关联；更新默认角色表。

### 响应

`{ error_code, msg, data: { result: "true新增管理员项目的条数：" + count } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | String | 结果描述："true新增管理员项目的条数：" + 条数 |

---

## 24. op=Enterprise.initUserInfo — 新建用户（含项目列表）

### 请求格式
{"op": "Enterprise.initUserInfo", "action": "user", "data": {"eid": ..., "userInfo": {...}, "projectList": [...]}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID（缺省从在线信息取） |
| userInfo | JSONObject | 是 | `{ email(必填), name, mobile, job, thirdPartyUserid, defaultRoleId }` |
| projectList | JSONArray | 否 | `[{ projectid, roleid }]` 或 `[{ thirdPartyProjectid, roleid }]` |

### 核心逻辑

创建用户并关联到指定项目组，支持 `projectid` 和 `thirdPartyProjectid` 两种项目定位方式。默认角色为 CREW。写入 `db_user` 和 `db_enterprise_user_relation`。

### 响应

`{ error_code, msg, data: { result: userId(已存在则返回0) / 1(新用户但已存在) / -1 } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 成功返回 userId；0 已存在 / 1 新用户但已存在 / -1 异常 |

---

## 25. op=Enterprise.roleList — 企业角色列表

### 请求格式
{"op": "Enterprise.roleList", "action": "user", "data": {"eid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID |

### 响应

`{ error_code, msg, data: { list: [ { roleId, roleName, roleDescr }, ... ] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 角色列表 |
| data.list[].roleId | Integer | 角色 ID |
| data.list[].roleName | String | 角色名称 |
| data.list[].roleDescr | String | 角色描述 |

---

## 26. op=Enterprise.add — 新建企业

### 请求格式
{"op": "Enterprise.add", "action": "user", "data": {"enterpriseInfo": {...}, "onlineUserInfo": {"email": "..."}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| enterpriseInfo | JSONObject | 是 | `DbEnterpriseInfo`，`name`、`fullname`、`channel` 必填 |
| onlineUserInfo.email | String | 是 | 在线用户邮箱 |

### 核心逻辑

注册新企业。名称重复返回 `ENTERPRISE_NAME_EXIST_ERROR`。

### 响应

`{ error_code, msg, data: { result: eid } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 新建企业 ID（eid） |

---

## 27. op=Enterprise.getByThirdPartyEid — 按第三方 ID 查企业

### 请求格式
{"op": "Enterprise.getByThirdPartyEid", "action": "user", "data": {"thirdPartyEid": "..."}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| thirdPartyEid | String | 是 | 第三方企业 ID |

### 核心逻辑

通过第三方标识查询企业信息。

### 响应

`{ error_code, msg, data: { object: DbEnterpriseInfo(JSON) } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 企业信息对象（DbEnterpriseInfo） |

---

### 涉及表

- `db_enterprise`（`DbEnterpriseInfo`）— 企业基本信息
- `db_enterprise_expand`（`DbEnterpriseExpand`）— 企业扩展信息
- `db_enterprise_user_relation`（`DbEnterpriseUserRelation`）— 企业-用户-项目关联
- `db_enterprise_auth_record`（`DbEnterpriseAuthRecord`）— 审核记录
- `db_user`（`DbUserInfo`）— 用户表
- `db_user_role`（`DbUserRoleInfo`）— 用户默认角色
- `db_system_param`（`DbSystemParam`）— 系统参数（默认密码等）
- `view_enterprise_user`（`ViewEnterpriseUser`）— 企业用户视图
- `view_enterprise_info`（`ViewEnterpriseInfo`）— 企业信息视图
