# Firecrawl Skills Catalog

Read-only distribution catalog for all Firecrawl agent skills. **Every skill directory here is a CI-synced mirror — do not edit skills in this repo.**

## Layout and sources

- `skills/cli/` — mirror of `firecrawl/cli` `skills/` (CLI skills + the research/developer index skills)
- `skills/build/` — mirror of the `firecrawl` monorepo `skills/`
- `skills/workflows/` — mirror of `firecrawl/firecrawl-workflows` `skills/`

Only repo metadata is authored here: `.cursor-plugin/`, `.claude-plugin/`, `.codex-plugin/`, `README.md`, `AGENTS.md`, `.mcp.json`, `mcp.json`, `.github/`.

## Routing rule

CLI skills (including the research/developer index skills) → PR `firecrawl/cli`. Build/SDK skills → PR `firecrawl` (monorepo, `skills/`). Workflow skills → PR `firecrawl/firecrawl-workflows`. Never PR skill content against this catalog — CI overwrites it on the next sync.

## Intent

Use the skills here when the task is:

- live web work during a session, or querying the research paper / developer indexes (CLI skills)
- adding Firecrawl to a codebase, choosing between `/scrape`, `/search`, and `/interact`, getting `FIRECRAWL_API_KEY` into `.env` (build skills)
- end-to-end recipes like lead gen, deep research, SEO audits (workflow skills)
