# 技术选型记录

> 记录 OfferForge 各部分的技术决策及理由。未定项标记为 **TBD**,确定后回填。

## 已确定

| 维度 | 决策 | 说明 |
| --- | --- | --- |
| 仓库形态 | Monorepo 单仓 | 前端 / 后端 / 爬虫同仓,顶层目录分离 |
| 后端主体 | Java | 标准 Java 后端项目 |
| 形态 | 带前端的全栈项目 | 前端独立目录 |

## 待定(TBD)

| 维度 | 候选 | 决策 |
| --- | --- | --- |
| 后端框架 | Spring Boot / ... | TBD |
| 构建工具 | Maven / Gradle | TBD |
| 数据库 | MySQL / PostgreSQL / ... | TBD |
| ORM | MyBatis / JPA / ... | TBD |
| 前端框架 | React / Vue 3 / ... | TBD |
| 前端构建 | Vite / ... | TBD |
| 爬虫技术 | Python(Scrapy/requests) / Java / ... | TBD |
| 检索/筛选 | 数据库查询 / Elasticsearch | TBD |
| 部署 | Docker Compose / K8s / ... | TBD |

## 决策日志

- 2026-06-07:确定 Monorepo 单仓;后端以 Java 为主体;先建空目录骨架,选型逐步落地。
