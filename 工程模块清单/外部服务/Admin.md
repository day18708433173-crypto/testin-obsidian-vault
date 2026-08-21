# Admin

> 提供方：Admin 管理后台（独立部署，地址由 `Config.ADMIN_URL` 配置）
> 通信方式：HTTP REST（hutool HttpUtil.post，JSON Body）
> 包路径：cn.testin.api.admin.AdminApi

## 概述
平台管理后台服务。脚本服务 仅在脚本导出场景调用它：导出文件生成后，将文件地址注册到 Admin 后台的磁盘清理队列，由 Admin 负责后续的过期文件物理删除。

## 脚本服务 使用的接口

| 端点 | 说明 | 调用场景 |
|------|------|----------|
| POST `{ADMIN_URL}/api/clean/disk/addToDel.htm` | 注册待删除文件地址 | 脚本导出完成后登记导出文件，供磁盘清理任务删除 |

## 调用示例

```java
// ExportScript（ApiServlet）中导出完成后调用
ResponseBean res = ...; // 含导出文件地址
AdminApi.insertToDel(res);
```

## 依赖方
- [ExportScript](../脚本服务/07-开放接口文档/脚本管理/ExportScript.md)（唯一调用点：ExportScript.java）

## 备注
- 调用失败仅记录 error 日志，不影响导出主流程（导出文件可能残留，由磁盘监控兜底）。
- 与 [FileUpload](../文件管理服务/00-首页.md) 的区别：FileUpload 负责文件的上传/存储，Admin 负责存储侧文件的清理调度。
