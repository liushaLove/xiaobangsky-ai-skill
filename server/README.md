# fs-aoi-biz 后端协作 Skill 包

这个目录是一套面向 `fs-aoi-biz` 项目的 Codex/AI Agent 协作资料包，用来帮助 AI 在处理 AOI 业务受理后端任务时快速理解项目边界、服务注册链路、开发规范、测试方式和交付要求。

从当前文件内容看，本资料包聚焦的是 `fs-aoi-biz` Java 后端工程，而不是 `fs-aoi-web` 前端工程。核心项目特征是 Java 8、Maven 多模块、Spring Boot 2.5.8、KJDP/Koca、MyBatis，以及大量通过 `META-INF/service`、`META-INF/atom`、`META-INF/bex`、`META-INF/mybatis` XML 元数据注册和路由的业务服务。

## 目录结构

```text
server/
├── agents/
│   ├── lead-developer.md
│   ├── project-assistant.md
│   └── tester.md
├── skills/
│   ├── project-tech-and-standards.skill.md
│   ├── aoi-java-backend/
│   │   └── SKILL.md
│   ├── aoi-service-registry/
│   │   └── SKILL.md
│   ├── aoi-backend-test/
│   │   └── SKILL.md
│   └── git-commit-aigc/
│       └── SKILL.md
├── repowiki/
│   └── zh/
│       └── content/
│           ├── 项目概述.md
│           └── 后端开发速查.md
└── doc/
```

## Agent 角色

### 后端主程

`agents/lead-developer.md` 定义 AOI Java 后端主程角色，负责功能设计、缺陷修复、服务注册、XML/MyBatis 配置、模块构建和代码评审。它要求先阅读项目技术规范，再通过 `rg` 追踪服务码、业务码、ATOM/BEX、mapper 和调用方，并在最小影响模块内修改。

### 项目协作助手

`agents/project-assistant.md` 定义项目协作与交付协调角色，负责需求拆解、风险同步、验证闭环和知识沉淀。它强调先确认影响模块、服务码、ATOM/BEX、MyBatis XML 和测试范围，并在发布或提测前确认编译命令、测试命令、未验证风险和回滚点。

### 后端测试工程师

`agents/tester.md` 定义后端测试工程师角色，负责业务测试、服务链路验证、接口入参出参核对、数据库/缓存/mock 数据准备和缺陷复现。它特别提醒 `fs-aoi-test` 的 Surefire 默认 `skip=true`，不能把“编译通过”等同于“测试已执行”。

## Skill 说明

### project-tech-and-standards

`skills/project-tech-and-standards.skill.md` 是总规范，覆盖项目定位、技术栈、模块边界、启动配置、业务注册模式、Java 编码约定、Maven 命令、变更前检索清单和质量门槛。

使用它可以帮助 AI 避免把项目误判为普通 REST 项目或前端项目，并确保修改前先追踪 KJDP XML 注册链路。

### aoi-java-backend

`skills/aoi-java-backend/SKILL.md` 面向 Java 后端开发任务，包括 Java 实现、缺陷修复、模块导航、Maven 构建、Spring Boot 启动配置、KJDP/Koca 业务逻辑、MyBatis、XML 元数据和代码评审。

它的核心流程是：

1. 阅读项目总规范。
2. 识别目标模块。
3. 追踪 `service XML -> ATOM/BEX XML -> Java 类/方法 -> MyBatis XML -> 测试或 mock 数据`。
4. 保持变更范围收敛并遵循邻近代码风格。

### aoi-service-registry

`skills/aoi-service-registry/SKILL.md` 专门处理 KJDP 服务注册与接口调用链，覆盖 `service_code`、`business_code`、`bex_codes`、`atom_code`、`bex_code`、`FrameDao` 调用、自动任务和业务任务配置。

它总结了 5 类常见入口：

1. 通用 service 携带 `bex_codes` 转发到 BEX。
2. `service_code` 通过 `business_code` 映射到 ATOM。
3. `service_code` 通过 `business_code` 映射到 BEX，再到 MyBatis。
4. 自动任务通过 `upm_qrtz_job_details.job_class_name` 配置 Java 类。
5. 业务任务通过 `opp_busi_trans_conf.class_name` 配置 Java 类。

