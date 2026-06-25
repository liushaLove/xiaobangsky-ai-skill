---
name: aoi-service-registry
description: 新增、修改、追踪或排查 fs-aoi-biz 的 KJDP 服务注册与后端接口调用链。任务提到 service_code、business_code、bex_codes、atom_code、bex_code、META-INF/service、META-INF/atom、META-INF/bex、META-INF/mybatis、FrameDao 调用、自动任务、upm_qrtz_job_details、opp_busi_trans_conf 或后端服务路由时使用。
---

# AOI 服务注册与接口调用链

## 注册模型

KJDP 服务暴露主要由元数据驱动：

- `META-INF/service/*.xml`：对外服务定义。`service_code` 是请求侧服务码，`service_business.business_code` 路由到具体业务单元。
- `META-INF/atom/*.xml`：ATOM 定义。`atom_code` 路由到 `atom_class` 和 `atom_method`。
- `META-INF/bex/*.xml`：BEX 定义。`bex_code` 通常对应数据库查询、更新、批量处理或外部系统调用。
- `META-INF/mybatis/*.xml`：BEX/DAO 逻辑使用的 SQL 语句和 mapper namespace。
- Java 代码可能通过 `FrameDao.doAtomCall`、`FrameDao.doBexCall`、`FrameDao.doAtomInvoke`、`FrameDao.doBexInvoke` 调用已注册单元。
- 除 XML 外，还存在数据库配置驱动的入口，例如 `upm_qrtz_job_details.job_class_name` 和 `opp_busi_trans_conf.class_name`。

## 接口开发的五种常见模式

### 1. 通用服务码转发 BEX 编码

典型入口：前端调用类似 `P9999999`、`Y3000001` 这类通用 service 接口，请求入参中携带 `bex_codes` 或类似字段，例如 `bex_codes:YGT_A1100807`。

调用链：

```text
前端 service=P9999999/Y3000001
  -> 入参携带 bex_codes=YGT_A1100807
  -> 后端通用服务解析 bex_codes
  -> FrameDao.doBexCall / doBexInvoke("YGT_A1100807", ...)
  -> META-INF/bex 中查找 bex_code="YGT_A1100807"
  -> 根据 bex_type/attr1 决定后续目标
```

账户相关 BEX 要继续区分新老账户：

- 新账户模式：走类似 `fs-uas-act://F011100807` 的服务发现模式，通常对接 UAS/新账户服务。
- 老账户模式：走类似 `KUAS.L1100807` 的老账户模式，会调用老账户 DLL 或 KUAS 侧能力。

以 `YGT_A1100807` 为例，项目中可见类似：

```xml
<bex bex_code="YGT_A1100807" bex_type="9" page_mode="0" bex_cate="1" attr1="KUAS.L1100807" />
```

追踪这类接口时，先搜 `service` 与 `bex_codes`，再搜具体 BEX 编码：

```powershell
rg -n "Y3000001|P9999999|bex_codes|YGT_A1100807" -g "*.xml" -g "*.java"
rg -n "bex_code=\"YGT_A1100807\"|attr1=\"KUAS\.L1100807\"|fs-uas-act://F011100807" -g "*.xml" -g "*.java"
```

### 2. service_code 通过 business_code 映射到 ATOM

典型入口：前端调用类似 `Y1001137` 的 service 接口，`META-INF/service/*.xml` 中的 `service_business.business_code` 指向 ATOM 编码，ATOM 再映射到 Java 类和方法。

调用链：

```text
前端 service=Y1001137
  -> META-INF/service/*.xml 找 service_code="Y1001137"
  -> 读取 service_business.business_code
  -> business_code 对应 META-INF/atom/*.xml 的 atom_code
  -> atom_code 映射 atom_class + atom_method
  -> 反射/框架调用 Java 类的方法
  -> 方法内可能继续调用 BEX/ATOM/外部服务
```

追踪命令：

