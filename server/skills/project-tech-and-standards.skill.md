---
name: project-tech-and-standards
description: fs-aoi-biz 的 AOI Java 后端项目规范。用于理解模块边界、Maven 命令、Java 编码约定、服务注册、测试、打包和安全变更实践。
---

# fs-aoi-biz 项目技术与协作规范

## 项目定位

`fs-aoi-biz` 是 AOI 业务受理后端 Java 多模块工程，不是前端工程。处理任何任务前，先从代码本身确认模块、服务码、ATOM/BEX、MyBatis XML 和测试支撑，不沿用前端 `fs-aoi-web`、Vue、Kui、Vite、npm 相关经验。

## 技术栈

- Java 8，根 `pom.xml` 使用 `maven-compiler-plugin` source/target `1.8`。
- Maven 多模块工程，根工程 `com.szkingdom.kcop:fs-aoi-biz:${revision}`，当前 `revision` 为 `1.0.5.0`。
- Spring Boot `2.5.8`，启动模块是 `fs-aoi-biz-starter`。
- KJDP/Koca 体系：`kjdp.version=3.5.8.0`，`koca.version=5.2.0`。
- MyBatis `3.5.2`，MyBatis-Spring `2.0.2`，数据库驱动覆盖 MySQL、Oracle、SQL Server、openGauss。
- 日志以 Log4j2 为主，启动模块配置 `conf/log4j2-spring.xml`。

## 模块边界

- `fs-aoi-biz-starter`：启动与打包模块，入口 `com.szkingdom.fs.AoiServiceApplication`，默认服务名 `fs-aoi`，打包输出目录名 `fs-aoi-biz-app`。
- `fs-aoi-config`：Spring Java 配置，替代部分旧 XML。关注 `KcopExternalConfig`、`KcopWebConfig` 等。
- `fs-aoi-frame`：框架公共能力、门户/参数/权限/公共 ATOM、Web Controller、工具类。
- `fs-aoi-atom`：主要业务 ATOM、业务处理流程、客户/账户/影像/移动端/定时任务等业务逻辑。
- `fs-aoi-workflow`：业务流程、任务认领/分派、流程数据处理。
- `fs-aoi-access`：准入、检查、白名单、规范性检查等。
- `fs-aoi-adapter`：外部适配与报表适配，含 `fs-aoi-adapter-api`、`external`、`report`、`kone` 等子模块。
- `fs-aoi-subsys`：子系统业务模块，如 `kuas`、`kspb`、`kgis`、`kgob`、`kfis`、`kidm` 等。
- `fs-aoi-qs`：券商个性化模块。
- `fs-aoi-test`：测试与 mock 支撑模块，默认 Surefire `skip=true`，跑测试时要显式处理。

## 启动与配置

- 启动类：`fs-aoi-biz-starter/src/main/java/com/szkingdom/fs/AoiServiceApplication.java`。
- `@SpringBootApplication` 排除了 `QuartzAutoConfiguration`、`DataSourceAutoConfiguration`、`LoadBalancerZookeeperAutoConfiguration`。
- `@ComponentScan` 覆盖 `com.szkingdom.kjdp`、`com.szkingdom.kcop`、`com.szkingdom.aoi`、`com.szkingdom.koca.hdb.function.mybatis.config`。
- 根路径 `GET /${spring.application.name}` 的默认响应是 `ok`。
- `fs-aoi-biz-starter` 支持 `tomcat`、`kwas`、`tongweb`、`bes` profile，默认激活 `tomcat`。

## 业务注册模式

本工程大量业务不是通过 Spring MVC Controller 暴露，而是通过 KJDP XML 元数据注册：

- `META-INF/service/*.xml`：服务码注册，`service_code` 对外，`service_business.business_code` 指向 BEX/ATOM/业务编码。
- `META-INF/atom/*.xml`：ATOM 注册，`atom_code` 绑定 `atom_class` 和 `atom_method`。
- `META-INF/bex/*.xml`：BEX 注册，`bex_code` 通常映射查询、增删改、批量处理等后端业务。
- `META-INF/mybatis/*.xml`：SQL 映射。修改 BEX 时同步查找对应 mapper。
- `META-INF/view/*.xml`：历史视图/菜单配置仍在后端资源内，除非任务明确涉及，不按前端页面重构处理。

