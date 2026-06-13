# BteloDB — Release & 公开镜像

**BteloDB** 是用 Rust 写的 **S3-First 流批一体湖仓数据库**：一个引擎统一
**流（Kafka 式追加 + 实时订阅）+ SQL（S3 Parquet + DataFusion）+ 检索（扫描式）**，
冷数据全部落开放格式 Parquet on S3/R2。

本仓库（public）发布**数据面**的官方 Docker 镜像 —— 任何人都能 `docker run` 在本地起一套完整的 BteloDB。

> 商业化**控制面 Console**（多租户云平台 Dashboard）不在本镜像内，也不公开发布。

---

## 快速开始

```bash
# 拉取并启动（数据默认落容器内 ~/.btelo-db）
docker run --rm -p 4382:4382 -p 4380:4380 -p 4383:4383 -p 4384:4384 \
  ghcr.io/btelolabs/btelo-db-release:latest
```

启动后四个协议入口同时可用：

| 端口 | 协议 | 用途 |
|---|---|---|
| `4382` | HTTP + JSON | `/v1/*` API · 浏览器打开 **`http://localhost:4382/studio`**（内置管理控制台 Studio） |
| `4380` | Redis (RESP2) | `redis-cli -p 4380` 原生互通（「不会丢数据的 Redis」） |
| `4383` | Postgres (v3) | `psql -h localhost -p 4383` SELECT/INSERT 直连 |
| `4384` | Kafka | Produce/Fetch（topic = 表，一表一序） |

打开 **http://localhost:4382/studio** 即可在浏览器里建表、写流、跑 SQL、看实时订阅与架构图。

### 持久化到宿主机

```bash
docker run --rm -p 4382:4382 \
  -v btelo-data:/root/.btelo-db \
  ghcr.io/btelolabs/btelo-db-release:latest
```

### 接真 S3 / Cloudflare R2（冷数据上对象存储）

```bash
docker run --rm -p 4382:4382 \
  -e BTELO_S3_BUCKET=my-bucket \
  -e BTELO_S3_ENDPOINT=https://<accountid>.r2.cloudflarestorage.com \
  -e AWS_ACCESS_KEY_ID=... -e AWS_SECRET_ACCESS_KEY=... \
  ghcr.io/btelolabs/btelo-db-release:latest
```

无 S3 凭据时默认用本地对象存储（`~/.btelo-db`），开箱即用。

---

## 五分钟试一下

```bash
# 1) 建表
curl -X PUT localhost:4382/v1/tables/events \
  -H 'authorization: Bearer tnt_demo_pro'

# 2) 追加文档
curl -X POST localhost:4382/v1/tables/events/docs \
  -H 'authorization: Bearer tnt_demo_pro' -H 'content-type: application/json' \
  -d '[{"user":"u1","event":"click","n":1},{"user":"u2","event":"view","n":2}]'

# 3) SQL 查询
curl -X POST localhost:4382/v1/query \
  -H 'authorization: Bearer tnt_demo_pro' -H 'content-type: application/json' \
  -d '{"sql":"SELECT event, COUNT(*) c FROM events GROUP BY event ORDER BY c DESC"}'
```

> `tnt_demo_pro` 是 dev token（本地测试态）。生产用 JWT（`BTELO_JWT_SECRET`）。

---

## 版本

各版本镜像与发行说明见本仓库的 [Releases](../../releases)。
镜像标签：`:latest` 与 `:vX.Y.Z`（与 Release 一一对应）。

## 反馈

镜像与本仓库是发布产物；问题与建议欢迎在本仓库提 Issue。
