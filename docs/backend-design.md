# OfferForge 后端设计文档

> 本文档记录 OfferForge 首期后端设计。目标是先把模块边界、核心流程、数据模型、API 设计和部署约束讲清楚,后续随着实现持续补充。

## 1. 背景与目标

### 1.1 背景

OfferForge 用于收集互联网大厂面经,辅助用户准备面试。项目核心关注 AI 相关面试,同时覆盖 Java 后端、前端等技术栈。

### 1.2 首期目标

- 支持爬虫采集后的面经数据入库。
- 支持按公司、技术栈、关键词、高频程度浏览和筛选面经/题目。
- 支持查看题目与答案。
- 支持基础模拟面试:按公司或技术栈随机生成题目。
- 支持 Web 前端调用后端 API。

### 1.3 非目标范围

- 首期不做完整用户账号体系。
- 首期不接入 OpenSearch/Elasticsearch。
- 首期不引入消息队列。
- 首期不拆分微服务。
- 首期不实现完整 AI 自动评分。

## 2. 技术方案概览

首期后端采用 Spring Boot 模块化单体。一个后端应用内按业务模块划分包和边界,统一部署。

| 维度 | 方案 |
| --- | --- |
| 语言 | Java 21 |
| 框架 | Spring Boot 3.x |
| 构建工具 | Maven |
| 数据库 | PostgreSQL |
| ORM / 持久层 | MyBatis-Plus |
| 缓存 | Redis |
| 数据库迁移 | Flyway |
| API 文档 | springdoc-openapi |
| 参数校验 | Spring Validation |
| 部署 | 云服务器 + Docker Compose |

## 3. 总体架构

### 3.1 系统边界

```text
+-------------+        +-------------------+        +----------------+
| Web Frontend| -----> | Spring Boot API   | -----> | PostgreSQL     |
+-------------+        | Modular Monolith  |        +----------------+
                       |                   |
+-------------+        |                   |        +----------------+
| Crawler     | -----> | Ingestion API     | -----> | Redis          |
+-------------+        +-------------------+        +----------------+
```

### 3.2 数据流

```text
内容平台 -> Python 爬虫 -> 结构化数据 -> 后端 ingestion API -> 校验/去重 -> PostgreSQL
用户 Web 操作 -> 后端 API -> PostgreSQL/Redis -> 返回列表、详情、题目、答案
```

## 4. 模块划分

### 4.1 模块列表

| 模块 | 职责 | 备注 |
| --- | --- | --- |
| common | 通用响应、异常处理、错误码、配置、工具类 | 不承载具体业务 |
| company | 公司信息、公司筛选、公司统计 | 被面经和题目模块引用 |
| interview | 面经内容、来源平台、原始内容、面经详情 | 核心内容模块 |
| question | 题目、答案、标签、高频统计 | 支撑浏览和模拟面试 |
| mockinterview | 模拟面试题单、用户作答 | 首期先做基础流程 |
| ingestion | 爬虫数据接收、校验、去重、入库 | 对 crawler 暴露入口 |

### 4.2 建议包结构

```text
com.offerforge
  OfferForgeApplication
  common
    config
    error
    response
    validation
  company
    controller
    service
    mapper
    model
  interview
    controller
    service
    mapper
    model
  question
    controller
    service
    mapper
    model
  mockinterview
    controller
    service
    mapper
    model
  ingestion
    controller
    service
    model
```

### 4.3 模块边界原则

- Controller 只负责请求接入、参数校验和响应转换。
- Service 负责业务规则、事务边界和跨表逻辑。
- Mapper 负责数据库访问。
- 各业务模块优先通过 Service 调用,避免跨模块直接访问对方 Mapper。
- common 不能反向依赖业务模块。

## 5. 核心业务流程

### 5.1 爬虫数据入库流程

```text
1. crawler 采集并清洗平台内容。
2. crawler 组装结构化请求。
3. 调用 ingestion API。
4. 后端校验必填字段。
5. 后端计算去重 key。
6. 判断公司、技术栈、题目是否已存在。
7. 写入面经、题目、答案、来源记录。
8. 返回入库成功、重复跳过或失败原因。
```

### 5.2 面经浏览与筛选流程

```text
1. 用户在前端选择公司、技术栈、关键词、排序条件。
2. 前端调用面经列表 API。
3. 后端根据筛选条件查询 PostgreSQL。
4. 高频或热点查询可读取 Redis 缓存。
5. 返回分页结果。
```

### 5.3 模拟面试流程

```text
1. 用户选择技术栈或公司。
2. 前端请求生成模拟面试题单。
3. 后端从题库中随机抽取题目。
4. 返回题目列表。
5. 用户作答并提交。
6. 首期保存作答记录或仅返回参考答案,后续再评估 AI 反馈。
```

## 6. 数据模型设计

> 本节先列候选表和核心字段,正式建表前需要进一步细化字段类型、索引、约束和 Flyway 脚本。

### 6.1 候选表

