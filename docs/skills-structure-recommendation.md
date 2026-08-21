# Skills Structure: Recommendation & Migration Plan

Response to the August 2026 skills-repos handoff. Additive-only per the rule:
every install command that works today keeps working.

## Recommendation in one paragraph

Make `firecrawl/skills` the **full catalog** — the one repo users install from
and the one path we promote everywhere. Keep each skill's **source of truth
co-located with the code it teaches**: CLI skills stay in `firecrawl/cli`,
build skills move to the `firecrawl` monorepo next to the SDKs, and workflow +
reference skills are authored directly in the catalog. CI (never humans) syncs
copies from each source into the catalog, and from the catalog out to the
legacy `firecrawl-workflows` mirror. No install path moves or breaks; changes
are additive.

## Why this works: install paths and authoring are decoupled

skills.sh keys install counts to `owner/repo/skill-name` — the repo users
install from, not where the skill is authored, and not the directory path
inside the repo. Two consequences:

1. Moving *authorship* of the build skills into the monorepo costs zero
   installs, because their install path (`firecrawl/skills`) never changes.
2. Reorganizing the catalog into category subdirectories does not reset any
   counter, because the key is the skill name, not the path.

We only start new counters where we change the *promoted* install path: CLI
skills and workflows get new catalog counters starting at zero, while their
old paths keep working (and keep counting) for anyone still using them.

| Family | Source of truth (humans edit) | Install path (promoted) | Legacy path | Count effect |
|---|---|---|---|---|
| CLI (10) | `firecrawl/cli` `skills/` | catalog `skills/cli/` | `firecrawl/cli` keeps working | new catalog counters; old 725K freezes as legacy installs taper |
| Build (5) | `firecrawl` monorepo `skills/` (moved) | catalog `skills/build/` | n/a — same repo as today | **none** — 256K keeps accruing uninterrupted |
| Workflows (16) | catalog `skills/workflows/` (moved) | catalog `skills/workflows/` | `firecrawl-workflows` mirror keeps working | new catalog counters; old 498K keeps counting for legacy installers |
| Reference/index (2) | catalog `skills/reference/` | catalog `skills/reference/` | n/a — same repo as today | none |

## Target catalog layout

```
firecrawl/skills
├── skills/
│   ├── cli/           # MIRROR — synced from firecrawl/cli, do not edit here
│   │   └── firecrawl, firecrawl-scrape, … (10)
│   ├── build/         # MIRROR — synced from firecrawl monorepo, do not edit here
│   │   └── firecrawl-build, firecrawl-build-scrape, … (5)
│   ├── workflows/     # SOURCE — PRs welcome here
│   │   └── firecrawl-lead-gen, firecrawl-deep-research, … (16)
│   └── reference/     # SOURCE — PRs welcome here
│       └── firecrawl-developer-index, firecrawl-research-index
├── .claude-plugin/  .cursor-plugin/  .codex-plugin/   # consolidated plugins
├── .mcp.json
└── README.md          # routing rule + mirror notice per category
```

The `npx skills` CLI walks skill containers up to three levels deep, so
`skills/<category>/<name>/SKILL.md` is discovered natively. `--skill <name>`
and `owner/repo@skill` are name-based, so
`npx skills add firecrawl/skills --skill firecrawl-build` works identically
before and after categorization.

Bare `npx skills add firecrawl/skills` changes meaning from "5 build skills"
to "pick from the full catalog" — accepted deliberately. Without `-y` the CLI
shows an interactive skill multi-select, so this is a picker over ~33 skills,
not a blind bulk install.

## Sync design

Four push-based workflows, modeled on Anthropic's battle-tested
claude-code-action → claude-code-base-action mirror (461+ automated sync
commits). Pattern per workflow: trigger `on: push` with a `paths:` filter +
`workflow_dispatch`, wipe **only the owned subdirectory** in the target,
`cp -r` fresh content, skip commit when `git diff --quiet`, commit as
`Sync from <repo>@<shortsha>` with the full source SHA in the body, push with
a **repo-scoped deploy key** (no PAT expiry, no `workflow` scope wall).

```
1. firecrawl/cli            push(skills/**)          ─▶ catalog skills/cli/
2. firecrawl (monorepo)     push(skills/**)          ─▶ catalog skills/build/
3. catalog                  push(skills/workflows/**) ─▶ firecrawl-workflows skills/
4. repo-lockdown on firecrawl-workflows (auto-close + lock PRs/issues with redirect comment)
```

Details:

- Workflows 1 and 2 write disjoint catalog paths; add
  `concurrency: { group: catalog-sync }` per source repo and a
  `git pull --rebase && git push` retry — never force-push.
- Workflow 3 copies **only the workflows subset**, so
  `npx skills add firecrawl/firecrawl-workflows` keeps meaning "the 16
  workflow skills," not the whole catalog.
- Mirrored directories get a `README.md` banner prepended on every sync:
  "Mirror of `<source>` — do not edit here, PR against `<source>`." The
  legacy mirror additionally gets issues disabled and branch protection so
  only the deploy key writes.
- Not chosen: git subtree (slow, re-splits history every push, no benefit for
  a snapshot mirror), pull-based cron (staleness + GitHub auto-disables
  scheduled workflows in quiet public repos after 60 days), symlinks/submodules
  (the skills CLI follows symlinks fragilely and does not clone submodules).

## `firecrawl init` revamp

Keep init — it is the only install surface whose UX we control — but fix what
it installs:

