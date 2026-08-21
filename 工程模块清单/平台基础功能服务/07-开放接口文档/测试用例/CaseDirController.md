# CaseDirController — 测试用例目录

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/testCase/CaseDirController.java`
> 类级路由：`/v3/test_case/case_dir`
> Service 实现：`cn.testin.business.impl.testCase.CaseDirServiceImpl`（约 368 行，委托给 `ICaseDirService`）
> 业务：测试用例目录树的增删改查、移动、用例数统计。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|---|
| 1 | POST | `/v3/test_case/case_dir` | addCaseDir | 新增用例目录 | 无 |
| 2 | PUT | `/v3/test_case/case_dir` | editCaseDir | 修改目录名称 | 无 |
| 3 | GET | `/v3/test_case/case_dir/case_dirs` | getCaseDirList | 获取目录树（带子目录递归） | 无 |
| 4 | DELETE | `/v3/test_case/case_dir/{case_dir_id}` | deleteCaseDir | 删除目录（级联删子目录） | 无 |
| 5 | POST | `/v3/test_case/case_dir/move` | moveCaseDir | 移动目录（同级排序/跨级移动） | 无 |
| 6 | GET | `/v3/test_case/case_dir/case_num` | getCaseNum | 获取目录及子目录下的用例总数 | 无 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`；写操作返回 `BaseDataResultDTO { Long result }`。

---

## 1. POST /v3/test_case/case_dir — 新增用例目录

### 入口

`CaseDirController.addCaseDir(@RequestBody CaseDir caseDir)`

### 请求参数（CaseDir，JSON Body）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业 ID |
| projectId | 是 | 项目 ID |
| caseDirName | 是 | 目录名称 |
| parentId | 是 | 父目录 ID |
| caseDirOrder | 否 | 排序号，缺省时自动取同级最大+1 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 新目录主键 ID。

### 实现意图

校验同级目录下不能存在同名（`caseDirName`），自动计算排序号为同级最大 + 1，插入后返回主键。

### 调用链

```
CaseDirController.addCaseDir
└─ CaseDirServiceImpl.addCaseDir
   ├─ ICaseDirDAO.selectCount (同名检查)
   ├─ ICaseDirDAO.selectMaxOrderByParentId (排序计算)
   └─ ICaseDirDAO.insert (db_case.case_dir)
```

### 涉及表

- `db_case.case_dir`

---

## 2. PUT /v3/test_case/case_dir — 修改用例目录

### 入口

`CaseDirController.editCaseDir(@RequestBody CaseDirDTO caseDir)`

### 请求参数（CaseDirDTO）

| 字段 | 必填 | 说明 |
|---|---|---|
| caseDirName | 是 | 新目录名称 |
| caseDirId | 是 | 目录 ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

仅更新目录名称，通过 `caseDirId` 定位记录。

### 涉及表

- `db_case.case_dir`

---

## 3. GET /v3/test_case/case_dir/case_dirs — 获取目录树

### 入口

`CaseDirController.getCaseDirList(@RequestParam(required=false) Integer caseDirId, Integer eid, Integer projectId)`

### 请求参数（Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| case_dir_id | 否 | 指定根目录 ID，不传则返回项目根目录 |
| eid | 是 | 企业 ID |
| project_id | 是 | 项目 ID |

### 响应结构

`ResponseResult<List<CaseDirTreeDTO>>`（包装为单元素列表）。

### 返回参数（CaseDirTreeDTO 元素结构）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Array\<CaseDirTreeDTO\> | 目录树（通常单元素根节点） |
| data[].caseDirId | Integer | 目录 ID |
| data[].parentDirId | Integer | 父目录 ID |
| data[].caseDirName | String | 目录名称 |
| data[].caseDirOrder | Integer | 排序号 |
| data[].children | Array\<CaseDirTreeDTO\> | 子目录（递归嵌套，同结构） |

### 实现意图

若 `caseDirId` 为空则取 `parentId IS NULL` 的根目录（不存在则自动创建，以项目名作为根目录名称）；否则从指定目录开始，递归拼装完整目录树（`children` 嵌套）。

### 关键代码摘录

