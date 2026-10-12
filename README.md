# Langfuse v3 — open-source LLM observability, tracing, evals and prompt management. Deploy on Railway with one click.

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template/IWuNP-)

## Services
| Service | Image | Notes |
|---|---|---|
| langfuse-web | `langfuse/langfuse:3` | UI + API, port 3000, public domain, runs DB migrations |
| langfuse-worker | `langfuse/langfuse-worker:3` | async ingestion, port 3030, private |
| Postgres | Railway Postgres | transactional data |
| Redis | Railway Redis | queue/cache (`?family=0` for IPv6 private net) |
| ClickHouse | `clickhouse/clickhouse-server:24.8` | traces/observations, volume `/var/lib/clickhouse` |
| langfuse-bucket | Railway Storage Bucket | S3 event + media uploads |

## Variables (shared by web + worker)
DATABASE_URL=${{Postgres.DATABASE_URL}}
SALT=${{secret(32)}}  ENCRYPTION_KEY=64-hex (secret)  NEXTAUTH_SECRET=${{secret(32)}} (web)
NEXTAUTH_URL=https://${{RAILWAY_PUBLIC_DOMAIN}} (web)  HOSTNAME=::  PORT=3000 / 3030
CLICKHOUSE_URL=http://${{ClickHouse.RAILWAY_PRIVATE_DOMAIN}}:8123
CLICKHOUSE_MIGRATION_URL=clickhouse://${{ClickHouse.RAILWAY_PRIVATE_DOMAIN}}:9000
CLICKHOUSE_USER / CLICKHOUSE_PASSWORD -> ${{ClickHouse.*}}  CLICKHOUSE_CLUSTER_ENABLED=false
REDIS_CONNECTION_STRING=${{Redis.REDIS_URL}}?family=0
LANGFUSE_S3_{EVENT,MEDIA}_UPLOAD_{BUCKET,REGION,ENDPOINT,ACCESS_KEY_ID,SECRET_ACCESS_KEY} -> ${{langfuse-bucket.*}}
LANGFUSE_S3_*_FORCE_PATH_STYLE=false, prefixes events/ and media/
ClickHouse: CLICKHOUSE_USER=clickhouse, CLICKHOUSE_PASSWORD=${{secret(32)}}, CLICKHOUSE_DB=default

## Notes
- First boot: worker may log missing-table errors until web finishes Prisma migrations; it recovers (restart once if needed).
- Health: `/api/public/health`. Sign up at `/auth/sign-up` to create the first org.
