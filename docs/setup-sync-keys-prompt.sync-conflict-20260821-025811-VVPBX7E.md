# Task: reconcile Firecrawl sync deploy keys to the pure-catalog architecture

You need admin access to these GitHub repos: `firecrawl/skills`, `firecrawl/cli`, `firecrawl/firecrawl`, `firecrawl/firecrawl-workflows`, and the `gh` CLI authenticated (`gh auth status`).

Context: three sync keypairs were previously provisioned. The architecture then changed — the catalog (`firecrawl/skills`) is now fully read-only and `firecrawl-workflows` remains a source repo, so the catalog→workflows sync was **reversed** to workflows→catalog. One keypair is now wrong-direction garbage, and one new keypair is needed. The other two stay.

## Target state

| # | Source (private key as Actions secret) | Secret name | Target (public key as write deploy key) | Action |
|---|---|---|---|---|
| A | `firecrawl/cli` | `CATALOG_DEPLOY_KEY` | `firecrawl/skills` (title: "sync from firecrawl/cli") | **keep** |
| B | `firecrawl/firecrawl` | `CATALOG_DEPLOY_KEY` | `firecrawl/skills` (title: "sync from firecrawl/firecrawl") | **keep** |
| C | `firecrawl/skills` | `WORKFLOWS_MIRROR_DEPLOY_KEY` | `firecrawl/firecrawl-workflows` (title: "sync from firecrawl/skills") | **remove both halves** |
| D | `firecrawl/firecrawl-workflows` | `CATALOG_DEPLOY_KEY` | `firecrawl/skills` (title: "sync from firecrawl-workflows") | **create** |

Rules: public halves are deploy keys on the repo being pushed to (write access); private halves are Actions secrets on the repo running the workflow. Never swap them. A deploy key attaches to exactly one repo, so D must be a fresh keypair — do not reuse A/B/C material.

## Step 1 — create pair D

```bash
cd "$(mktemp -d)"
ssh-keygen -t ed25519 -N "" -f workflows-to-catalog -C "sync: firecrawl/firecrawl-workflows -> firecrawl/skills"

gh repo deploy-key add workflows-to-catalog.pub -R firecrawl/skills --allow-write -t "sync from firecrawl-workflows"
gh secret set CATALOG_DEPLOY_KEY -R firecrawl/firecrawl-workflows < workflows-to-catalog

# Destroy local copies; the key now lives only in GitHub
cd - && rm -rf "$OLDPWD"
```

## Step 2 — remove pair C (both halves)

```bash
# Find the deploy key id on the old target, then delete it
gh repo deploy-key list -R firecrawl/firecrawl-workflows
gh repo deploy-key delete <ID-of-"sync from firecrawl/skills"> -R firecrawl/firecrawl-workflows

# Delete the orphaned secret on the catalog
gh secret delete WORKFLOWS_MIRROR_DEPLOY_KEY -R firecrawl/skills
```

Do NOT touch the two deploy keys on `firecrawl/skills` titled "sync from firecrawl/cli" and "sync from firecrawl/firecrawl", and do not touch `CATALOG_DEPLOY_KEY` on `firecrawl/cli` or `firecrawl/firecrawl` — those are pairs A and B and stay live.

## Step 3 — verify

```bash
gh repo deploy-key list -R firecrawl/skills                # expect exactly 3 write keys: cli, firecrawl, firecrawl-workflows
gh repo deploy-key list -R firecrawl/firecrawl-workflows   # expect 0 sync keys (pair C gone)
gh secret list -R firecrawl/cli                            # expect CATALOG_DEPLOY_KEY
gh secret list -R firecrawl/firecrawl                      # expect CATALOG_DEPLOY_KEY
gh secret list -R firecrawl/firecrawl-workflows            # expect CATALOG_DEPLOY_KEY
gh secret list -R firecrawl/skills                         # expect NO sync secrets (WORKFLOWS_MIRROR_DEPLOY_KEY gone)
```

If the sync workflow already exists on `firecrawl-workflows` main, run one end-to-end test:

```bash
gh workflow run sync-catalog.yml -R firecrawl/firecrawl-workflows && gh run watch -R firecrawl/firecrawl-workflows
```

Success: a `Sync from firecrawl/firecrawl-workflows@<sha>` commit lands on `firecrawl/skills` `main` (or the run logs "No changes to sync"). Failure `Permission denied (publickey)` means the public/private halves were swapped — redo pair D.

## Report back

- Output of the six verify commands
- Whether the test run succeeded (or that the workflow isn't merged yet)
- Anything you had to deviate from