```java
if (caseDirId == null) {
    lambdaQueryWrapper.isNull(CaseDir::getParentId);
    CaseDir rootCaseDir = caseDirDAO.selectOne(lambdaQueryWrapper);
    if (rootCaseDir == null) {
        rootCaseDir = new CaseDir();
        rootCaseDir.setCaseDirName(dbProjectInfo.getName());
        caseDirDAO.insert(rootCaseDir);
    }
}
return concatCaseDirTreeDTO(dirTreeDTO); // 递归拼装 children
```

### 涉及表

- `db_case.case_dir`

---

## 4. DELETE /v3/test_case/case_dir/{case_dir_id} — 删除目录

### 入口

`CaseDirController.deleteCaseDir(@PathVariable Integer caseDirId, @RequestParam Integer userId)`

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| case_dir_id | 是 | 路径参数，目录 ID |
| user_id | 是 | 操作人 ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

递归查询所有子目录 ID（`getAllChildCaseDir`，层级遍历直到无新节点），批量逻辑删除（`status`）。删除后清除受影响目录的用例数 Redis 缓存（向上递归所有父目录）。

### 调用链

```
CaseDirController.deleteCaseDir
└─ CaseDirServiceImpl.deleteCaseDir
   ├─ getAllChildCaseDir (递归收集子目录ID)
   ├─ ICaseDirDAO.batchDeleteByIds (db_case.case_dir)
   └─ updateCaseNum (清除Redis目录用例数缓存)
```

### 涉及表

- `db_case.case_dir`

---

## 5. POST /v3/test_case/case_dir/move — 移动目录

### 入口

`CaseDirController.moveCaseDir(@RequestBody MoveCaseDirDTO moveCaseDirDTO)`

### 请求参数（MoveCaseDirDTO）

| 字段 | 必填 | 说明 |
|---|---|---|
| caseDirId | 是 | 要移动的目录 ID |
| parentCaseDirId | 否 | 原父目录 ID（由后端查询填充） |
| targetCaseDirId | 是 | 目标父目录 ID |
| caseDirOrder | 是 | 原排序号 |
| targetCaseDirOrder | 否 | 目标排序号（同级排序用） |

### 响应结构

`ResponseResult<BaseDataResultDTO>`。

### 实现意图

`@Transactional`。三种场景：①同级排序：更新排序号并调用 DAO 调整区间内其他目录顺序 ②跨级移动到目标目录末尾：自动取目标目录最大排序号 + 1 ③跨级移动到指定位置：更新 `parentId` 和排序号并调整目标区间。每次移动后清除新旧目录的用例数缓存。

### 调用链

```
CaseDirController.moveCaseDir
└─ CaseDirServiceImpl.moveCaseDir (@Transactional)
   ├─ ICaseDirDAO.updateCaseDirOrder (同级排序区间调整)
   ├─ ICaseDirDAO.updateCaseDirOrderAndParentId (跨级排序调整)
   ├─ ICaseDirDAO.updateById (db_case.case_dir)
   └─ updateCaseNum (清除Redis)
```

### 涉及表

- `db_case.case_dir`

---

## 6. GET /v3/test_case/case_dir/case_num — 目录用例数统计

### 入口

`CaseDirController.getCaseNum(@RequestParam Integer caseDirId)`

### 请求参数（Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| case_dir_id | 是 | 目录 ID |

### 响应结构

`ResponseResult<Map<Integer, Integer>>`，key=目录 ID，value=该目录及所有子目录下的用例总数。

### 实现意图

递归收集所有子目录 ID，对每个目录优先从 Redis 缓存读取用例数（`CASE_NUM_REDIS_KEY + dirId`，TTL 24h），未命中则查 `db_case.case_info` 统计后写入缓存。

### 调用链

```
CaseDirController.getCaseNum
└─ CaseDirServiceImpl.getCaseNum
   ├─ getAllChildCaseDir (递归收集子目录)
   └─ getCaseNumForMap
      ├─ redisService.get (CASE_NUM_REDIS_KEY)
      └─ ICaseInfoDAO.countByCondition (db_case.case_info)
```

### 涉及表

- `db_case.case_info`
- `db_case.case_dir`

