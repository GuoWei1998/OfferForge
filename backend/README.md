# backend

OfferForge 的 **Java 后端服务**:对外提供 API,承载面经的归类、存储、检索/筛选,以及模拟面试题目的生成等业务逻辑。

## 职责

- 面经数据的存储与归类(按公司、技术栈)。
- 面经检索与多维筛选 API(公司 / 频率 / 技术栈)。
- 模拟面试:题目生成与作答相关接口。
- 为前端提供数据接口;消费爬虫产出的数据。

## 状态

🚧 空骨架。首期后端技术栈已确定为 Java 21 + Spring Boot 3.x + Maven + PostgreSQL + MyBatis-Plus + Redis,见 [../docs/tech-stack.md](../docs/tech-stack.md)。
