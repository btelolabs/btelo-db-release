# BteloDB — Releases & Public Image

**BteloDB** is an **S3-first streaming + batch lakehouse database** written in Rust: one engine that unifies
**streams (Kafka-style append + live subscriptions) + SQL (S3 Parquet + DataFusion) + search (scan-based)**,
with all cold data stored as open-format Parquet on S3/R2.

This (public) repository publishes the official Docker image of the **data plane** — anyone can `docker run` a complete BteloDB locally.

> The commercial **Console** control plane (multi-tenant cloud dashboard) is not included in this image and is not published publicly.

---

## Quick start

```bash
# Pull and run (data defaults to ~/.btelo-db inside the container)
docker run --rm -p 4382:4382 -p 4380:4380 -p 4383:4383 -p 4384:4384 \
  ghcr.io/btelolabs/btelo-db-release:latest
```

All four protocol entry points are available at once:

| Port | Protocol | Use |
|---|---|---|
| `4382` | HTTP + JSON | `/v1/*` API · open **`http://localhost:4382/studio`** in a browser (the built-in admin console, Studio) |
| `4380` | Redis (RESP2) | native `redis-cli -p 4380` ("a Redis that doesn't lose data") |
| `4383` | Postgres (v3) | `psql -h localhost -p 4383`, SELECT/INSERT directly |
| `4384` | Kafka | Produce/Fetch (topic = table, one log per table) |

Open **http://localhost:4382/studio** to create tables, write streams, run SQL, and watch live subscriptions and the architecture map — all in the browser.

### Persist to the host

```bash
docker run --rm -p 4382:4382 \
  -v btelo-data:/root/.btelo-db \
  ghcr.io/btelolabs/btelo-db-release:latest
```

### Connect real S3 / Cloudflare R2 (cold data on object storage)

```bash
docker run --rm -p 4382:4382 \
  -e BTELO_S3_BUCKET=my-bucket \
  -e BTELO_S3_ENDPOINT=https://<accountid>.r2.cloudflarestorage.com \
  -e AWS_ACCESS_KEY_ID=... -e AWS_SECRET_ACCESS_KEY=... \
  ghcr.io/btelolabs/btelo-db-release:latest
```

Without S3 credentials it defaults to local object storage (`~/.btelo-db`) and works out of the box.

---

## Try it in five minutes

```bash
# 1) Create a table
curl -X PUT localhost:4382/v1/tables/events \
  -H 'authorization: Bearer tnt_demo_pro'

# 2) Append documents
curl -X POST localhost:4382/v1/tables/events/docs \
  -H 'authorization: Bearer tnt_demo_pro' -H 'content-type: application/json' \
  -d '[{"user":"u1","event":"click","n":1},{"user":"u2","event":"view","n":2}]'

# 3) Query with SQL
curl -X POST localhost:4382/v1/query \
  -H 'authorization: Bearer tnt_demo_pro' -H 'content-type: application/json' \
  -d '{"sql":"SELECT event, COUNT(*) c FROM events GROUP BY event ORDER BY c DESC"}'
```

> `tnt_demo_pro` is a dev token (for local testing). Use a JWT in production (`BTELO_JWT_SECRET`).

---

## Versions

See this repository's [Releases](../../releases) for per-version images and release notes.
Image tags: `:latest` and `:vX.Y.Z` (one-to-one with each Release).

## Feedback

The image and this repository are release artifacts; questions and suggestions are welcome as Issues here.
