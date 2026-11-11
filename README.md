<p align="center">
  <img src="assets/banner.png" alt="LinkedIn Ads Manager MCP — manage LinkedIn campaigns from Claude, ChatGPT, Gemini and Cursor" width="100%">
</p>

<h1 align="center">LinkedIn Ads Manager MCP</h1>

<p align="center">
  <b>Run your LinkedIn Ads account by chatting with the AI tool you already use.</b><br>
  Plan, launch, pause, audit and report on campaigns from Claude, ChatGPT, Gemini or Cursor —<br>
  with a preview-and-confirm safety gate in front of every action that could spend money.
</p>

<p align="center">
  <img alt="Version" src="https://img.shields.io/badge/version-1.0.0-0A66C2?style=flat-square">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-2EA44F?style=flat-square">
  <img alt="Node" src="https://img.shields.io/badge/node-%E2%89%A520-339933?style=flat-square&logo=node.js&logoColor=white">
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-5.9-3178C6?style=flat-square&logo=typescript&logoColor=white">
  <img alt="MCP" src="https://img.shields.io/badge/MCP-STDIO%20%7C%20Streamable%20HTTP-5AB0FF?style=flat-square">
  <img alt="Tools" src="https://img.shields.io/badge/tools-31-FFC857?style=flat-square">
  <img alt="Docker" src="https://img.shields.io/badge/Docker-ready-2496ED?style=flat-square&logo=docker&logoColor=white">
</p>

<p align="center">
  <a href="#-quick-start">Quick start</a> ·
  <a href="#-what-you-can-ask-it">Examples</a> ·
  <a href="#-tool-reference">Tools</a> ·
  <a href="#-safety-model">Safety</a> ·
  <a href="ARCHITECTURE.md">Architecture</a> ·
  <a href="#-roadmap">Roadmap</a> ·
  <a href="#-contributors">Contributors</a>
</p>

---

## 📌 Table of contents

