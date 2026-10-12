Kortix is open source, and every part of its configuration is a file you can read and edit.

# Configuration

A Kortix project has two configuration surfaces, and they do not overlap. The platform reads `kortix.yaml` at the repo root: it drives the trigger sweep, the sandbox builder, session token minting, and the dashboard. The agent harness reads the agent files, the skills, and the harness files. The join between the two is the agent's own `.md` file.

| Path | What it holds |
|---|---|
| `kortix.yaml` | The manifest: agents and what each may access, triggers, connectors, env, sandbox templates. |
| `agents/<name>.md` | One agent: frontmatter and system prompt. |
| `skills/<name>/SKILL.md` | One skill. Every harness loads it. |
| `memory/` | The project memory, as files that accumulate. |

## The v2 manifest, key by key

The manifest in this repo is `kortix.yaml`. Point your editor at the schema for live validation, then read the keys below.

```yaml
# yaml-language-server: $schema=https://kortix.com/schema/kortix.v2.schema.json
kortix_version: 2
default_agent: deploy-ops

project:
  name: openwork-alternative-deploy
  description: Self-host deployment template for the open-source AI Operating System.

env:
  required: []
  optional:
    - KORTIX_SANDBOX_PROVIDER_KEY
    - GITHUB_AGENT_TOKEN
```

`kortix_version: 2` is required and must be the number 2. `default_agent` is required and must name an agent that the manifest declares. A file that declares a version higher than 2 is rejected, so the platform never silently misreads a future field. Unknown top-level keys are ignored, so you can park your own metadata in the file.

`project.name` and `project.description` are display metadata. `env.required` and `env.optional` list environment variable names, and `required` is advisory: it is never enforced at session start.

Two rules catch most mistakes. Behavioral fields are a hard error in this file: the system prompt, the model, the mode, temperature, and permission live only in the agent's `.md`. And schema version 2 has no place for a channel declaration: a connected channel is a connector with its own provider, so leave channels out of the manifest.

## The agent block governs, the agent file behaves

The `agents:` block sets governance only: which agents may launch and what each may touch. The behavior lives in `agents/<name>.md`, named by `agents.<name>.file`.

```yaml
agents:
  deploy-ops:
    file: agents/deploy-ops.md
    connectors: all
    secrets:
      - KORTIX_SANDBOX_PROVIDER_KEY
      - GITHUB_AGENT_TOKEN
    kortix_permissions: all
    skills: all
```

Create `agents/deploy-ops.md` beside the manifest. Its frontmatter and body hold the behavior:

```markdown
---
description: Deploy-ops agent: install, update, back up, and doctor the stack.
mode: primary
---
You are the deploy-ops agent for a self-hosted Kortix instance.

Run `kortix self-host status` and `kortix self-host doctor`. Confirm the
Postgres, storage, and .env backups were written in the last 24 hours. Record
the running version. Write a short pass/fail line per check to
ops/self-host-health.md, then open a change request against main.
```

Grants named in the manifest are not frontmatter keys: the agent file holds behavior only, so leave grant and tool keys out of it.

## Triggers

A trigger starts a session with no person present, either on a cron schedule or on a signed webhook. The manifest in this repo schedules a nightly doctor run.

```yaml
triggers:
  - slug: nightly-self-host-doctor
    name: Nightly self-host doctor
    type: cron
    agent: deploy-ops
    cron: "0 3 * * *"
    timezone: UTC
    prompt: |
      Check the self-hosted Kortix stack and open a change request with the result.
```

The prompt opens a change request rather than merging. Merge stays default-deny for agents, so the nightly run reports and waits for a person.

## Environment and secrets

No secret value belongs in the repo. `kortix.yaml` lists secret names and grants; the values live in the instance `.env` or the dashboard. Secrets are encrypted at rest with a key per project, and connector credentials are brokered server-side and never enter the machine.

The Compose template reads these values from the `.env` beside `docker-compose.yml`:

| Variable | Used by | What to set |
|---|---|---|
| `POSTGRES_PASSWORD` | Postgres, API, migrate | A strong password for the local database. |
| `KORTIX_IMAGE_TAG` | All Kortix services | The image tag to run. Defaults to `stable`. |
| `PUBLIC_URL` | API, frontend | The public URL of the app. |
| `API_PORT` | API | Host port for the API. Defaults to `8008`. |
| `FRONTEND_PORT` | Frontend | Host port for the frontend. Defaults to `3000`. |
| `GATEWAY_INTERNAL_TOKEN` | API, gateway | A shared token the API and gateway use to talk. |
| `KORTIX_SANDBOX_PROVIDER_KEY` | API | The sandbox provider credential. |

The canonical stack renders an `.env` with many more keys, including the Supabase signing keys. Do not hand-copy those; let `kortix self-host init` write them.

## Models

Kortix is model-agnostic. It runs any model provider with your own keys, or the ChatGPT plan you already pay for. On the managed cloud you can also sign in with your OpenCode Console account for OpenCode Zen and Go. On a self-hosted instance you connect your own key in the model picker once the stack is up, and your provider bills the tokens directly.

Name model families rather than version numbers in anything you write into the repo, so a config note does not go stale within the month.

## Connectors and policy

A connector is an app an agent can reach. Connector credentials are brokered server-side, and you scope each connector action to Allow, Ask, or Block. An Ask holds the call until a person approves it, and the agent's turn pauses and resumes when the decision lands.

Approval gates are off until you set them. The project-wide default is `allow_all`, so set `policy:` explicitly when you want an action to require approval:

```yaml
policy:
  default_mode: risk
```

Use the nested form in YAML. The dotted `policy.default_mode` is a docs path, not a key.

## The machine image

By default, a session boots the platform's sandbox image. To pin your own, add a `sandbox.templates` block and point an agent at it with `sandbox: <slug>`. The template takes either an `image` or a `dockerfile`, then its hardware:

```yaml
sandbox:
  templates:
    - slug: ops
      name: Ops
      dockerfile: Dockerfile.ops
      cpu: 2
      memory: 4
agents:
  deploy-ops:
    sandbox: ops
```

A template names exactly one of `image` or `dockerfile`, never both. A Dockerfile path is relative to the repo root and cannot escape it.

## Where to go next

- [Deployment](deploy.md) covers install, DNS, and the first session.
- [Operations](operations.md) covers updates, backups, and rollback.
- The [OpenWork vs Kortix comparison](https://openworkalternative.com/openwork-vs-kortix) shows how the two keep configuration in different places, one desktop app against one owned repo.
- [Read the docs](https://kortix.com/docs) for the full manifest reference and every CLI subcommand.
