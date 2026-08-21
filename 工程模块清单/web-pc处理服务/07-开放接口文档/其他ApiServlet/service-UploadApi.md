# service-UploadApi — 文件上传

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/file/upload/UploadApi.java`（SpringHelper Bean `file.upload.UploadApi`）
> 类型：远端代理（→ File 服务）
> 转发方式：HTTP Multipart POST（`HttpPoster.postWithRes(Config.UPLOAD_URL, bytes, headers)`，参数 JSON 放 header `uploadParam`，含 apikey/sig 签名）

## 方法列表

### 1. upload — 上传文件

```java
public String upload(String originalName, Map<String, Object> dataMap, JSONObject contentJson) throws GeneralException
```

**用途**：把本地文件（Excel 报告、截图等）上传到 File 服务，返回可下载的 downloadUrl。使用内置上位机 apikey/secretKey（免登录上传），sig 由 `AbstractApi.getSig`（排序拼接 + MD5）计算。

**转发目标**：`action=file, op=FileApi.upload`（POST 到 `Config.UPLOAD_URL`，请求体为文件字节流，参数在 header `uploadParam`）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| originalName | String | 是 | 本地文件路径（`new File(originalName)`，null/非法路径读取失败） |
| dataMap | Map&lt;String,Object&gt; | 否 | data 节点参数 |
| contentJson | JSONObject | 否 | 请求体（当前调用多传 null） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| downloadUrl | String | `data.downloadUrl`（失败/响应为空返回 null） |

**调用者**：
- `GenerateReportServiceImpl.java` / （上传 Excel 报告）
- `pojo/pdf/PDFThread.java`（PDF 报告上传）
- `pojo/mongo/reportDetail/StepInfo.java`（步骤截图上传）

## 相关文档

- [00-分支索引](00-分支索引.md)
- [File 服务](../../../文件管理服务/00-首页.md)
- [service-ReportApi](service-ReportApi.md)
