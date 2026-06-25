---
name: aoi-backend-test
description: 运行、新增或修复 fs-aoi-biz 后端测试。处理 fs-aoi-test、JUnit 测试、mock 资源、Maven Surefire skip 配置、test application.yml、mock tabledata/rpc/mcdata，或验证 Java 后端变更时使用。
---

# AOI 后端测试

## 测试模块事实

- 测试主要位于 `fs-aoi-test`。
- 测试辅助类位于 `fs-aoi-test/src/main/java`，包含 mock request、common-param、data-exchange 等工具。
- 测试用例位于 `fs-aoi-test/src/test/java`。
- mock 资源位于 `fs-aoi-test/src/test/resources/mock`。
- `fs-aoi-test/pom.xml` 将 `maven-surefire-plugin` 配置为 `skip=true`，普通根构建会跳过这些测试。

## 编写测试前

1. 先搜索目标业务附近是否已有测试。
2. 复用 `TestMain`、`MockCommonParam`、`MockDataExchangeAssembly`、`MockGenericRequest` 和既有 mock JSON 模式。
3. 优先编写聚焦的单元或业务流测试，避免过度依赖完整环境。
4. 测试数据尽量放在 `fs-aoi-test/src/test/resources/mock` 下。

## 运行测试

```powershell
mvn -pl fs-aoi-test -DskipTests=false -Dtest=<TestClassName> test
mvn -pl fs-aoi-test -am test-compile -DskipTests
```

如果 Surefire 配置或项目装配仍导致测试跳过，要准确说明实际行为，并把 IDE 中定点运行类/方法作为备选建议。除非输出明确显示测试已执行，否则不要声称测试已运行。

## 测试设计清单

- 金融、客户、账户相关变更要覆盖正常路径、关键字段缺失、外部 BEX/ATOM 失败和边界值。
- 断言 `GenericResult`/`AtomResult` 的 code、prompt 和关键 data 字段。
- Map 密集型代码要断言精确的大写字段名。
- 流程类变更要验证任务状态、认领/分派行为，以及必要的幂等性。
- SQL/BEX 变更要覆盖真实 mock 表数据、无数据场景；业务要求唯一时，还要覆盖多数据场景。

## 结果说明

最终回复中说明测试命令、测试是否真实执行或被跳过，以及是否存在内部 Nexus、数据库、Redis、Zookeeper、JNI、mock 数据缺失等阻塞。