1. [Why this exists](#-why-this-exists)
2. [Features at a glance](#-features-at-a-glance)
3. [How it works](#-how-it-works)
4. [What you can ask it](#-what-you-can-ask-it)
5. [Quick start](#-quick-start)
6. [Install options](#-install-options)
7. [Connect your AI client](#-connect-your-ai-client)
8. [LinkedIn setup and credentials](#-linkedin-setup-and-credentials)
9. [Configuration reference](#%EF%B8%8F-configuration-reference)
10. [Tool reference](#-tool-reference)
11. [Safety model](#-safety-model)
12. [Security](#-security)
13. [Project structure](#-project-structure)
14. [Development](#-development)
15. [Deployment](#-deployment)
16. [Troubleshooting](#-troubleshooting)
17. [Roadmap](#-roadmap)
18. [Contributing](#-contributing)
19. [Contributors](#-contributors)
20. [License](#-license)

---

## 💡 Why this exists

LinkedIn Campaign Manager is powerful but slow to click through: pulling a weekly report, cloning a campaign for a new city, or checking which ads are wasting budget can each take many screens.

This project turns those jobs into plain-language requests. It is a **Model Context Protocol (MCP) server** — a small service that gives AI assistants a safe, well-defined set of LinkedIn Ads actions. You ask; the assistant calls the right tool; you approve anything that touches spend.

| Without this | With this |
|---|---|
| Export CSVs, build pivot tables by hand | *"Show spend, CTR and leads per campaign for last 30 days"* |
| Copy a campaign field by field | *"Clone the Atlanta campaign as a draft for Dallas"* |
| Hunt for under-performing ads | *"Audit the account and list anything wasting budget"* |
| Worry about a mistyped budget | Every spend change shows a **preview** and needs **`confirm=true`** |

---

## ✨ Features at a glance

| | Feature | Details |
|---|---|---|
| 📊 | **Reporting** | Campaign and account analytics, spend, leads, conversions over any date range |
| 🚀 | **Campaign management** | Create (draft by default), clone, update, pause, resume |
| 🗂️ | **Campaign groups** | List, inspect, create, update |
| 🎨 | **Creatives** | List, inspect, create, update |
| 🎯 | **Targeting and budgets** | Read and change audience targeting and daily/total budgets |
| 🩺 | **Audits** | Account-wide and per-campaign health checks |
| 📝 | **Organization posts** *(optional)* | Publish and edit company-page posts when a posts token is set |
| 🛡️ | **Safety gate** | Structured preview + explicit confirmation for anything that spends, launches or posts |
| 🔐 | **Secret hygiene** | Tokens never returned by tools; logs and errors are redacted |
| 🌐 | **Two transports** | Local **STDIO** and remote **Streamable HTTP** (`/mcp`, `/health`) |
| 🐳 | **Docker** | Production image runs as a non-root user with a health check |
| 🧩 | **Claude plugin** | Backwards-compatible `/linkedin-setup` command and `linkedin-ads-manager` skill |

---

## 🧭 How it works

<p align="center">
  <img src="assets/architecture.png" alt="Architecture: AI clients connect over STDIO or Streamable HTTP to one MCP server, which applies a safety gate before calling LinkedIn's Marketing and Community Management APIs" width="100%">
</p>

```mermaid
flowchart LR
    A["Claude · ChatGPT · Gemini · Cursor"] -->|"STDIO or HTTPS /mcp"| B["MCP server<br/>31 tools"]
    B --> C{"Safety gate"}
    C -->|"read"| D["LinkedIn client"]
    C -->|"write / high-risk<br/>without confirm"| E["Preview returned<br/>nothing executed"]
    C -->|"confirm=true"| D
    D --> F["LinkedIn Marketing API"]
    D -.->|"optional"| G["Community Management API"]
```

There is **one** server implementation in [`mcp-server/`](mcp-server/). Everything under `clients/` and `examples/` is configuration only, so every AI client gets identical behaviour and the same safety rules. The full design is in **[ARCHITECTURE.md](ARCHITECTURE.md)**.

---

## 💬 What you can ask it

> **Reporting**
> - "What did we spend on LinkedIn this month, by campaign?"
> - "Which three campaigns had the lowest CTR in the last 14 days?"
> - "How many leads did the *Agentic Enterprise — Atlanta* campaign bring in?"

> **Changes (you will see a preview first)**
> - "Clone campaign 123456789 as a draft named *Dallas Roadshow*."
> - "Pause every campaign in group 987654 that spent over $500 with zero leads."
> - "Raise the daily budget on campaign 123456789 from $50 to $75."

> **Audits and posts**
> - "Run an account audit and give me the top five fixes."
> - "Post this announcement on our company page." *(needs the optional posts token)*

More prompts per client: [`examples/claude`](examples/claude/prompts.md) · [`examples/chatgpt`](examples/chatgpt/prompts.md) · [`examples/gemini`](examples/gemini/prompts.md) · [`examples/cursor`](examples/cursor/prompts.md)

---

## ⚡ Quick start

**You need:** Node.js 20 or newer, a LinkedIn ad account, and a LinkedIn access token with `rw_ads` and `r_ads_reporting` ([how to get one](#-linkedin-setup-and-credentials)).

```bash
# 1. Get the code
git clone https://github.com/nexoradev-stack/linkedin-ads-manager-ai-plugin.git
cd linkedin-ads-manager-ai-plugin

# 2. Add your credentials (never commit this file)
cp .env.example .env
#   then edit .env: LINKEDIN_CAMPAIGNS_TOKEN=...  LINKEDIN_ACCOUNT_ID=...

# 3. Install and build
npm install
npm run build

# 4. Check it starts
npm run start:stdio      # local mode for Claude Desktop / Cursor / Gemini CLI
# or
PORT=3000 npm start      # remote mode, then: curl http://localhost:3000/health
```

Then point your AI client at it — see [Connect your AI client](#-connect-your-ai-client).

---

## 📦 Install options

| Method | Best for | How |
|---|---|---|
| **A. Claude plugin** | Claude / Cowork users | Add this repo as a plugin marketplace, install `linkedin-ads-manager`, run `/linkedin-setup` |
| **B. Local MCP (source)** | Developers, Cursor, Gemini CLI | [Quick start](#-quick-start) above |
| **C. npm package** | One-line local install | `npx @eneelkant/linkedin-ads-mcp --stdio` — *package is prepared in `mcp-server/package.json` but not yet published* |
| **D. Docker** | Self-hosting the remote server | `docker compose up --build` |
| **E. Remote MCP** | ChatGPT and teams with no local setup | Deploy once, then give clients `https://YOUR-DOMAIN/mcp` |

> GitHub is the source and distribution home. It is **not** an authentication dependency: users of a hosted remote MCP never need a GitHub login.

---

## 🔌 Connect your AI client

| Client | Local STDIO | Remote HTTP | Guide |
|---|:---:|:---:|---|
| **Claude** (Desktop, Cowork, connectors) | ✅ | ✅ | [docs/claude.md](docs/claude.md) · [clients/claude](clients/claude/) |
| **ChatGPT** | — | ✅ *(public HTTPS required)* | [docs/chatgpt.md](docs/chatgpt.md) · [clients/chatgpt](clients/chatgpt/) |
| **Gemini / Gemini CLI** | ✅ | ✅ | [docs/gemini.md](docs/gemini.md) · [clients/gemini](clients/gemini/) |
| **Cursor** (Agent, CLI) | ✅ | ✅ | [docs/cursor.md](docs/cursor.md) · [clients/cursor](clients/cursor/mcp.json) |

<details>
<summary><b>Example: local STDIO config (Claude Desktop / Cursor / Gemini CLI)</b></summary>

```json
{
  "mcpServers": {
    "linkedin-ads": {
      "command": "node",
      "args": ["/absolute/path/to/linkedin-ads-manager-ai-plugin/mcp-server/dist/index.js", "--stdio"],
      "env": {
        "LINKEDIN_CAMPAIGNS_TOKEN": "your-token",
        "LINKEDIN_ACCOUNT_ID": "123456789"
      }
    }
  }
}
```
</details>

<details>
<summary><b>Example: remote HTTP config</b></summary>

```json
{
  "mcpServers": {
    "linkedin-ads": {
      "url": "https://YOUR-DOMAIN.example.com/mcp",
      "headers": { "Authorization": "Bearer YOUR_MCP_HTTP_BEARER_TOKEN" }
    }
  }
}
```
</details>

---

## 🔑 LinkedIn setup and credentials

LinkedIn needs **two separate developer apps** if you want both ads and company-page posting, because one app cannot hold both products.

| App | LinkedIn product | Scopes | Env variable | Required? |
|---|---|---|---|---|
| **App 1 — Campaigns** | Marketing Developer Platform | `rw_ads`, `r_ads_reporting` | `LINKEDIN_CAMPAIGNS_TOKEN` | ✅ Yes |
| **App 2 — Posts** | Community Management API | `w_organization_social` | `LINKEDIN_POSTS_TOKEN` | Optional |

**Steps for App 1**

1. Go to [linkedin.com/developers/apps](https://www.linkedin.com/developers/apps) and create an app.
2. Request the **Marketing Developer Platform** product and the `rw_ads` + `r_ads_reporting` scopes.
3. Under **Products → View Ad Accounts**, whitelist your ad account.
4. Generate a token under **Auth → OAuth 2.0 tools**.
5. Copy your numeric ad account ID from Campaign Manager into `LINKEDIN_ACCOUNT_ID`.

> ⏳ LinkedIn access tokens expire (typically after about 60 days). When tools start returning `401`, generate a new token and update `.env` or your deployment secret.

Full guide: [docs/authentication.md](docs/authentication.md)

---

## ⚙️ Configuration reference

| Variable | Required | Default | What it does |
|---|:---:|---|---|
| `LINKEDIN_CAMPAIGNS_TOKEN` | ✅ | — | Token for the Marketing API (App 1) |
| `LINKEDIN_ACCOUNT_ID` | ✅ | — | Numeric ad account ID |
| `LINKEDIN_POSTS_TOKEN` | | — | Enables organization-post tools (App 2) |
| `MCP_HTTP_BEARER_TOKEN` | | — | Requires `Authorization: Bearer …` on `/mcp`. **Set this for any public deployment.** |
| `LINKEDIN_TRUSTED_EXECUTION` | | `false` | Lets ordinary writes run without `confirm=true`. High-risk tools still always need it. |
| `MCP_TRANSPORT` | | auto | `stdio` or `http`. Auto picks `http` when `PORT` is set. |
| `PORT` | | `3000` | HTTP port |
| `ALLOWED_HOSTS` | | localhost | Extra `Host` header values accepted (comma-separated) |
| `MCP_RATE_LIMIT_MAX` | | `120` | Requests per window per client |
| `MCP_RATE_LIMIT_WINDOW_MS` | | `60000` | Rate-limit window |
| `LINKEDIN_API_VERSION` | | `202601` | Value sent in the `LinkedIn-Version` header |
| `LINKEDIN_HTTP_TIMEOUT_MS` | | `30000` | Timeout for each LinkedIn API call |
| `LOG_LEVEL` | | `info` | `debug`, `info`, `warn`, `error` |

---

## 🧰 Tool reference

31 tools, grouped by area. 🟢 read · 🟡 write (needs `confirm=true` unless trusted mode) · 🔴 high-risk (always preview, always `confirm=true`)

| Area | Tool | Risk |
|---|---|:---:|
| **Account** | `linkedin_get_ad_account` · `linkedin_list_ad_accounts` | 🟢 |
| **Campaigns** | `linkedin_list_campaigns` · `linkedin_get_campaign` | 🟢 |
| | `linkedin_update_campaign` | 🟡 |
| | `linkedin_create_campaign` · `linkedin_clone_campaign` · `linkedin_pause_campaign` · `linkedin_resume_campaign` | 🔴 |
| **Campaign groups** | `linkedin_list_campaign_groups` · `linkedin_get_campaign_group` | 🟢 |
| | `linkedin_create_campaign_group` · `linkedin_update_campaign_group` | 🟡 |
| **Creatives** | `linkedin_list_creatives` · `linkedin_get_creative` | 🟢 |
| | `linkedin_create_creative` · `linkedin_update_creative` | 🟡 |
| **Targeting & budget** | `linkedin_get_campaign_targeting` · `linkedin_get_campaign_budget` | 🟢 |
| | `linkedin_update_campaign_targeting` · `linkedin_update_campaign_budget` | 🔴 |
| **Reporting** | `linkedin_get_campaign_analytics` · `linkedin_get_account_analytics` · `linkedin_get_spend` · `linkedin_get_leads` · `linkedin_get_conversions` | 🟢 |
| **Audits** | `linkedin_audit_account` · `linkedin_audit_campaign` | 🟢 |
| **Organization** *(optional)* | `linkedin_get_organization` | 🟢 |
| | `linkedin_update_organization_post` | 🟡 |
| | `linkedin_create_organization_post` | 🔴 |

There is deliberately **no** generic "call any LinkedIn endpoint" tool. Every action is narrow, validated and reviewable.

---

## 🛡️ Safety model

```mermaid
sequenceDiagram
    participant You
    participant AI as AI client
    participant MCP as MCP server
    participant LI as LinkedIn
    You->>AI: "Raise campaign 123's budget to $75/day"
    AI->>MCP: linkedin_update_campaign_budget(…)
    MCP->>LI: read current campaign
    MCP-->>AI: Preview (from $50 → to $75, financial impact)<br/>nothing changed yet
    AI-->>You: Shows the preview and asks to confirm
    You->>AI: "Yes, go ahead"
    AI->>MCP: same call + confirm=true
    MCP->>LI: apply change
    MCP-->>AI: ok: true, result
```

| Level | Examples | Rule |
|---|---|---|
| 🟢 **Read** | list, get, analytics, audits | Runs immediately |
| 🟡 **Write** | update campaign name, create group or creative | Needs `confirm=true`, unless `LINKEDIN_TRUSTED_EXECUTION=true` |
| 🔴 **High-risk** | create, clone, pause, resume, budget, targeting, org post | **Always** returns a preview first and **always** needs `confirm=true` |

Each preview includes the current state, the requested state, any budget or targeting change, the expected impact, the financial implications and whether the action is reversible. **The server never silently raises a budget or launches a campaign.** New campaigns default to `DRAFT`.

---

## 🔐 Security

- Tokens live only in environment variables, `.env` or your host's secret store — **never** in tool output.
- Bearer tokens and secret-looking values are redacted from logs and error messages.
- Remote `/mcp` supports a bearer token, a `Host` allowlist and per-client rate limiting.
- The Docker image runs as a non-root user.
- `npm run security:scan` checks the repo for committed secrets.

Security design notes: [docs/security.md](docs/security.md). If you find a vulnerability, please contact the maintainer privately instead of opening a public issue.

---

## 🗂️ Project structure

```text
linkedin-ads-manager-ai-plugin/
├── mcp-server/                 # ⭐ the one canonical MCP server (TypeScript)
│   ├── src/
│   │   ├── index.ts            # CLI entry: --stdio | --http | --help
│   │   ├── server.ts           # creates the MCP server, registers tools
│   │   ├── tools/              # 31 tool definitions + shared helpers
│   │   ├── safety/             # confirmation gate + action previews
│   │   ├── linkedin/           # REST client, types, typed errors
│   │   ├── auth/               # credential loading, status
│   │   ├── transport/          # stdio.ts, http.ts (/mcp, /health)
│   │   └── utils/              # logger, rate limiter, redaction
│   └── tests/                  # vitest: protocol, HTTP, client, safety, redaction
├── linkedin-ads-manager/       # Claude plugin: /linkedin-setup + skill + Python CLI
├── clients/                    # per-client config files only (no logic)
├── examples/                   # prompt libraries per client
├── docs/                       # setup, auth, deployment, security, troubleshooting
├── assets/                     # banner and architecture images (SVG source + PNG)
├── scripts/security-scan.mjs   # secret scanner
├── Dockerfile · docker-compose.yml
├── ARCHITECTURE.md · CHANGELOG.md · LICENSE
└── .env.example                # placeholders only
```

---

## 🛠️ Development

```bash
npm install
npm run lint           # ESLint
npm run typecheck      # tsc --noEmit
npm test               # vitest unit tests
npm run test:mcp       # MCP protocol + HTTP transport tests
npm run smoke          # end-to-end MCP smoke check
npm run build          # compile to mcp-server/dist
npm run security:scan  # look for committed secrets
```

**Adding a tool:** register it in `mcp-server/src/tools/register.ts` with a zod input schema; if it writes, add it to `WRITE_TOOLS` or `HIGH_RISK_TOOLS` in `safety/confirmation.ts`, route it through `gateWrite`, and add tests.

---

## 🚀 Deployment

```bash
docker compose up --build
curl http://localhost:3000/health
```

`/health` reports whether credentials are configured (never their values). Put the server behind HTTPS, set `MCP_HTTP_BEARER_TOKEN` and `ALLOWED_HOSTS`, and store tokens in your platform's secret manager. Platform notes: [docs/deployment.md](docs/deployment.md) · [docker/README.md](docker/README.md)

---

## 🧯 Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `401` from LinkedIn | Token expired or wrong app | Generate a new token in App 1 |
| `403` from LinkedIn | Ad account not whitelisted, or scope missing | Add the account under *View Ad Accounts*; check `rw_ads` / `r_ads_reporting` |
| Org-post tools missing | `LINKEDIN_POSTS_TOKEN` not set | Add the App 2 token |
| `401 Missing bearer token` on `/mcp` | `MCP_HTTP_BEARER_TOKEN` is set | Send `Authorization: Bearer …` from the client |
| `429 Rate limit exceeded` | Too many requests from one client | Wait `retryAfterMs`, or raise `MCP_RATE_LIMIT_MAX` |
| Host rejected | Custom domain not allowed | Add it to `ALLOWED_HOSTS` |

More: [docs/troubleshooting.md](docs/troubleshooting.md)

---

## 🗺️ Roadmap

Planned work is tracked as [GitHub issues](https://github.com/nexoradev-stack/linkedin-ads-manager-ai-plugin/issues), grouped into milestones:

- **v1.1 — Hardening:** automatic retries, pagination, constant-time auth, CI restored
- **v1.2 — Reporting+:** daily breakdowns, demographic pivots, CSV export, budget pacing alerts
- **v1.3 — Campaign ops:** matched audiences, lead-gen forms, conversion tracking, bulk actions
- **v2.0 — Team scale:** multi-account, OAuth refresh, audit log of writes, shared rate limits

Ideas welcome — open an issue with the `enhancement` label.

---

## 🤝 Contributing

1. Fork, then branch from `main` (`feat/…`, `fix/…`, `docs/…`).
2. Keep **all LinkedIn API logic in `mcp-server/`** — client folders stay config-only.
3. Keep the Claude plugin backwards compatible (`/linkedin-setup` and the skill must keep working).
4. Any new write tool must go through the safety gate and come with tests.
5. Run `npm run lint && npm run typecheck && npm test && npm run security:scan` before opening a PR.
6. **Never commit secrets.** Use `.env.example` for placeholders.

Good starting points carry the [`good first issue`](https://github.com/nexoradev-stack/linkedin-ads-manager-ai-plugin/labels/good%20first%20issue) label.

---

## 👥 Contributors

| | Contributor | Role |
|:---:|---|---|
| 🧑‍💻 | **Deepanjan Dasgupta** ([@nexoradev-stack](https://github.com/nexoradev-stack)) | Maintainer — product direction, LinkedIn Ads workflows, design |
| 🤖 | **Claude** (Anthropic) | AI pair programmer — documentation, architecture, issue planning |
| ⌨️ | **Cursor** | AI code editor — implementation and refactoring |
| 🌱 | **Neelkant Ekbote** ([@eneelkant](https://github.com/eneelkant)) | Original author of the plugin and MCP server |

---

## 📄 License

[MIT](LICENSE) — free to use, modify and distribute.

<p align="center"><sub>Not affiliated with or endorsed by LinkedIn Corporation. "LinkedIn" is a trademark of LinkedIn Corporation.</sub></p>