- Installs the **CLI skills by default** (a coding-agent CLI user's actual need).
- Shows a **multi-select for workflow skills** as optional extras.
- **Stops installing build skills** — they are for people integrating the SDK
  into product code, not CLI users. Build skills remain one command away and
  are what the editor plugins bundle.
- Implemented as a thin wrapper over
  `npx skills add firecrawl/skills --skill … -y`, so init's install volume
  accrues to catalog counters from day one and we maintain no install plumbing.

## Plugins (and the Codex fix)

Consolidate Cursor/Claude/Codex plugin metadata in the catalog repo, bundling
**CLI usage + build skills** (`skills/cli/` + `skills/build/`), not the 16
workflows — workflows are session recipes, not editor-integration material.

This fixes "Codex has the wrong skills" as a side effect: today the Codex
plugin in `firecrawl/skills` points at `./skills/`, which contains only build
skills. After the catalog sync lands, the same repo contains the CLI skills
too, and the plugin manifest is updated to reference the two bundled
categories. Bonus: the skills CLI also reads `.claude-plugin/marketplace.json`,
so plugin manifests in the catalog do double duty for discovery.

Plugin metadata in `firecrawl-workflows` stays as-is (backward compat) but is
no longer promoted.

## Contributor routing rule (goes in every README)

> **CLI skills → PR `firecrawl/cli`. Build/SDK skills → PR `firecrawl`
> (monorepo, `skills/`). Everything else (workflows, reference) → PR
> `firecrawl/skills`. New skills always land in an existing repo — when in
> doubt, `firecrawl/skills` under `skills/workflows/`.**

## Backward-compatibility matrix

| Existing surface | After migration |
|---|---|
| `npx skills add firecrawl/cli --skill firecrawl` | unchanged — cli repo is still the CLI source of truth |
| `npx skills add firecrawl/skills` | still works; now offers the full catalog via interactive picker |
| `npx skills add firecrawl/skills --skill <build-skill>` | unchanged — name-based selection is path-independent |
| `npx skills add firecrawl/firecrawl-workflows` | unchanged — served by the CI-synced mirror |
| `firecrawl init` | still works; installs CLI skills + optional workflows instead of build skills |
| Editor plugin installs | unchanged manifests keep resolving; bundles gain the CLI skills |
| skills.sh counts | no existing counter breaks; build counters continue; cli/workflows old counters keep counting legacy installs |

## Migration action plan

Additive-only; each phase leaves every surface working. Sequencing rule:
**populate and sync the catalog before any README or command points at it.**

### Phase 0 — prep (no user-visible change)
1. Generate deploy keys: cli→catalog, monorepo→catalog, catalog→workflows-mirror.
2. Archive `web-agent` and `opencode-firecrawl` (GitHub archive keeps installs
   working); point their READMEs at the catalog first.

### Phase 1 — build the catalog (additive inside `firecrawl/skills`)
3. Restructure: move the 5 build skills to `skills/build/`, the 2 index skills
   to `skills/reference/` (safe — counts are name-keyed, `--skill` is name-based).
4. Copy the 16 workflow skills into `skills/workflows/` — this is the new
   source of truth for workflows. `firecrawl-workflows` is untouched and still
   canonical until Phase 2 flips it.
5. Add sync workflow 1 to `firecrawl/cli` → catalog `skills/cli/` lands via CI.
6. Add the routing rule + per-category mirror notices to the catalog README.

### Phase 2 — flip sources of truth
7. Move build skills' authorship into the `firecrawl` monorepo root `skills/`
   (replacing today's pointer-README stubs); add sync workflow 2
   (monorepo → catalog `skills/build/`). Mark catalog `skills/build/` as mirror.
8. Add sync workflow 3 (catalog `skills/workflows/` → `firecrawl-workflows`);
   convert `firecrawl-workflows` to mirror: banner README, issues off,
   repo-lockdown, branch protection. Routing rule in its README points PRs at
   the catalog.
9. Freeze direct edits to mirrored dirs (banner + lockdown; social enforcement
   is enough given CI overwrites drift on next sync).

### Phase 3 — fix the integrations
10. Update the catalog's Codex/Cursor/Claude plugin manifests to bundle
    `skills/cli/` + `skills/build/`. This closes the Codex bug.
11. Revamp `firecrawl init`: CLI skills by default, workflow multi-select,
    drop build skills, delegate to `npx skills add firecrawl/skills`.

### Phase 4 — flip promotion
12. Update every README, doc, and marketing surface to promote exactly one
    command: `npx skills add firecrawl/skills` (plus `firecrawl init` for CLI
    onboarding). Old commands are never mentioned as removed — they simply
    stop being promoted.
13. Add the routing rule to `firecrawl/cli` and monorepo READMEs.

### Phase 5 — steady state
14. New skills land per the routing rule; the growth lever (keep shipping,
    especially workflows) now has exactly one obvious destination per skill type.
15. Revisit the parked A/B question (granular vs consolidated packaging) once
    catalog activation data accumulates — the catalog's category layout makes
    adding experimental consolidated bundles additive and cheap.

## Risks & open items

- **Catalog counters start at zero** for CLI and workflow skills. Mitigated by
  `firecrawl init` volume flowing to the catalog immediately, and by never
  breaking the old paths.
- **Drive-by PRs against mirrored catalog dirs** will happen. CI overwrite +
  banner + a maintainer redirect comment is sufficient; repo-lockdown on the
  catalog itself is not an option since two of its categories accept PRs.
- **skills.sh stale entries** (~41 shown vs 7 real in this repo) have no
  self-service cleanup mechanism today; they stay visible on the org page.