变更服务链路时，用 `rg -n "<service_code>|<business_code>|<atom_code>|<bex_code>"` 追完整调用链。更详细的接口入口模式见 `.codex/skills/aoi-service-registry/SKILL.md`，尤其要区分以下 5 类：

1. 前端调用通用 service（如 `P9999999`、`Y3000001`），入参携带 `bex_codes`，再转到具体 BEX；账户类 BEX 还要区分新账户 `fs-uas-act://...` 服务发现模式和老账户 `KUAS.L...` DLL/老账户模式。
2. 前端 service（如 `Y1001137`）通过 `business_code` 映射到 ATOM，再由 ATOM XML 映射 Java 类和方法。
3. 前端 service（如 `YG000342`）通过 `business_code` 映射到 BEX，再由 BEX 映射到 MyBatis SQL id。
4. 自动任务通过 `upm_qrtz_job_details.job_class_name` 配置 Java 类，例如 `com.szkingdom.kdop.peripheral.jobs.ApptBusiDataJob`。
5. 业务任务配置通过 `opp_busi_trans_conf.class_name` 配置 Java 类，例如专业投资者续期业务处理类 `com.szkingdom.opp.atom.business.appropriateManage.OpenProfessionalInvesterRight`。

## Java 编码约定

- 保持 Java 8 兼容，不引入 Java 9+ API、var、record、Stream.toList 等。
- 优先沿用现有 `Map`/`GenericResult`/`AtomResult`/`DataExchangeAssembly` 模式，不随意改造成 DTO 或 REST 风格。
- 调用 ATOM/BEX 常见入口为 `FrameDao.doAtomCall`、`FrameDao.doBexCall`、`FrameDao.doAtomInvoke`、`FrameDao.doBexInvoke`；新增调用时保留原有错误码与 `prompt` 透传风格。
- 涉及业务处理链时，先找抽象父类和接口，如 `OppBaseAtom`、`OppBaseHandleProcess`、`OppBaseConfirmProcess`、`BusinessModel`。
- 金融业务字段多为大写下划线 key。新增 Map key 必须与 XML、SQL、前端/调用方契约一致。
- 不随意吞异常。现有业务常用 `AtomException`、`KcopBizException` 或返回 `GenericResult` 错误码，按邻近代码风格处理。

## Maven 命令

- 全量编译：`mvn clean install -DskipTests`
- 定点编译模块及依赖：`mvn -pl <module> -am clean install -DskipTests`
- 启动模块打包：`mvn -pl fs-aoi-biz-starter -am clean package -DskipTests`
- 指定容器 profile：`mvn -pl fs-aoi-biz-starter -am clean package -Ptomcat -DskipTests`
- 跑测试模块时注意 `fs-aoi-test` 的 Surefire 默认 `skip=true`，需要结合具体测试类显式关闭或用 IDE/命令单独运行。

## 变更前检索清单

- 改服务码：查 `META-INF/service`、`META-INF/atom`、`META-INF/bex`、`META-INF/mybatis`。
- 改 ATOM：查 Java 类、注册 XML、调用方 `FrameDao.doAtomCall`、测试类。
- 改 BEX/SQL：查 BEX XML、mapper XML、表字段、mock tabledata。
- 改启动配置：查 `fs-aoi-biz-starter`、`fs-aoi-config`、`application.yml`、`log4j2-spring.xml`。
- 改子系统：先定位 `fs-aoi-subsys/<submodule>` 或 `fs-aoi-qs/<broker-module>`，不要把券商个性化逻辑上提到公共模块。

## 质量门槛

- 代码改动必须至少能说明编译验证命令；无法运行时说明原因。
- XML 改动必须保持 well-formed，服务码/ATOM/BEX 编码不得重复。
- MyBatis SQL 改动要考虑 Oracle/MySQL/SQL Server/openGauss 差异，避免使用单一数据库私有语法，除非邻近 mapper 已限定。
- 不提交生成目录、日志、打包输出和临时文件。
- 保留用户未请求的业务逻辑和乱码注释，不做无关格式化。