```powershell
rg -n "service_code=\"Y1001137\"|Y1001137" -g "*.xml"
rg -n "<business_code>|atom_code=\"<business_code>\"" -g "*.xml" -g "*.java"
rg -n "<atom_class>|<atom_method>" -g "*.java" -g "*.xml"
```

新增或修改这类接口时，要同时检查：

- `service_code` 是否已存在。
- `service_business.business_code` 是否能在 atom XML 中找到。
- `atom_class` 是否在启动扫描范围或依赖模块内。
- `atom_method` 的参数、返回值是否符合邻近 ATOM 调用方式。

### 3. service_code 通过 business_code 映射到 BEX，再映射 MyBatis

典型入口：前端调用类似 `YG000342` 的 service 接口，`business_code` 指向 BEX，BEX 再映射到 MyBatis 层中的 SQL id。

调用链：

```text
前端 service=YG000342
  -> META-INF/service/*.xml 找 service_code="YG000342"
  -> 读取 service_business.business_code
  -> business_code 对应 META-INF/bex/*.xml 的 bex_code
  -> BEX 配置定位 mapper namespace / SQL id / attr1
  -> META-INF/mybatis/*.xml 执行对应 select/insert/update/delete
  -> 返回 GenericResult / 列表 / 分页数据
```

追踪命令：

```powershell
rg -n "service_code=\"YG000342\"|YG000342" -g "*.xml"
rg -n "bex_code=\"<business_code>\"|<business_code>" -g "*.xml" -g "*.java"
rg -n "id=\"<mapper-id>\"|namespace=\"<namespace>\"|<表名>|<字段名>" -g "*.xml"
```

修改这类接口时要重点检查：

- BEX 分页模式、`page_mode`、`bex_type` 是否与邻近配置一致。
- MyBatis 入参字段是否与前端入参/通用服务转换后的字段一致。
- SQL 是否兼容 Oracle、MySQL、SQL Server、openGauss 等目标数据库。
- 返回字段名是否被前端或后续 Java 逻辑依赖。

### 4. 自动任务通过 upm_qrtz_job_details.job_class_name 配置 Java 类

典型场景：自动任务，例如“预受理业务”任务 `ApptBusiDataJob`。任务入口不一定来自前端 service，而是 Quartz/调度体系读取表配置。

调用链：

```text
调度器触发任务
  -> 查询 upm_qrtz_job_details
  -> 读取 job_class_name
  -> 例如 com.szkingdom.kdop.peripheral.jobs.ApptBusiDataJob
  -> 实例化/执行实现了 Job 的 Java 类
  -> execute/job 上下文中查询业务数据、组装 commParams
  -> 调用业务处理类、ATOM/BEX、流程发起或外部服务
```

项目示例：

```text
com.szkingdom.kdop.peripheral.jobs.ApptBusiDataJob
com.szkingdom.kdop.peripheral.jobs.ApptBusiDataJobSingle
```

追踪命令：

```powershell
rg -n "ApptBusiDataJob|job_class_name|upm_qrtz_job_details" -g "*.java" -g "*.xml" -g "*.sql" -g "*.txt"
rg -n "class ApptBusiDataJob|implements Job|execute\(" -g "*.java"
```

修改自动任务时要重点确认：

- 表中 `job_class_name` 配置的完整类名是否与 Java 类一致。
- 类是否实现调度框架期望的接口，例如 `Job`，以及是否继承项目上下文基类如 `PiJobContext`。
- 任务是否支持重复执行、断点恢复、并发控制和失败重试。
- 任务使用的公共参数、机构、操作员、渠道等字段是否完整。
- 如果任务会发起流程或更新业务状态，必须检查幂等性和异常回滚路径。

### 5. 业务任务配置通过 opp_busi_trans_conf.class_name 配置 Java 类

典型场景：业务流程中的处理任务由表 `opp_busi_trans_conf` 配置，字段 `class_name` 指向具体 Java 处理类。例如专业投资者续期业务，跑批配置业务任务为：

```text
com.szkingdom.opp.atom.business.appropriateManage.OpenProfessionalInvesterRight
```

调用链：