### aoi-backend-test

`skills/aoi-backend-test/SKILL.md` 面向后端测试，覆盖 `fs-aoi-test`、JUnit、mock 资源、Maven Surefire skip 配置、测试 `application.yml`、mock tabledata/rpc/mcdata，以及 Java 后端变更验证。

常用命令：

```powershell
mvn -pl fs-aoi-test -DskipTests=false -Dtest=<TestClassName> test
mvn -pl fs-aoi-test -am test-compile -DskipTests
```

### git-commit-aigc

`skills/git-commit-aigc/SKILL.md` 定义 AI 生成或 AI 辅助修改代码时的提交信息规范。提交信息必须包含大写 `AIGC` 标记；如果用户要求标注采纳率，则追加 `，采纳率XX%`。

示例：

```text
ZQSPB-8954：补充网络投票上场文件。AIGC
ZQSPB-8954：补充网络投票上场文件。AIGC，采纳率80%
```

## Repowiki 内容

`repowiki/zh/content/项目概述.md` 提供项目简介、启动入口和主要模块说明。

`repowiki/zh/content/后端开发速查.md` 提供常用检索命令、常用构建命令和关键注意事项，适合作为 AI 处理任务前的快速上下文。

## 核心工作方式

处理 `fs-aoi-biz` 任务时，建议按以下顺序给 AI 提供上下文或让 AI 自行读取：

1. 先读 `skills/project-tech-and-standards.skill.md`，确认项目边界和总规范。
2. 开发任务读 `skills/aoi-java-backend/SKILL.md`。
3. 服务链路、接口路由、自动任务或业务任务配置读 `skills/aoi-service-registry/SKILL.md`。
4. 测试、新增用例、mock 数据或验证任务读 `skills/aoi-backend-test/SKILL.md`。
5. 提交代码时读 `skills/git-commit-aigc/SKILL.md`。
6. 需要快速概览时读 `repowiki/zh/content/项目概述.md` 和 `repowiki/zh/content/后端开发速查.md`。

## URL 与敏感地址脱敏

当前资料中未检索到真实 `http://` 或 `https://` URL，也未检索到真实 IPv4 地址。后续如果补充环境地址、接口地址或截图说明，请统一做脱敏处理：

- 将真实 IP 替换为保留示例地址，例如 `192.0.2.10`、`198.51.100.10` 或 `203.0.113.10`。
- 将真实端口替换为假端口，例如 `18080`、`19090`。
- 将真实地址 `http://<真实IP>:<真实端口>/` 写成类似 `http://192.0.2.10:18080/`。
- 内部系统域名、账号、token、库表连接串、Nexus 地址、Redis/Zookeeper 地址也应按同样规则替换为示例值。

## 常用命令

```powershell
rg -n "<服务码或业务码>" -g "*.xml" -g "*.java"
rg -n "FrameDao\.doAtomCall|FrameDao\.doBexCall|<atom-code>|<bex-code>" -g "*.java" -g "*.xml"
rg -n "<字段名或表名>" -g "*.java" -g "*.xml" -g "*.json"

mvn clean install -DskipTests
mvn -pl <module> -am clean install -DskipTests
mvn -pl fs-aoi-biz-starter -am clean package -DskipTests
mvn -pl fs-aoi-test -DskipTests=false -Dtest=<TestClassName> test
```

## 质量门禁

- 修改前必须追踪服务注册链路，不能只看单个 Java 类。
- Java 代码保持 Java 8 兼容。
- XML 注册必须保持 well-formed，服务码、ATOM 编码、BEX 编码不得重复。
- Map 字段、错误码、`prompt`、返回结构和 SQL 字段名必须与调用方契约一致。
- 金融、客户、账户、权限、审核、流程状态、外部适配类变更默认按高风险处理。
- 测试结论必须说明执行命令，以及测试是否真实执行或被 Surefire 配置跳过。
