# service-UserApi — 用户信息查询代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/user/UserApi.java`（继承 `AbstractApi`，Spring bean 名 `user.UserApi`）
> 类型：远端代理（→ UserManager 服务）
> 转发方式：V1 ApiServlet 前缀 `UserManager`，`ApiUtil.doPress(userPrefixName, reqJson)`

## 方法列表

### 1. get — 获取用户信息

```java
public UserInfo get(Integer eid, Integer userid, String email, String sid) throws GeneralException
```

**用途**：按 userid / email / sid 任一条件查询用户信息（user_info）。

**转发目标**：

```java
reqJson.put("action", "user");
reqJson.put("op", "User.getUser");
ApiUtil.doPress(this.userPrefixName, reqJson.toString()); // UserManager
```

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 id |
| userid | Integer | 否 | 用户 id |
| email | String | 否 | 用户邮箱 |
| sid | String | 否 | 会话 sid |

> userid / email / sid 三者至少一个有效，全空直接返回 null。

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | UserInfo | 用户信息对象（由 `jsonObjInfo` gson 反序列化，字段见 UserManager 服务，代码未确认） |

**调用者**：
- `GenericBaseService.java` — `userapi.get(eid, userid, null, null)`
- `NoticeServiceImpl.java / 6305` — 通知场景取用户信息
- `ActionLogService.java` — 操作日志记录用户名
- `TaskServiceImpl.java`

## 相关文档

- [00-分支索引](00-分支索引.md)
- [UserManager](../../../平台基础功能服务/00-首页.md)
