# deploy

OfferForge 的 **部署与运维配置**:容器编排、环境配置、数据库初始化等。

## 预期内容

- 容器编排:首期使用 Docker Compose。
- 各环境配置(本地 / 测试 / 生产)。
- 数据库初始化脚本。
- 后续按规模评估 K8s 等云原生编排方案。

## 状态

最小后端服务的 Docker Compose 配置已就位。首期线上部署方案为云服务器 + Docker Compose,见 [../docs/tech-stack.md](../docs/tech-stack.md)。

## 最小部署

服务器需要安装 Git、Docker 和 Docker Compose。拉取代码后执行:

```bash
cd OfferForge/deploy
docker compose up -d --build
```

验证服务:

```bash
curl http://localhost:8080/api/health
```

查看日志:

```bash
docker compose logs -f backend
```

停止服务:

```bash
docker compose down
```
