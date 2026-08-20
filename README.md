# Firecrawl Skills

The full catalog of Firecrawl skills for AI coding agents, following the [Agent Skills](https://agentskills.io) format. Available as a plugin for Claude Code, Cursor, and OpenAI Codex.

## Install

```bash
npx skills add firecrawl/skills
```

Without `-y` this opens an interactive picker over the whole catalog. Install a specific skill by name (names are path-independent):

```bash
npx skills add firecrawl/skills --skill firecrawl-build
```

For CLI onboarding (installs the Firecrawl CLI plus the CLI skills, with optional workflow skills):

```bash
npx -y firecrawl-cli@latest init
```

## Catalog layout

| Category | Path | What it's for | Source of truth |
|---|---|---|---|
| CLI | [`skills/cli/`](./skills/cli) | Live web work during an agent session: search, scrape, crawl, interact from the terminal | **Mirror** of [`firecrawl/cli`](https://github.com/firecrawl/cli/tree/main/skills) — do not edit here |
| Build | [`skills/build/`](./skills/build) | Integrating Firecrawl APIs into product code: SDKs, REST, endpoint selection, API keys | **Mirror** of the [`firecrawl` monorepo `skills/`](https://github.com/firecrawl/firecrawl/tree/main/skills) — do not edit here |
| Workflows | [`skills/workflows/`](./skills/workflows) | End-to-end session recipes: lead gen, deep research, SEO audit, knowledge bases, and more | **Authored here** — PRs welcome |
| Reference | [`skills/reference/`](./skills/reference) | Query Firecrawl's research paper index and developer index | **Authored here** — PRs welcome |

## Contributing: where does my PR go?

> **CLI skills → PR [`firecrawl/cli`](https://github.com/firecrawl/cli). Build/SDK skills → PR the [`firecrawl`](https://github.com/firecrawl/firecrawl) monorepo (`skills/`). Everything else (workflows, reference) → PR this repo. New skills always land in an existing repo — when in doubt, this repo under `skills/workflows/`.**

Mirrored directories (`skills/cli/`, `skills/build/`) are overwritten by CI on every upstream sync; PRs against them will be redirected to the source repo.

## Skills

### CLI (`skills/cli/`, mirror)

Skills that teach agents to use the [Firecrawl CLI](https://github.com/firecrawl/cli) for live web work: `firecrawl`, `firecrawl-scrape`, `firecrawl-search`, `firecrawl-crawl`, `firecrawl-map`, `firecrawl-interact`, `firecrawl-agent`, `firecrawl-monitor`, `firecrawl-parse`, `firecrawl-download`.

### Build (`skills/build/`, mirror)

| Skill | Description |
|---|---|
| [`firecrawl-build`](./skills/build/firecrawl-build) | Firecrawl application API umbrella skill |
| [`firecrawl-build-onboarding`](./skills/build/firecrawl-build-onboarding) | Get `FIRECRAWL_API_KEY` into a project and choose the right SDK/docs |
| [`firecrawl-build-scrape`](./skills/build/firecrawl-build-scrape) | Integrate `/scrape` for single-page extraction |
| [`firecrawl-build-search`](./skills/build/firecrawl-build-search) | Integrate `/search` for discovery-first workflows |
| [`firecrawl-build-interact`](./skills/build/firecrawl-build-interact) | Integrate `/interact` for clicks, forms, and dynamic flows after scrape |

### Workflows (`skills/workflows/`, authored here)

End-to-end recipes such as [`firecrawl-lead-gen`](./skills/workflows/firecrawl-lead-gen), [`firecrawl-deep-research`](./skills/workflows/firecrawl-deep-research), [`firecrawl-seo-audit`](./skills/workflows/firecrawl-seo-audit), [`firecrawl-competitive-intel`](./skills/workflows/firecrawl-competitive-intel), and more — see [`skills/workflows/`](./skills/workflows) for the full set, and [`firecrawl-workflows`](./skills/workflows/firecrawl-workflows) for the umbrella skill and authoring guide.

### Reference (`skills/reference/`, authored here)

| Skill | Description |
|---|---|
| [`firecrawl-research-index`](./skills/reference/firecrawl-research-index) | Find papers in the research paper index — biomedical and life-science literature (PubMed, bioRxiv, medRxiv) plus arXiv preprints |
| [`firecrawl-developer-index`](./skills/reference/firecrawl-developer-index) | Answer developer questions from issues, pull requests, READMEs, and documentation pages |

## MCP Server

The plugin includes Firecrawl MCP configuration for the official [Firecrawl MCP server](https://github.com/firecrawl/firecrawl-mcp-server), so editors that support bundled MCP metadata can wire Firecrawl tools with `FIRECRAWL_API_KEY`.

## Plugins

This repo serves as a plugin for multiple platforms, bundling the CLI and build skills (workflows are session recipes, not editor-integration material):

- **Claude Code** - `.claude-plugin/`
- **Cursor** - `.cursor-plugin/`
- **OpenAI Codex** - `.codex-plugin/`

## Prerequisites

- A Firecrawl account or self-hosted Firecrawl deployment
- API key stored in `FIRECRAWL_API_KEY` for cloud usage

Get your API key at [firecrawl.dev/app](https://www.firecrawl.dev/app). If you do not have one yet, [`firecrawl-build-onboarding`](./skills/build/firecrawl-build-onboarding) includes the browser authorization flow.

## Which skills do I need?

- **"Search/scrape the web for me right now, during this session"** → CLI skills (`skills/cli/`)
- **"Add Firecrawl to this codebase"** → build skills (`skills/build/`)
- **"Run an end-to-end task like lead gen or deep research"** → workflow skills (`skills/workflows/`)
- **"Query published research papers or developer docs indexes"** → reference skills (`skills/reference/`)

Both usage paths follow Firecrawl's onboarding skill (same install, different use cases): [firecrawl.dev/agent-onboarding/SKILL.md](https://www.firecrawl.dev/agent-onboarding/SKILL.md).

## Docs (Source of Truth)

Read the source-of-truth page for your project language:

- **Node / TypeScript**: [docs.firecrawl.dev/agent-source-of-truth/node](https://docs.firecrawl.dev/agent-source-of-truth/node)
- **Python**: [docs.firecrawl.dev/agent-source-of-truth/python](https://docs.firecrawl.dev/agent-source-of-truth/python)
- **Rust**: [docs.firecrawl.dev/agent-source-of-truth/rust](https://docs.firecrawl.dev/agent-source-of-truth/rust)
- **Java**: [docs.firecrawl.dev/agent-source-of-truth/java](https://docs.firecrawl.dev/agent-source-of-truth/java)
- **Elixir**: [docs.firecrawl.dev/agent-source-of-truth/elixir](https://docs.firecrawl.dev/agent-source-of-truth/elixir)
- **cURL / REST**: [docs.firecrawl.dev/agent-source-of-truth/curl](https://docs.firecrawl.dev/agent-source-of-truth/curl)

## License

ISC
