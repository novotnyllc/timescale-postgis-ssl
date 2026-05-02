# SSL-enabled PostgreSQL 18 + TimescaleDB + PostGIS

This repository builds the Railway image for PostgreSQL 18 with TimescaleDB, PostGIS, and SSL enabled. The published image supports `linux/amd64` and `linux/arm64`.

When you deploy the Railway template, the service runs the prebuilt image from GHCR rather than building this repository in Railway.

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.com/deploy/postgresql-18-with-timescaledb-postgis-a)

Template image: [`assets/template-icon.png`](assets/template-icon.png)

### Why though?

The [timescale/timescaledb-ha](https://hub.docker.com/r/timescale/timescaledb-ha) image in Docker Hub does not come with SSL baked in.

Since this could pose a problem for applications or services attempting to connect, we decided to roll our own image with SSL enabled.

### How does it work?

The current Railway template uses `Dockerfile.pg18-ts2.26`, which starts from `timescale/timescaledb-ha:pg18.3-ts2.26.4-oss`.

The upstream TimescaleDB HA image already includes the PostGIS packages. This repository adds the missing Railway/SSL behavior and first-init extension setup:

- `init-ssl.sh` creates PostgreSQL SSL certificates and enables SSL.
- `init-extensions.sql` runs `CREATE EXTENSION IF NOT EXISTS timescaledb;` and `CREATE EXTENSION IF NOT EXISTS postgis;` for the initial database.

The template mounts the Railway volume at `/home/postgres/pgdata` and sets:

```env
PGDATA=/home/postgres/pgdata/data
```

That keeps the Railway volume mount path as the parent directory while using PostgreSQL's current data-directory layout under `data`.

#### Wrapper script

The PostgreSQL 18 Dockerfile replaces the image entrypoint with `wrapper.sh`. The wrapper prepares the Railway volume, validates the expected mount path, then unsets `PGHOST` and `PGPORT` before handing off to the upstream TimescaleDB/PostgreSQL entrypoint.

The `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`, `DATABASE_URL`, and `POSTGRES_*` variables are present so Railway recognizes the service as a database and shows the Database tab.

For services running inside Railway, use `DATABASE_URL`; it points at the private Railway domain. `DATABASE_PUBLIC_URL` uses the TCP proxy and is intended for external clients.

#### Cert expiry
By default, the cert expiry is set to 820 days.  You can control this by configuring the `SSL_CERT_DAYS` environment variable as needed.

### Updates and security fixes

The GitHub Actions workflow builds only the current PostgreSQL 18 image and publishes a multi-architecture GHCR manifest for `linux/amd64` and `linux/arm64`.

The workflow runs:

- manually via `workflow_dispatch`;
- on pull requests that change the Dockerfile, init scripts, wrapper, or workflow, using a local `linux/amd64` build check;
- on pushes to `main` for those same files.

Each build uses `pull: true`, so when a build is intentionally triggered it starts from the current upstream base image. Pushes to `main` publish the multi-architecture `linux/amd64` and `linux/arm64` image. Dependabot is enabled for Dockerfile and GitHub Actions updates. It opens PRs for upstream tag/action updates that need review before changing the database base image version.
