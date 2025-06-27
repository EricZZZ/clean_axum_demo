# Clean Axum Demo

一个使用 Axum 和 SQLx 的极简、领域驱动的 Rust API 服务器模板。  
专为清晰性、可扩展性和快速开发而设计。

---

## 🔧 特性

- **整洁架构**：领域层、基础设施层和 API 层的清晰分离
- **模块化领域**：自包含的功能模块（认证、用户、设备、文件）
- **SQLx 集成**：离线模式下的编译时检查查询
- **JWT 认证**：安全的身份验证和授权
- **文件上传**：异步处理和安全的资源服务
- **OpenAPI 文档**：由 Utoipa 驱动的 Swagger UI
- **可观测性**：OpenTelemetry 追踪和指标
- **测试**：使用 `tokio::test` 和 `tower::ServiceExt` 的单元测试和集成测试

---

## 📦 项目结构

推荐布局：

```text
├── src/
│   ├── main.rs                         # 应用程序入口点
│   ├── app.rs                          # 路由设置和中间件
│   ├── lib.rs                          # 模块声明
│   ├── common.rs
│   ├── common/                         # 共享组件和工具
│   │   ├── app_state.rs                # 用于依赖注入的 AppState 结构体
│   │   ├── bootstrap.rs                # 服务初始化和 AppState 构建
│   │   ├── config.rs                   # 环境变量配置加载器
│   │   ├── dto.rs                      # 共享/全局 DTOs
│   │   ├── error.rs                    # AppError 枚举和错误映射器
│   │   ├── hash_util.rs                # 哈希工具（如 bcrypt）
│   │   ├── jwt.rs                      # JWT 编码、解码和验证
│   │   ├── multipart.rs                # 多部分助手
│   │   ├── opentelemetry.rs            # OpenTelemetry 设置
│   │   └── ts_format.rs                # 自定义时间戳序列化格式

│   ├── domains.rs                      # 领域模块声明
│   ├── domains/                        # 功能模块
│   │   ├── <feature>/                  # 例如：auth、user、device、file
│   │   │   ├── api/
│   │   │   │   ├── handlers.rs         # 路由处理器
│   │   │   │   └── routes.rs           # 路由定义
│   │   │   ├── domain/                 # 领域模型、特征
│   │   │   │   ├── model.rs
│   │   │   │   ├── repository.rs
│   │   │   │   └── service.rs
│   │   │   ├── dto/                    # 数据传输对象
│   │   │   │   └── <feature>_dto.rs
│   │   │   └── infra/                  # 基础设施层实现
│   │   │       ├── impl_repository.rs
│   │   │       └── impl_service.rs
│   │   ├── <feature>.rs                 # 模块入口点

├── tests/
│   ├── asset/
│   ├── test_helpers.rs                 # 测试的共享设置和工具
│   └── test_<feature>_routes.rs
├── .env                                # 本地开发的环境变量
├── .env.test                           # 测试环境的环境变量覆盖
```

添加新功能模块时，需要在以下文件中注册：

> - `src/domains.rs`
> - `src/app.rs`
> - `src/common/app_state.rs`
> - `src/common/bootstrap.rs`

---

## 🛠 快速开始

### 前置条件

- Rust（最新稳定版）
- PostgreSQL
- Docker & Docker Compose（可选）

### 快速启动

选择您偏好的设置方式：

**使用 Docker Compose：**

```bash
docker-compose up --build
```

停止并清理：

```bash
docker-compose down --rmi all
```

**手动设置：**

1. 创建数据库表和种子数据：

   ```bash
   db-seed/01-tables.sql
   db-seed/02-seed.sql
   ```

2. 在 `.env` 中配置环境变量：

   ```env
   DATABASE_URL=postgres://testuser:pass@localhost:5432/testdb
   JWT_SECRET_KEY=your_super_secret_key
   SERVICE_PORT=8080
   ```

3. 准备 SQLx 离线模式并验证：

   ```bash
   cargo sqlx prepare --check
   ```

4. 本地运行服务器：

   ```bash
   cargo run
   ```

---

## 🚀 使用方法

### 身份验证和调用受保护的 API

1. 登录以获取 JWT 令牌：

   ```bash
   curl -X POST http://localhost:8080/auth/login \
     -H "Content-Type: application/json" \
     -d '{"client_id":"apitest01","client_secret":"test_password"}'
   ```

2. 使用返回的 `token` 访问受保护的端点：

   ```bash
   curl http://localhost:8080/user -H "Authorization: Bearer $token"
   ```

### API 文档

