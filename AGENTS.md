# Firecrawl Skills Catalog

Full catalog of Firecrawl agent skills, organized by category under `skills/`.

## Layout and edit rules

- `skills/cli/` — **mirror** of `firecrawl/cli` `skills/`. Do not edit here; CI overwrites on every sync. PR against `firecrawl/cli`.
- `skills/build/` — **mirror** of the `firecrawl` monorepo `skills/`. Do not edit here; CI overwrites on every sync. PR against `firecrawl/firecrawl`.
- `skills/workflows/` — **source of truth**, authored here. Safe to edit.
- `skills/reference/` — **source of truth**, authored here. Safe to edit.

Plugin metadata and top-level docs are safe to edit:

- `.cursor-plugin/`, `.claude-plugin/`, `.codex-plugin/`
- `README.md`, `AGENTS.md`, `.mcp.json`, `mcp.json`
- `.github/workflows/` (sync automation)

## Routing rule

CLI skills → PR `firecrawl/cli`. Build/SDK skills → PR `firecrawl` (monorepo, `skills/`). Everything else (workflows, reference) → PR this repo. New skills always land in an existing repo — when in doubt, this repo under `skills/workflows/`.

## Intent

Use the skills here when the task is:

- live web work during a session (CLI skills)
- adding Firecrawl to a codebase, choosing between `/scrape`, `/search`, and `/interact`, getting `FIRECRAWL_API_KEY` into `.env` (build skills)
- end-to-end recipes like lead gen, deep research, SEO audits (workflow skills)
- querying the research paper index or developer index, which `/search` does not query (reference skills)

## Authoring Rules (workflows + reference)

- Keep each `SKILL.md` concise and trigger-oriented.
- Lead with "use this when..." guidance.
- Favor endpoint names in slash notation: `/scrape`, `/search`, `/interact`.
- Keep CLI references short and defer to `firecrawl/cli` instead of duplicating command manuals.
- Treat [`https://www.firecrawl.dev/agent-onboarding/SKILL.md`](https://www.firecrawl.dev/agent-onboarding/SKILL.md) as the canonical source for the two-path framing.
- Skill layout: `skills/<category>/<skill-name>/SKILL.md`, deeper docs in `skills/<category>/<skill-name>/references/`, one level deep.
