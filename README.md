Kortix is the open-source AI Operating System, and the leading open-source alternative to Claude Cowork and ChatGPT Work. This repository is the deployment template for a team that wants an OpenWork-style AI workspace under its own control: a multi-service `docker-compose.yml`, a Kortix `kortix.yaml` v2 manifest, and three docs to take it live on a box you own.

# Deploy an open-source OpenWork alternative: self-host Kortix

OpenWork is a free, open-source desktop app for macOS, Windows, and Linux where AI agents work on your own local files. Kortix is the open-source system a company runs: your agents, their skills, your company memory, and every connector live in one git repo you own, and each session runs on its own cloud computer. Both run agents with your own model keys. The difference is the unit you own, one desktop or one repo, and this repo is the deployment half of the second one.

## What Kortix is

Kortix is the open-source AI Operating System: your agents, their skills, your company memory, and every connector in one git repo you own, with the agents working on real cloud computers. A session is one unit of agent work. It starts on its own isolated sandbox and branch, the agent works there, and its finished work reaches `main` only through a change request a person reviews as a diff. Merge is default-deny for agents.

You run any model provider with your own keys, or the ChatGPT plan you already pay for. You run the platform on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free, and the code is open source; read it, fork it, audit it.

## Kortix and OpenWork side by side

Both projects are open source and both let you bring your own model keys. They answer different questions. The table below covers where each keeps its work, where its agents run, and how work lands.

| | Kortix | OpenWork |
|---|---|---|
| What it is | Open-source AI Operating System. One git repo holds agents, skills, memory, connectors. | A free desktop app where agents work on your local files. |
| Open source | Yes. Open source. Read it, fork it, audit it. | App and core open source; the Den control plane is commercial. |
| Where agents run | Each session runs on its own isolated sandbox and branch. | Locally, in a desktop app for macOS, Windows, and Linux. |
| Models | Any model provider with your own keys. | Any model OpenCode supports: 50+ providers or local models. |
| How work lands | Session work reaches `main` through a change request. | Work happens directly on your local files. |
| Hosting | Self-host free on your VPC or on-prem. | Desktop runs locally; the Den control plane can be self-hosted. |
| Governance | Per-resource permissions for people and agents. | An admin interface publishes capabilities and manages access. |

OpenWork facts come from OpenWork's own site and repository; Kortix facts come from the Kortix docs and repository. Checked October 2026.

OpenWork's cloud is free for the first five seats, then $10 per seat per month as published on its pricing page, and its Team and Enterprise tiers carry SSO, desktop policies, and a managed inference add-on (checked October 2026). Kortix self-hosts for free with no per-seat price, or runs on managed cloud where Free is $0 with 500 credits a month and one project, the paid plan is Credits at 100 credits per dollar with 5,000 credits a month costing $50, and members are unlimited and free, as written on the Kortix pricing page (checked October 2026).

For the full decision, see the [OpenWork vs Kortix comparison](https://openworkalternative.com/openwork-vs-kortix) on the campaign site.

## What is inside

```
.
├── docker-compose.yml    multi-service stack: Postgres, API, gateway, frontend
├── kortix.yaml           v2 manifest: deploy-ops agent, secret grants, cron trigger
└── docs/
    ├── deploy.md         concrete deployment steps
    ├── configuration.md  the manifest, env, secrets, models and connectors
    └── operations.md     updates, backups, logs and rollback
```

`kortix.yaml` declares the deploy-ops agent, the secrets it may read, and a nightly cron trigger that doctors the stack and opens a change request with the result. Secret values never live in the repo: the manifest lists secret names only, and you set the values through the CLI or the dashboard.

## Two ways to bring it up

The supported production path is the Kortix CLI, which renders the complete stack (the frontend, the API, the LLM gateway, and the Supabase distribution, plus Caddy for TLS) and starts it.

```bash
curl -fsSL https://kortix.com/install | bash   # install the Kortix CLI
kortix login                                   # sign in, pick an account
kortix self-host init --domain kortix.example.com
kortix self-host start
kortix self-host status
```

Create the A/AAAA records for your domain and for `api.<domain>` before `init`, and open ports 80 and 443. For a trial with no domain, `kortix self-host init --tunnel cloudflare` gives an ephemeral URL. The step-by-step path is in [docs/deploy.md](docs/deploy.md).

The `docker-compose.yml` in this repo is the readable, multi-service template of that same stack. It brings up the application tier on a machine you configure by hand, so you can read and edit every service before you point a domain at it.

Create a `.env` beside `docker-compose.yml` with the values listed in [docs/configuration.md](docs/configuration.md), then start the stack:

```bash
docker compose up -d
docker compose ps
```

## Add the agent to an existing project

Copy `kortix.yaml` into a Kortix project, commit it, and review the change request. New agents and triggers reach future sessions once the change request merges to the default branch. Run a session against the agent with:

```bash
kortix sessions new --agent deploy-ops --prompt "Doctor the self-hosted stack and open a CR"
```

## Docs

- [Deployment](docs/deploy.md) covers install, DNS, the Compose stack, and the first session.
- [Configuration](docs/configuration.md) explains the v2 manifest, env, secrets, models, and connectors.
- [Operations](docs/operations.md) covers updates, backups, logs, memory limits, and rollback.

For the platform reference, [Read the docs](https://kortix.com/docs). For the code, see [Kortix on GitHub](https://github.com/kortix-ai/suna).

## Get started

Self-host the open-source alternative on your own box, or run it on managed cloud. The [self-hosting guide](https://openworkalternative.com/self-hosting) walks through the install, and [Kortix](https://kortix.com) is the platform it runs.