```text
业务受理/流程推进/跑批任务
  -> 根据业务代码、节点或转换配置查询 opp_busi_trans_conf
  -> 读取 class_name
  -> 反射/工厂实例化 Java 处理类
  -> 执行处理接口方法，例如校验、处理、确认、回滚等
  -> 类内部调用 BEX/ATOM/外部账户系统
  -> 更新 OPP_BUSI_DATA、流程状态或业务结果
```

追踪命令：

```powershell
rg -n "opp_busi_trans_conf|class_name|OpenProfessionalInvesterRight" -g "*.java" -g "*.xml" -g "*.sql" -g "*.txt"
rg -n "class OpenProfessionalInvesterRight|OppBaseHandleProcess|OppBaseConfirmProcess|process\(|confirm\(" -g "*.java"
```

修改这类业务任务时要重点确认：

- `opp_busi_trans_conf.class_name` 是否为完整类名，包名和类名不能写错。
- Java 类是否实现该业务阶段要求的接口，例如 `OppBaseHandleProcess`、`OppBaseConfirmProcess` 或邻近配置使用的处理接口。
- 类内部是否依赖 `DataExchangeAssembly` 中的 `params`、`commParams`、`businessParams`、`caches` 等上下文数据。
- 业务任务是否会被自动任务、人工审核、流程回退重复触发，必要时要保证幂等。
- 专业投资者、账户、适当性、协议签署等场景通常影响客户权限或账户状态，默认按高风险变更处理。

## 通用链路追踪流程

1. 先判断入口类型：通用 service+BEX 入参、service->ATOM、service->BEX/MyBatis、自动任务、业务任务配置。
2. 用已知编码在 XML、Java、SQL/脚本和 mock 数据中全局搜索。
3. 如果从 `service_code` 开始，先打开对应 `service_business`，查看每个 `business_code`。
4. 如果目标是 ATOM，在 `META-INF/atom/*.xml` 中找到 `atom_code`，再打开 `atom_class` 和 `atom_method`。
5. 如果目标是 BEX，在 `META-INF/bex/*.xml` 中找到 `bex_code`，再追踪 mapper XML、SQL id、`attr1` 和表字段。
6. 如果目标来自数据库配置，查表字段名对应的 Java 反射/工厂/调度入口。
7. 修改方法签名、入参字段、返回字段或状态流转前，先反查所有调用方。

## 新增或修改服务

- 新 XML 条目放在同模块、同业务附近。
- 命名保持一致：服务码常见 `P...`、`F...`、`Y...`；业务码常见前缀包括 `FRAME_`、`KCOP_`、`KDOP_`、`YZT_`、`KUAS_`。
- 不要复用已有 service、atom、bex 编码表达不同含义。
- 保留邻近条目使用的 XML 属性，例如 `service_type`、`service_cate`、`arrange_type`、`soap_enable`、`page_mode`、`attr*`。
- 绑定 Java 方法时，确认方法可见性、参数形态和框架调用方式匹配。
- 新增 SQL 时，检查数据库兼容性，并确认是否需要补充 `fs-aoi-test/src/test/resources/mock` 下的 mock 数据。
- 新增自动任务或业务任务配置时，同步提供表配置字段、完整类名、触发条件、幂等策略和验证方式。

## 验证方式

```powershell
rg -n "<新增编码或类名>" -g "*.xml" -g "*.java" -g "*.sql" -g "*.txt"
rg -n "atom_class=\"<完整类名>\"|atom_method=\"<方法名>\"" -g "*.xml"
rg -n "bex_code=\"<BEX编码>\"|id=\"<mapper-id>\"" -g "*.xml"
mvn -pl <所属模块> -am clean install -DskipTests
```

如果只是 XML 或表配置变更，也要检查：

- XML 标签闭合和属性是否与周边条目一致。
- 表配置中的类名是否能在源码中找到。
- 类是否在当前模块依赖和运行时 classpath 内。
- 任务或服务是否需要补充 mock 数据、数据库初始化脚本或人工验证 SQL。
