Kortix is open source, and you can deploy the whole platform on a machine you own.

# Deploy Kortix on your own box

A self-hosted Kortix instance is one Docker Compose stack. The frontend, the kortix-api image, the LLM gateway, and the Supabase distribution run on the box you pick, and agent sessions run off the box on a separate sandbox provider. The same Compose file runs on a laptop, a VPS, or a server in your own cloud account. A domain is only the value of `KORTIX_DOMAIN`, not a different setup.

Kortix is the open-source AI Operating System. Your agents, their skills, your company memory, and every connector live in one git repo you own, and each session runs on its own cloud computer. This guide takes the stack from a bare Linux box to a first session.

## What you need before you start

- A Linux host with Docker and the Docker Compose plugin. The one-shot bootstrap installs Docker for you; the manual path expects it already there.
- An 8 GiB host at the default limits. Each API service defaults to a 640 MiB memory limit, which suits an 8 GiB box. On a 16 GiB host with API traffic pressing that limit, raise it (see [Operations](operations.md)).
- A domain with an A/AAAA record for the domain and for `api.<domain>`, both pointing at the box, with ports 80 and 443 open. The bundled Caddy proxy uses them to issue a TLS certificate.
- A sandbox provider key. This is the one credential the stack cannot start without, because session compute runs on the provider rather than on the box.
- Outbound network access. The stack is not an offline install: it pulls images from docker.io/kortix/*, and the API reaches the sandbox provider over egress.

If you have no domain yet, the evaluation path uses a Cloudflare tunnel instead. The tunnel URL changes on every restart, so run production on a real domain.

## Path A: one command on a bare Linux box

The bootstrap installs Docker, installs the Kortix CLI, and starts the whole stack. Create the DNS records first and open ports 80 and 443.

```bash
curl -fsSL https://kortix.com/install | bash   # install the Kortix CLI
kortix login                                   # sign in, pick an account
kortix self-host init --domain kortix.example.com
kortix self-host start
kortix self-host status
```

`init` renders the Compose config, the `.env`, and, when you set a domain, a `Caddyfile` and an updater script into `~/.config/kortix/self-host/<instance>/`. `start` runs `docker compose up`.

## Path B: the CLI on any host you configure yourself

On macOS, or on a host you would rather configure by hand, install the CLI and run the manual path:

```bash
curl -fsSL https://kortix.com/install | bash
kortix self-host init --domain kortix.example.com
kortix self-host start
kortix self-host doctor
```

`doctor` validates the Docker tooling and the rendered config. To evaluate without a domain, run `kortix self-host init --tunnel cloudflare`.

## Path C: the Compose template in this repo

The `docker-compose.yml` here is the readable, multi-service application tier of the same stack. It brings up Postgres, the API, the LLM gateway, and the frontend so you can read and edit every service before you point a domain at it. The complete production stack adds the Supabase distribution and Caddy, which the CLI renders.

Create a `.env` beside `docker-compose.yml` with these values, then start the stack:

```bash
docker compose up -d
docker compose ps
```

The values each service reads are listed in [Configuration](configuration.md).

## After the stack is up

1. Set the sandbox provider key. Run `kortix self-host configure` for the interactive prompt, or set `KORTIX_SANDBOX_PROVIDER_KEY` in the instance `.env` and restart. Platinum is the default provider; Daytona remains available as a legacy fallback.
2. Open the app and create your account. Sign up in the dashboard on the domain you set.
3. Connect a model key. Models are BYOK in the app: open the model picker and add your own provider key. A self-hosted instance uses your own key by default.
4. Connect GitHub. In the dashboard, go to **Settings → Git** and authorize the repository the agents will work in.
5. Commit the manifest. Copy `kortix.yaml` and your `agents/deploy-ops.md` into the project, commit, and review the change request.

## Start your first session

```bash
kortix sessions new --agent deploy-ops --prompt "Doctor the self-hosted stack and open a CR"
```

The session runs on its own isolated sandbox and branch. When the agent is done, its work reaches `main` only through a change request a person reviews as a diff.

## When something does not start

- Sessions cannot run: the sandbox has no stable URL to call back to. Set a domain or a tunnel; a loopback URL is not reachable from a cloud sandbox.
- The API 500s on every `/v1/*` path: confirm the API container is healthy with `docker compose ps` and that the frontend points at the in-network service (`KORTIX_API_PROXY_TARGET`).
- The stack is slow to become ready: the first pull downloads the application images and the Supabase distribution. Watch `docker compose logs -f kortix-api`.

## Where to go next

- [Configuration](configuration.md) explains the v2 manifest, env, secrets, models, and connectors.
- [Operations](operations.md) covers updates, backups, logs, memory limits, and rollback.
- The [self-hosting guide](https://openworkalternative.com/self-hosting) covers the same install in the context of choosing an alternative to OpenWork.
- [Kortix](https://kortix.com/docs/host) is the platform reference for the self-host commands.
