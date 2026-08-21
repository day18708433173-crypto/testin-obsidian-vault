# CaseCommentController — 用例评论

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/testCase/CaseCommentController.java`
> 类级路由：`/v3/test_case/case_comment`
> Service 实现：`cn.testin.business.impl.testCase.CaseCommentServiceImpl`（约 87 行，委托给 `ICaseCommentService`）
> 业务：用例评论的增删改查，删除为逻辑删除（`status → DISABLED`）。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|---|
| 1 | POST | `/v3/test_case/case_comment` | addCaseComment | 新增用例评论 | 无 |
| 2 | PUT | `/v3/test_case/case_comment` | editCaseComment | 修改用例评论 | 无 |
| 3 | DELETE | `/v3/test_case/case_comment/{case_comment_id}` | deleteCaseComment | 删除评论（逻辑删） | 无 |
| 4 | GET | `/v3/test_case/case_comment/case_comments` | getCaseCommentList | 查询用例评论列表 | 无 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`；写操作返回 `BaseDataResultDTO { Long result }`。

---

## 1. POST /v3/test_case/case_comment — 新增评论

### 入口

`CaseCommentController.addCaseComment(@RequestBody CaseCommentDTO caseCommentDTO)`

### 请求参数（CaseCommentDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| caseId | Integer | 否 | 用例 ID（代码未强制校验，直接落库） |
| userId | Integer | 否 | 评论人 ID（同时作为 createUserId、updateUserId） |
| comment | String | 否 | 评论内容 |
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 否 | 项目 ID |

> 注：`addCaseComment` 无 `@Valid`，Service 层也无判空，字段必填性未在代码层强制（依赖 `db_case.case_comment` 表的 NOT NULL 约束兜底）。

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 新评论主键 ID。

### 实现意图

构造 `CaseComment` 实体（`status=ENABLED`，`createTime/updateTime` 均取当前时间），插入后返回自增 ID。

### 涉及表

- `db_case.case_comment`

---

## 2. PUT /v3/test_case/case_comment — 修改评论

### 入口

`CaseCommentController.editCaseComment(@RequestBody CaseCommentDTO caseCommentDTO)`

### 请求参数（CaseCommentDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| caseCommentId | Integer | 否 | 评论 ID（作为更新主键，代码未判空） |
| userId | Integer | 否 | 修改人 ID |
| comment | String | 否 | 新评论内容 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

更新 `comment` 内容和 `updateUserId`、`updateTime`。不支持修改 `caseId`。

### 涉及表

- `db_case.case_comment`

---

## 3. DELETE /v3/test_case/case_comment/{case_comment_id} — 删除评论

### 入口

`CaseCommentController.deleteCaseComment(@PathVariable Integer caseCommentId)`

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| case_comment_id | 是 | 评论 ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`。

### 实现意图

逻辑删除：将 `status` 更新为 `DISABLED`，不物理删除记录。

```java
CaseComment caseComment = CaseComment.builder()
    .id(caseCommentId)
    .status(CaseCommentStatusEnum.DISABLED.getCode())
    .build();
caseCommentDAO.updateById(caseComment);
```

### 涉及表

- `db_case.case_comment`

---

## 4. GET /v3/test_case/case_comment/case_comments — 查询评论列表

### 入口

`CaseCommentController.getCaseCommentList(@RequestParam Integer caseId)`

### 请求参数（Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| case_id | 是 | 用例 ID |

### 响应结构

`ResponseResult<List<CaseComment>>`，返回该用例所有评论（不过滤 `status`，含已逻辑删除的评论）。

### 返回参数（CaseComment 元素结构）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Array\<CaseComment\> | 评论列表 |
| data[].id | Integer | 评论主键 |
| data[].caseId | Integer | 用例 ID |
| data[].caseComment | String | 评论内容 |
| data[].createUserId | Integer | 创建人 ID |
| data[].updateUserId | Integer | 更新人 ID |
| data[].createTime | Date | 创建时间 |
| data[].updateTime | Date | 更新时间 |
| data[].status | Integer | 状态（ENABLED/DISABLED，删除后为 DISABLED） |

### 实现意图

按 `caseId` 等值查询全量列表，不做分页。

### 涉及表

- `db_case.case_comment`

