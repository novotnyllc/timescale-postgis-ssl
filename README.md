# SSL-enabled PostgreSQL 18 + TimescaleDB + PostGIS

This repository builds the Railway image for PostgreSQL 18 with TimescaleDB, PostGIS, and SSL enabled.

When you deploy the Railway template, the service runs the prebuilt image from GHCR rather than building this repository in Railway.

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.com/deploy/wZJzA-)

### Why though?

The [timescale/timescaledb-ha](https://hub.docker.com/r/timescale/timescaledb-ha) image in Docker Hub does not come with SSL baked in.

Since this could pose a problem for applications or services attempting to connect, we decided to roll our own image with SSL enabled.

### How does it work?

The current Railway template uses `Dockerfile.pg18-ts2.26`, which starts from `timescale/timescaledb-ha:pg18.3-ts2.26.4-oss`. The `init-ssl.sh` script is copied into `docker-entrypoint-initdb.d/` and runs during database initialization.

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

### A note about running the images locally

The images built from this repo and workflow do not contain support for ARM.  Check out the Dockerfiles that build them and feel free to copy the logic to build your own image with the appropriate base containing support for ARM.