| 表 | 说明 |
| --- | --- |
| companies | 公司信息 |
| tech_stacks | 技术栈字典 |
| interviews | 面经主表 |
| interview_sources | 面经来源与原始内容 |
| questions | 题目表 |
| answers | 答案表 |
| tags | 标签表 |
| question_tags | 题目与标签关联表 |
| interview_questions | 面经与题目关联表 |
| mock_interviews | 模拟面试记录 |
| mock_interview_answers | 模拟面试作答记录 |

### 6.2 关键设计点

- 面经和题目需要支持去重。
- 原始爬虫内容可使用 PostgreSQL JSONB 保存。
- 高频题可以通过统计题目出现次数得到,也可以后续做冗余字段。
- 关键词搜索首期使用 PostgreSQL 全文检索。
- 需要为公司、技术栈、来源平台、创建时间建立常用索引。

## 7. API 设计

> 详细字段后续由 springdoc-openapi 生成并维护。本文档只记录接口分组、核心用途和关键参数。

### 7.1 公司 API

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| GET | `/api/companies` | 公司列表 |
| GET | `/api/companies/{id}` | 公司详情 |

### 7.2 面经 API

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| GET | `/api/interviews` | 面经列表与筛选 |
| GET | `/api/interviews/{id}` | 面经详情 |

### 7.3 题目与答案 API

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| GET | `/api/questions` | 题目列表与筛选 |
| GET | `/api/questions/high-frequency` | 高频题列表 |
| GET | `/api/questions/{id}` | 题目详情与答案 |

### 7.4 模拟面试 API

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| POST | `/api/mock-interviews` | 生成模拟面试题单 |
| POST | `/api/mock-interviews/{id}/answers` | 提交作答 |
| GET | `/api/mock-interviews/{id}` | 查看模拟面试记录 |

### 7.5 爬虫入库 API

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| POST | `/api/ingestions/interviews` | 接收爬虫结构化面经数据 |

## 8. 统一响应与错误码

### 8.1 统一响应结构

```json
{
  "code": "OK",
  "message": "success",
  "data": {}
}
```

### 8.2 常见错误码

| 错误码 | 含义 |
| --- | --- |
| OK | 成功 |
| BAD_REQUEST | 请求参数错误 |
| NOT_FOUND | 资源不存在 |
| DUPLICATED_DATA | 重复数据 |
| VALIDATION_FAILED | 参数校验失败 |
| INTERNAL_ERROR | 服务内部错误 |

## 9. 检索与筛选设计

- 首期使用 PostgreSQL 查询与全文检索。
- 支持按公司、技术栈、来源平台、关键词、出现频率筛选。
- 支持分页和排序。
- 搜索复杂度上来后再评估 OpenSearch/Elasticsearch。

## 10. 缓存设计

首期 Redis 用于:

- 热门公司/技术栈筛选项缓存。
- 高频题结果缓存。
- 简单频控/计数。

缓存原则:

- 先只缓存读多写少的数据。
- 缓存 key 需要有统一命名规范。
- 数据更新后需要考虑缓存失效。

## 11. 安全与权限

- 首期如不做账号体系,多数浏览 API 可公开访问。
- ingestion API 需要鉴权或至少使用内部 token,避免被外部随意写入数据。
- 生产环境敏感配置通过环境变量管理,不提交到 Git。
- 后续如加入用户体系,再评估 Spring Security + JWT / Session。

## 12. 测试方案

- 单元测试:Service 层核心规则。
- 集成测试:Controller + 数据库关键流程。
- 数据库迁移测试:Flyway 脚本可重复在空库执行。
- 接口测试:列表、详情、筛选、入库、模拟面试。
- 爬虫入库测试:重复数据、缺失字段、非法字段。

## 13. 部署方案

- 首期使用云服务器 + Docker Compose。
- PostgreSQL、Redis、后端、前端由 Compose 编排。
- 前端静态资源可通过 Nginx 提供。
- Nginx 反向代理后端 API。
- 生产环境需要配置 HTTPS、日志目录、数据卷和备份脚本。

## 14. 风险与后续演进

| 风险 | 影响 | 应对 |
| --- | --- | --- |
| 爬虫平台规则变化 | 采集失败或数据质量下降 | 保持平台适配层隔离,增加失败日志 |
| 数据去重不准确 | 重复题目或重复面经 | 设计多维去重规则,人工抽样检查 |
| 全文检索效果不足 | 搜索体验差 | 首期优化 PostgreSQL 检索,后续评估 OpenSearch |
| 数据质量参差不齐 | 用户体验下降 | 增加清洗规则、置信度和人工修正入口 |
| 功能范围膨胀 | 延误上线 | 严格控制 MVP 范围 |

## 15. 待确认问题

- [ ] 首期是否需要用户账号体系。
- [ ] 答案来源是爬取附带、人工整理还是 AI 生成。
- [ ] 模拟面试是否保存历史记录。
- [ ] ingestion API 的鉴权方式。
- [ ] PostgreSQL 全文检索的中文分词方案。
- [ ] 高频题统计是实时聚合还是定时计算。
