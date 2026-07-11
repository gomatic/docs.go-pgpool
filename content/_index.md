---
title: go-pgpool
---

`go-pgpool` is the gomatic ecosystem's **PostgreSQL connection-pool** library (package `pgpool`). It opens tuned [pgx v5](https://github.com/jackc/pgx) connection pools for PostgreSQL and CockroachDB and provides the matching health checker.

- **Source:** [`gomatic/go-pgpool`](https://github.com/gomatic/go-pgpool)
- **API reference:** [pkg.go.dev/github.com/gomatic/go-pgpool](https://pkg.go.dev/github.com/gomatic/go-pgpool)

## Install

```sh
go get github.com/gomatic/go-pgpool
```

## Connecting

`Connect` creates a `pgxpool.Pool` configured for CockroachDB or PostgreSQL:

```go
pool, err := pgpool.Connect(ctx, pgpool.ConnectionString(connString))
```

When the `ConnectionString` is **empty**, pgx builds the connection from the standard libpq environment variables — `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`, `PGSSLMODE`, `PGAPPNAME`, `PGCONNECT_TIMEOUT` — and honors `~/.pgpass` and `~/.pg_service.conf`, matching `psql`'s behavior. When set, it overrides the env vars and accepts both the URL form (`postgres://user:pass@host:port/db?sslmode=disable`) and the libpq KV form (`host=localhost port=5432 user=alice dbname=xto sslmode=disable`).

## Health checking

`HealthChecker` wraps a pool for liveness probes — it satisfies the common health-checker shape (`Name() string`, `Check(ctx) error`) and pings the pool on `Check`:

```go
hc := pgpool.HealthChecker{Pool: pool}
if err := hc.Check(ctx); err != nil {
	// errors.Is(err, pgpool.ErrDatabaseHealthCheck)
}
```

## Pairing with go-app

CLIs built on [`gomatic/go-app`](https://github.com/gomatic/go-app) bind the same libpq `PG*` surface as flags with its `flags/pg` package: embed `pg.Config` in the command config, list `pg.Binder{Config: &cfg}.Flags()` in the command's flags, and feed `Config.ConnString()` straight to `Connect`. Empty fields are omitted, so the fallback to the `PG*` environment variables (and `~/.pgpass` / `~/.pg_service.conf`) is preserved end to end:

```go
pool, err := pgpool.Connect(ctx, pgpool.ConnectionString(cfg.ConnString()))
```

## Errors

Every failure the package can emit is a sentinel — a constant of [`gomatic/go-error`](https://github.com/gomatic/go-error)'s `errs.Const` — matchable with [`errors.Is`](https://pkg.go.dev/errors#Is): `ErrCreateConnectionPool`, `ErrDatabaseHealthCheck`, `ErrParseDatabaseConfig`, `ErrPingDatabase`. See the [API reference](https://pkg.go.dev/github.com/gomatic/go-pgpool#pkg-constants).

## Provenance

Extracted from [`xto-email/go-config`](https://github.com/xto-email/go-config)'s database-connection surface (`ConnectDB`/`DBHealthChecker`, renamed here to `Connect`/`HealthChecker`). Only the pool code carried forward — that package's environment loading and secret rotation were retired, superseded by [`go-app`](https://github.com/gomatic/go-app) flag binding.
