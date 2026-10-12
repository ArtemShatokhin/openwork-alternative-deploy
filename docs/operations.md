Kortix is open source, and running a self-hosted instance means keeping a small set of files and images current.

# Operations

A self-hosted Kortix instance is one Docker Compose stack with no Redis and no separate worker. Operating it comes down to four habits: watch the health, read the logs, keep the images current, and copy the data before you touch it.

## Check the health of the stack

```bash
kortix self-host status     # what is running, and the current version
kortix self-host doctor     # validates the Docker tooling and the rendered config
docker compose ps           # per-service state and health
```

`doctor` is the first command to run when something looks wrong. Each application service carries a healthcheck, so `docker compose ps` shows which one is degraded before a user notices.

## Read the logs

```bash
docker compose logs -f kortix-api
docker compose logs -f kortix-frontend
docker compose logs -f kortix-gateway
```

The API logs the request path and the sandbox calls. A session that cannot start usually shows up here as a sandbox reachability error: the sandbox has no stable URL to call back to. Set a domain or a tunnel to fix it.

## Updates

Every instance updates itself automatically. The `kortix-updater` container checks for a new image once a day at a fixed local clock time (`KORTIX_UPDATE_TIME`, default 02:00, in `KORTIX_UPDATE_TZ`, default `America/New_York`). If an image changed, it runs the migration job, then starts new containers before it stops the old ones. On a box with two API replicas, that swap is zero-downtime; a single-replica box uses a brief-downtime swap instead.

Every instance tracks one of two moving tags, or a version you pin explicitly:

| Channel | Meaning |
|---|---|
| `stable` (default) | Curated. A person promotes a proven version on a schedule separate from production releases. |
| `latest` | Every production release retags `latest` automatically. |
| `--tag <version>` | Pins an exact version and overrides the channel. |

To pin a version and turn the automatic updater off:

```bash
kortix self-host update --tag <version> --auto-update off
```

To roll back after a bad release:

```bash
kortix self-host rollback --release <version>
```

Self-hosted instances only consume images the release pipeline has already built and signed. They never build or sign anything themselves.

## Backups

Kortix has no separate backup system. Each instance stores its data in two directories and one file under `~/.config/kortix/self-host/<instance>/`:

- `volumes/db/data`, the Postgres database
- `volumes/storage`, the Supabase Storage files
- the instance `.env`, which holds every secret and signing key the stack uses

Copy all three before any destructive command, and on a schedule you can live with. A restore needs all three, so a backup of only the database will not bring the instance back. The nightly trigger in this repo's manifest checks that fresh copies exist and records the result in a change request.

## Memory limits

Each API container defaults to a 640 MiB memory limit, which suits an 8 GiB host. On a 16 GiB host with API traffic pressing that limit, raise it:

```bash
kortix self-host env set KORTIX_API_MEMORY_LIMIT=1024m
```

Confirm the applied limit with `docker stats --no-stream`. A memory change set through the environment must survive a config regeneration, so set it with `kortix self-host env set` rather than editing the rendered Compose file by hand.

## Scaling and placement

Agent sessions never run on the box that holds the stack. They run on a separate sandbox provider the API reaches over egress, with Daytona as the default and Platinum and E2B also supported. A bigger control plane does not make sessions faster; it only serves more API traffic. Point the sandbox provider at the capacity you need, and keep the stack sized for its own traffic.

## Where to go next

- [Deployment](deploy.md) covers install, DNS, and the first session.
- [Configuration](configuration.md) explains the manifest, env, secrets, models, and connectors.
- The [campaign FAQ](https://openworkalternative.com/faq) answers the common questions about moving off OpenWork.
- [Kortix](https://kortix.com/docs/host) is the platform reference for every `kortix self-host` subcommand.
