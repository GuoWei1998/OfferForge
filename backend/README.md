# backend

OfferForge 的 **Java 后端服务**:对外提供 API,承载面经的归类、存储、检索/筛选,以及模拟面试题目的生成等业务逻辑。

## 职责

- 面经数据的存储与归类(按公司、技术栈)。
- 面经检索与多维筛选 API(公司 / 频率 / 技术栈)。
- 模拟面试:题目生成与作答相关接口。
- 为前端提供数据接口;消费爬虫产出的数据。

## 状态

最小 Spring Boot 工程已初始化,当前提供健康检查接口。后续将逐步接入 PostgreSQL、MyBatis-Plus、Redis、Flyway 等能力。

## 本地运行

```bash
./mvnw spring-boot:run
```

启动后访问:

- `http://localhost:8080/api/health`
- `http://localhost:8080/actuator/health`

运行测试:

```bash
./mvnw test
```

## Docker

```bash
docker build -t offerforge-backend .
docker run --rm -p 8080:8080 offerforge-backend
```

完整技术选型见 [../docs/tech-stack.md](../docs/tech-stack.md)。
