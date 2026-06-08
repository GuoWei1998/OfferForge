# 技术选型记录

> 记录 OfferForge 各部分的技术决策及理由。未定项标记为 **TBD**,确定后回填。

## 已确定

| 维度 | 决策 | 说明 |
| --- | --- | --- |
| 仓库形态 | Monorepo 单仓 | 前端 / 后端 / 爬虫同仓,顶层目录分离 |
| 后端主体 | Java 21 | 标准 Java 后端项目,采用当前长期支持版本作为首期基线 |
| 后端框架 | Spring Boot 3.x | 先采用清晰的模块化单体,不在首期引入微服务复杂度 |
| 后端架构 | 模块化单体 | 以业务模块划分包与边界,后续可按需要演进为微服务 |
| 后端构建工具 | Maven | 约定清晰、生态成熟,适合首期 Spring Boot 模块化单体快速落地 |
| 主数据库 | PostgreSQL | 适合结构化业务数据、JSONB 半结构化数据、基础全文检索与统计查询 |
| ORM / 持久层 | MyBatis-Plus | 基础 CRUD 与分页用 MyBatis-Plus 提效,复杂筛选、统计、全文检索保留手写 SQL |
| 缓存 | Redis | 用于热点查询缓存、频控/计数,后续可扩展到会话缓存、分布式锁等场景 |
| 数据库迁移 | Flyway | 管理数据库表结构版本,保证本地、测试、生产环境 schema 可追踪 |
| API 文档 | springdoc-openapi | 生成 OpenAPI/Swagger UI,方便前后端联调与接口检查 |
| 参数校验 | Spring Validation | 基于 Bean Validation 做请求参数校验,减少业务层重复判断 |
| 本地依赖编排 | Docker Compose | 首期用于本地启动 PostgreSQL、Redis 等基础依赖 |
| Web 前端框架 | React | 首期 Web 前端采用 React,适合复杂筛选、模拟面试等交互 |
| Web 前端构建 | Vite + TypeScript | 使用 Vite 提供轻量快速的开发体验,TypeScript 提升接口协作与可维护性 |
| 小程序预留方案 | Taro + React | 未来接入微信小程序时优先沿用 React 技术栈,按需复用业务逻辑 |
| 爬虫主体 | Python + Scrapy | 以 Scrapy 承载系统化采集、调度、重试、限速与数据管道 |
| 爬虫解析 | BeautifulSoup/lxml | 用于 HTML 内容解析、字段抽取与清洗辅助 |
| 动态页面采集 | Playwright | 仅在页面必须 JS 渲染或需要交互时按需使用,避免全量浏览器化 |
| 爬虫数据交付 | 后端 ingestion API | 爬虫产出结构化数据后优先提交给后端,由后端统一校验、去重与入库 |
| 检索/筛选 | PostgreSQL 查询与全文检索 | 首期使用 PostgreSQL 内建能力完成公司、技术栈、频率、关键词等筛选与基础搜索 |
| 线上部署 | 云服务器 + Docker Compose | 首期以单机容器化部署为主,降低运维复杂度;后续规模上来再评估 K8s |
| 形态 | 带前端的全栈项目 | 前端独立目录 |

## 后端后续待评估

| 方向 | 候选 | 暂缓原因 |
| --- | --- | --- |
| 认证鉴权 | Spring Security + JWT / Session | 首期是否需要用户账号、收藏、练习记录尚未确定 |
| 搜索引擎 | OpenSearch / Elasticsearch | 首期先使用 PostgreSQL 查询与全文检索,搜索复杂度上来后再引入 |
| 消息队列 | RabbitMQ / RocketMQ / Kafka | 首期可同步处理;爬虫入库、AI 生成答案、批处理任务变多后再评估 |
| 对象存储 | MinIO / S3 / 云厂商对象存储 | 暂无截图、附件、原始 HTML 文件等明确存储需求 |
| 定时任务 | Spring Scheduler / XXL-JOB | 简单定时任务可先用 Spring Scheduler;复杂调度再评估 XXL-JOB |
| 可观测性 | Spring Boot Actuator / Prometheus / Grafana / OpenTelemetry | 首期先保留 Actuator 和结构化日志入口,生产化后再接完整监控链路 |
| API 网关与微服务治理 | Spring Cloud Gateway / Nacos / Sentinel | 当前采用模块化单体,暂不引入微服务治理复杂度 |
| 云原生编排 | K8s | 首期使用云服务器 + Docker Compose;多实例、高可用和自动扩缩容需求明确后再评估 |

## 决策日志

- 2026-06-08:确定首期检索/筛选使用 PostgreSQL 查询与全文检索;线上部署采用云服务器 + Docker Compose。
- 2026-06-08:确定爬虫采用 Python + Scrapy,解析使用 BeautifulSoup/lxml,动态页面按需使用 Playwright;数据交付优先走后端 ingestion API。
- 2026-06-08:确定 Web 前端采用 React + Vite + TypeScript;未来微信小程序预留 Taro + React 路线。
- 2026-06-08:补充后端启动骨架必需项:Java 21、Spring Boot 3.x、Flyway、springdoc-openapi、Spring Validation、Docker Compose 本地依赖编排。
- 2026-06-08:确定缓存采用 Redis;首期重点用于热点数据缓存、频控和计数类场景。
- 2026-06-08:确定 ORM / 持久层采用 MyBatis-Plus;简单 CRUD 提效,复杂 PostgreSQL 查询仍使用清晰可控的手写 SQL。
- 2026-06-08:确定主数据库采用 PostgreSQL;首期利用关系模型、JSONB 与基础全文检索能力支撑面经存储、筛选和统计。
- 2026-06-08:确定后端构建工具采用 Maven;优先选择稳定、低学习成本、Spring Boot 支持成熟的方案。
- 2026-06-08:确定后端采用 Spring Boot 模块化单体;首期先保证边界清晰和快速落地,暂不引入微服务治理组件。
- 2026-06-07:确定 Monorepo 单仓;后端以 Java 为主体;先建空目录骨架,选型逐步落地。
