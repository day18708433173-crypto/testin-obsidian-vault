# InterfaceCase

> 提供方：平台基础功能服务 模块
> 通信方式：V3 REST (ServiceRemoteV3Api)
> 服务端点：/v3/interfaceCase/*

## 概述
接口用例服务，提供接口用例信息查询功能。由 平台基础功能服务 模块提供，用于第三方集成场景中 脚本服务 脚本与接口用例的关联查询。

## 脚本服务 使用的接口

| 方法     | 说明            | 调用场景                             |
| ------ | ------------- | -------------------------------- |
| 用例列表查询 | 查询可关联的接口用例列表  | thirdparty.Script 绑定第三方功能时拉取候选用例 |
| 用例详情查询 | 查询单个接口用例的详细信息 | 脚本执行时获取关联用例的请求参数                 |
| 用例参数提取 | 从接口用例中提取可用参数  | 脚本参数自动填充                         |

## 调用说明

```java
// V3 REST 调用方式
// 通过 ServiceRemoteV3Api 调用 testin-core 的 /v3/interfaceCase 端点
String result = ServiceRemoteV3Api.call(
    "/interfaceCase/list",
    params
);
```

## 依赖方
- thirdparty.Script — 第三方脚本集成（bindThirdFunc、paramList 等方法）