在浏览器中打开 [http://localhost:8080/docs](http://localhost:8080/docs) 查看 Swagger UI。

- 通过 `/auth/login`（POST）使用 JSON 负载进行身份验证：

  ```json
  {
    "client_id": "apitest01",
    "client_secret": "test_password"
  }
  ```

- 复制返回的 JWT 令牌。
- 在 Swagger UI 中点击 🔒 授权按钮并粘贴令牌以授权请求。

---

## 💡 架构

- **领域层**：特征和模型定义核心业务逻辑。
- **基础设施层**：具体实现（SQLx 仓库、服务）
- **API 层**：Axum 处理器和路由定义
- **DTOs**：类型化的请求/响应契约
- **引导程序**：将依赖项连接到 `AppState`

1. 创建 `domains/<feature>/`，包含 `api/`、`domain/`、`infra/`、`dto/`
2. 在 `domains.rs`、`app.rs`、`common/app_state.rs`、`common/bootstrap.rs` 中注册

## 🧠 领域驱动设计与架构

### 领域层

- `model.rs`：包含代表实体或值对象的核心结构体和枚举。
- **模型类型参考**：Rust 和 PostgreSQL 类型之间的转换。  
  [查看 SQLx Postgres 类型映射](https://docs.rs/sqlx/latest/sqlx/postgres/types/index.html)
- `repository.rs`：声明封装功能持久化操作的特征（例如 `UserRepository`）。
- `service.rs`：声明功能服务操作的特征。

### 基础设施层

每个功能都拥有自己的 `impl_repository.rs` 和 `impl_service.rs`

`sqlx::query`

- 运行时检查
- 灵活性：当 SQL 必须动态构建时很方便——例如，动态添加 WHERE 子句。

`sqlx::query!`

- 编译时检查：宏在构建时读取您的 SQL（如果配置为"离线模式"）并根据您的数据库模式进行验证。拼写错误的列名或错误的参数类型会变成编译器错误，而不是运行时意外。
- 自动类型推断：您在 SQL 字符串后列出 Rust 值，SQLx 会找出如何将它们映射到占位符类型（$1、$2、...）。
- 结构级安全性：如果您使用 query_as!，它还会确认您选择的列与目标结构体的字段匹配。

### API 层

- 路由处理器接受 DTOs，调用功能逻辑，并返回序列化响应。
- 每个功能都拥有自己的 `routes.rs` 和 `handlers.rs`。
- 支持带验证的异步多部分文件上传。
- 安全文件服务验证用户权限并清理文件路径。

### DTOs 和验证

- 请求和响应 DTOs 位于每个功能的 `dto.rs` 中。
- DTOs 和功能模型之间的显式映射。
- 使用 `serde` 和可选的 [validator](https://docs.rs/validator) crate 进行输入验证。

### 用例隔离和依赖倒置

- 领域服务特征定义业务契约。
- 具体实现位于 `impl_service.rs` 中，通过工厂方法构建。
- `bootstrap.rs` 连接服务并构建 `AppState` 用于依赖注入。

## 从"CREATE TABLE"脚本自动生成领域代码

- [domain_codegen](https://github.com/sukjaelee/domain_codegen) 项目提供了专为 clean_axum_demo 项目设计的代码生成器。它会在 gen/ 下自动生成功能层结构，您可以根据需要复制和自定义。

---

## 🧱 数据库模式

查看 `db-seed/` 目录了解表定义和示例数据。  
数据库结构在实体关系图中说明：

![ER 图](./ERD.png)

---

## 📚 API 文档

- Swagger UI 可在 `/docs` 获取（由 Utoipa 驱动）。在浏览器中打开 [http://localhost:8080/docs](http://localhost:8080/docs) 查看 Swagger UI。
- DTOs 和端点已为 OpenAPI 规范添加注释。

---

## 📦 API 响应格式

所有端点返回一致的 JSON 信封：

```json
{
  "status": 200,
  "message": "success",
  "data": { ... }
}
```

实现为：

- `ApiResponse<T>` – 通用响应包装器
- `RestApiResponse<T>` – 实现 Axum 的 `IntoResponse` 特征的包装器

查看 `common/dto.rs` 中的定义。

---

## 🧪 测试

- 单元测试覆盖功能逻辑和核心组件。
- 集成测试练习 HTTP 端点和数据库交互。
- 使用 `#[tokio::test]` 和 `tower::ServiceExt` 进行 HTTP 模拟。
- 测试资源和助手位于 `tests/` 目录中。

---

## 🚨 错误处理

- 集中式 `AppError` 枚举实现 `IntoResponse`。
- 错误映射到适当的 HTTP 状态码和 JSON 结构，例如：

```json
{
  "status": 400,
  "message": "Invalid request data",
  "data": null
}
```

---

## 🧪 环境配置

通过项目根目录的 `.env` 进行配置。  
设置数据库 URL、JWT 密钥、服务端口和资源设置。

示例 `.env`：

```env
DATABASE_URL=postgres://testuser:pass@localhost:5432/testdb
JWT_SECRET_KEY=your_super_secret_key
SERVICE_PORT=8080
```

---

## 📡 OpenTelemetry（追踪和指标）

此项目通过 OpenTelemetry 支持分布式追踪、日志记录和指标。

### 设置 Jaeger 收集器：

```bash
docker run --rm --name jaeger \
  -e COLLECTOR_OTLP_ENABLED=true \
  -p 16686:16686 \
  -p 4317:4317 \
  -p 4318:4318 \
  -p 5778:5778 \
  -p 9411:9411 \
  jaegertracing/jaeger:2.6.0
```

在 [http://localhost:16686](http://localhost:16686) 访问 Jaeger UI。

### 启用 OpenTelemetry 功能：

- 使用 OpenTelemetry 运行：

  ```bash
  cargo run --features opentelemetry
  ```

- 使用 OpenTelemetry 构建：

  ```bash
  cargo build --features opentelemetry
  ```

详情请参阅 [Jaeger 入门指南](https://www.jaegertracing.io/docs/2.6/getting-started/)。

---

## 🤝 贡献

欢迎贡献！随时打开问题、建议改进或提交包含您想法的拉取请求。

---

## 📄 许可证和资源

- MIT 许可证。详情请参阅 [LICENSE](./LICENSE)。

### 有用链接

- [Axum](https://docs.rs/axum)
- [SQLx](https://docs.rs/sqlx)
- [Utoipa (OpenAPI)](https://docs.rs/utoipa)
- [Tokio](https://tokio.rs/)
- [Validator (crate)](https://docs.rs/validator)