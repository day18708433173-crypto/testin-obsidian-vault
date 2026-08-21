---
tags: [常见问题]
---

# 构建与部署FAQ

## 后端

### Q: `mvn package` 后 `target/` 里没有产物？

产物输出目录被改成了 `${basedir}/deploy/`——去 `deploy/` 找 jar、`lib/`、`resources/`、`bin/`。见 [部署与打包](../02-技术架构/部署与打包.md)。

### Q: 单独把 jar 拷到服务器，`java -jar` 起不来？

资源文件被 maven-jar-plugin 排除在 jar 外，必须按 `java -cp resources:lib/*:...` 方式启动（`resources/` 在 classpath 最前）。直接参考根目录 `startup.sh` / `startup_runner.sh`。见[部署与打包](../02-技术架构/部署与打包.md)

### Q: 新增依赖后运行时 `NoSuchMethodError` / 类冲突？

lib 目录**按文件名字典序加载**，与 JMeter 冲突的第三方包靠命名靠前优先加载（pom 里 system scope jar 的注释说明了这一点）。检查新依赖是否与 JMeter 自带包冲突，必要时调整命名/exclusion。

### Q: 执行端打包忘了 `-f runnerPom.xml`？

`mvn package -f runnerPom.xml` 才产出执行端（artifactId `testin-api-backend-runner`，入口 `cn.testin.runner.RunnerApplication`）。默认 `mvn package` 只打服务端。

### Q: 构建报依赖找不到？

后端走阿里云 mirror，不依赖公司内网 Artifactory；确认 `mvn -v` 是 JDK 1.8。easyexcel 已排除 `poi-ooxml-schemas`，若新依赖传递引入旧 poi 会再次冲突。

### Q: Docker 镜像构建后容器里没有代码？

`Dockerfile` 是 `ADD target /data/target`，**镜像构建不含 mvn 步骤**——需先 `mvn package`，再把 `deploy/` 归置为 `target/` 再 `docker build`（CI 已处理）。

### Q: 容器启动后连的主平台地址不对？

启动脚本用环境变量 `OPENAPI_URL` 全局替换 yml 里的 `http://openapi.pro.testin.cn`。检查部署时是否注入了 `OPENAPI_URL`。

### Q: 测试机执行 MongoDB 校验失败，报 shell 找不到？

MongoDB 校验依赖部署机上的 mongo shell，路径配在 profile yml 的 `mongodbshell.path.v4`（默认 `/opt/mongodb-linux-v4/bin`）。`resources/deb/` 与 `python_dep/` 也是启动脚本装的系统依赖，缺了会有类似"命令不存在"的错。

## 前端

### Q: 用 npm/yarn 装依赖后构建行为异常？

**必须用 pnpm**（README 明确要求），依赖全部精确版本锁定。换回 pnpm 并删除 `node_modules` 重装。

### Q: `pnpm build` 失败但从哪步开始错看不出来？

build 是三步串联：`vitepress build docs && vue-tsc && vite build`。分开单跑定位：类型错误单跑 `vue-tsc`，文档错误单跑 `pnpm docs:build`。

### Q: 客户浏览器白屏，本地 Chrome 正常？

构建目标是 chrome71（`@vitejs/plugin-legacy`）。先怀疑用了未 polyfill 的新语法/API——查 `vite.config.ts` 的 polyfills 列表。另一个高频原因：`el-table-column` 用了 `prop` 取值（低版本浏览器 bug），改用 `<template #default="{ row }">`。

### Q: 部署后发现引用了旧 JS / 缓存串？

产物文件名按构建时间戳目录输出（`YYYYMMDDHHmm/[name]-[hash].js`），这是刻意的 CDN 缓存隔离机制；不要改成固定 `assets/` 路径。若仍有缓存问题，确认 index.html 本身未被 CDN 长缓存。

### Q: CI 构建与本地不一致？

CI（`.workflow/*.yml`）用 **node 14** 跑 `npm install && npm run build`。本地 node 版本过高可能表现不同，对齐版本再排查。

## 相关阅读

- [部署与打包](../02-技术架构/部署与打包.md) / [后端双形态架构](../02-技术架构/后端双形态架构.md) / [前端架构](../02-技术架构/前端架构.md) / [任务执行问题排查](任务执行问题排查.md)
