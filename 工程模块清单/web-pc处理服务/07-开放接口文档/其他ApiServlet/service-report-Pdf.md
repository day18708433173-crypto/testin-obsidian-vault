# service-report-Pdf — HTML 报告转 PDF（队列异步）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/report/Pdf.java`（继承 `GenericBaseService`）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=report, op=Pdf.<方法>
> 本地注入：无（使用静态工具 `Html2PdfUtils` 的内存 workMap/workQueue）

## 方法列表

### 1. parse — 提交/查询 HTML 转 PDF 任务

```java
public String parse(ApiRequest apirequest) throws Exception
```

**用途**：将报告页面 url 转为 PDF。轮询式接口：code -1 失败 / 0 等待 / 1 成功。

**流程**：
1. 校验 `url` 非空
2. 对 url 取 MD5 作为 md5Key
3. 查 `Html2PdfUtils.getWorkMap()`：
   - 命中 → 取缓存 `ResponseBean`；若 code 为 -1 或 1（终态），返回后从 Map 移除
   - 未命中 → 新建 `ResponseBean`（code=0、md5Key、targetUrl=url），放入 workMap 并 offer 进 workQueue 等待后台消费转换
4. 返回 ResponseBean

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| url | String | 是 | 待转换的报告页面 url |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | ResponseBean（见下） |
| data.objInfo.code | Integer | -1 失败 / 0 等待 / 1 成功 |
| data.objInfo.md5Key | String | url 的 MD5 值（缓存 key） |
| data.objInfo.targetUrl | String | 待转换的目标 url |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| 内存 | Html2PdfUtils.workMap / workQueue | 读写 |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-report-Report](service-report-Report.md)
- [service-report-Excel](service-report-Excel.md)
