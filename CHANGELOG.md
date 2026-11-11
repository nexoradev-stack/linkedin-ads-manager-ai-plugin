# Changelog

## Unreleased — 2026-11-11

### Documentation

- README rewritten: banner, badges, feature overview, example prompts, quick start, client connection examples, configuration reference, tool table with risk levels, safety sequence diagram, project structure, troubleshooting table, roadmap and contributors
- New root `ARCHITECTURE.md` covering components, request lifecycle, safety model, credentials, transports, error model, testing, deployment, design decisions and known limitations; `docs/architecture.md` now points to it
- New images in `assets/`: `banner.svg`/`banner.png` and `architecture.svg`/`architecture.png`
- Clone URLs now point at `nexoradev-stack/linkedin-ads-manager-ai-plugin`
- Roadmap tracked as GitHub issues and milestones

### Notes

- The GitHub Actions workflow listed under 1.0.0 is not currently in the repository (it was removed until the pushing token has `workflow` scope). Restoring it is tracked as an issue.

## 1.0.0 — 2026-09-29

### Added

- Canonical TypeScript MCP server (`mcp-server` / `@eneelkant/linkedin-ads-mcp`)
- STDIO transport for local Claude / Cursor / Gemini usage
- Streamable HTTP transport with `/mcp` and `/health`
- Narrowly scoped LinkedIn Ads MCP tools (accounts, campaigns, groups, creatives, targeting, budgets, reporting, audits)
- Optional organization posting tools gated on `LINKEDIN_POSTS_TOKEN`
- Safety layer with structured previews and mandatory confirmation for high-risk spend/launch/post actions
- Token redaction for logs and tool errors
- Client configuration packages for Claude, ChatGPT, Gemini, and Cursor
- Docker + docker-compose deployment support
- GitHub Actions CI (lint, typecheck, tests, MCP smoke, security scan, build)
- Cross-platform product documentation and architecture docs

### Changed

- README rewritten for cross-platform MCP distribution
- Semantic version bumped from Claude plugin `0.2.4` to product `1.0.0` for the architecture change

### Compatibility

- Claude plugin workflow preserved: `/linkedin-setup`, `linkedin-ads-manager` skill, Python CLI scripts, `.env` credential behavior
- ChatGPT remote MCP compatibility via Streamable HTTP
- Gemini CLI STDIO + HTTP configuration examples
- Cursor project/global MCP configuration examples

### Security

- `.gitignore` hardened for `.env` / `.env.*` with `.env.example` exception
- No secrets committed; CI secret scanning added
- Remote optional bearer authentication and rate limiting
