---
name: aoi-java-backend
description: 处理 fs-aoi-biz Java 后端开发任务。用于 Java 实现、缺陷修复、模块导航、Maven 构建、Spring Boot 启动配置、KJDP/Koca 业务逻辑、MyBatis、XML 元数据和后端代码评审。
---

# AOI Java 后端开发

## 起步步骤

1. 先阅读 `.codex/skills/project-tech-and-standards.skill.md`。
2. 根据路径、服务码、包名或业务描述识别目标模块。
3. 编辑前先追踪业务链路：service XML -> ATOM/BEX XML -> Java 类/方法 -> MyBatis XML -> 测试或 mock 数据。
4. 保持变更范围收敛，并遵循邻近代码的既有写法。

## 仓库模块地图

- 启动模块：`fs-aoi-biz-starter`，主类 `com.szkingdom.fs.AoiServiceApplication`。
- Spring 配置：`fs-aoi-config`。
- 公共框架与基础业务：`fs-aoi-frame`。
- 主业务 ATOM：`fs-aoi-atom`。
- 流程与任务分派：`fs-aoi-workflow`。
- 准入、检查、白名单：`fs-aoi-access`。
- 外部、报表、KONE 适配：`fs-aoi-adapter`。
- 子系统模块：`fs-aoi-subsys`。
- 券商个性化模块：`fs-aoi-qs`。
- 测试与 mock 支撑：`fs-aoi-test`。

## 编码规则

- 保持 Java 8 兼容。
- 优先沿用现有 `Map`、`GenericRequest`、`GenericResult`、`AtomResult`、`DataExchangeAssembly`、`BusinessModel` 等模式。
- 使用 `FrameDao.doAtomCall/doAtomInvoke` 和 `FrameDao.doBexCall/doBexInvoke` 时，与周边代码保持一致。
- 除非任务明确要求，否则不要改变服务码、错误码、提示语、业务字段名和返回结构。
- 不要为局部修复新增框架层、REST Controller、DTO 体系或依赖升级。
- 涉及金融、客户、账户、流程的变更，要补充定点测试，或说明可复现的验证路径。

## 检索模式

```powershell
rg -n "<服务码或业务码>" -g "*.xml" -g "*.java"
rg -n "FrameDao\.doAtomCall|FrameDao\.doBexCall|<atom-code>|<bex-code>" -g "*.java" -g "*.xml"
rg -n "<表名或字段名>|<Map字段>|<BUSI_CODE>" -g "*.java" -g "*.xml" -g "*.json"
```

## 构建命令

```powershell
mvn clean install -DskipTests
mvn -pl fs-aoi-atom -am clean install -DskipTests
mvn -pl fs-aoi-biz-starter -am clean package -DskipTests
```

选择能覆盖本次改动的最小模块构建。如果依赖解析需要内部 Nexus，而当前网络受限，要在结果中明确说明。

## 评审清单

- XML 注册是否仍然指向真实存在的 Java 类和方法？
- 服务码、业务码、ATOM 编码、BEX 编码是否保持唯一？
- Map 字段是否与 mapper SQL 和调用方一致？
- 错误码和 prompt 是否按邻近代码风格透传？
- SQL 是否兼容本项目涉及的数据库？
- 是否误改了券商个性化或子系统专属逻辑？
