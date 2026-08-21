# Task: set up deploy keys for the Firecrawl skills sync workflows

You need admin access to these GitHub repos: `firecrawl/skills`, `firecrawl/cli`, `firecrawl/firecrawl`, `firecrawl/firecrawl-workflows`, and the `gh` CLI authenticated (`gh auth status`).

Three CI sync workflows push commits across repos. Each needs its own SSH deploy keypair: the **public** key goes on the repo being pushed to (deploy key, write access), the **private** key goes on the repo running the workflow (Actions secret). Do not swap these.

| # | Source (private key as secret) | Secret name | Target (public key as deploy key) |
|---|---|---|---|
| A | `firecrawl/cli` | `CATALOG_DEPLOY_KEY` | `firecrawl/skills` |
| B | `firecrawl/firecrawl` | `CATALOG_DEPLOY_KEY` | `firecrawl/skills` |
| C | `firecrawl/skills` | `WORKFLOWS_MIRROR_DEPLOY_KEY` | `firecrawl/firecrawl-workflows` |

A deploy key attaches to exactly one repo, so A and B need separate keypairs even though both target `firecrawl/skills`.

Run exactly this:

```bash
cd "$(mktemp -d)"

ssh-keygen -t ed25519 -N "" -f cli-to-catalog       -C "sync: firecrawl/cli -> firecrawl/skills"
ssh-keygen -t ed25519 -N "" -f monorepo-to-catalog  -C "sync: firecrawl/firecrawl -> firecrawl/skills"
ssh-keygen -t ed25519 -N "" -f catalog-to-workflows -C "sync: firecrawl/skills -> firecrawl/firecrawl-workflows"

# Public keys -> deploy keys on targets (write access required)
gh repo deploy-key add cli-to-catalog.pub       -R firecrawl/skills              --allow-write -t "sync from firecrawl/cli"
gh repo deploy-key add monorepo-to-catalog.pub  -R firecrawl/skills              --allow-write -t "sync from firecrawl/firecrawl"
gh repo deploy-key add catalog-to-workflows.pub -R firecrawl/firecrawl-workflows --allow-write -t "sync from firecrawl/skills"

# Private keys -> Actions secrets on sources (names must match exactly)
gh secret set CATALOG_DEPLOY_KEY          -R firecrawl/cli       < cli-to-catalog
gh secret set CATALOG_DEPLOY_KEY          -R firecrawl/firecrawl < monorepo-to-catalog
gh secret set WORKFLOWS_MIRROR_DEPLOY_KEY -R firecrawl/skills    < catalog-to-workflows

# Destroy local copies; keys now live only in GitHub
cd - && rm -rf "$OLDPWD"
```

## Verify

```bash
gh repo deploy-key list -R firecrawl/skills                # expect 2 keys, both read/write
gh repo deploy-key list -R firecrawl/firecrawl-workflows   # expect 1 key, read/write
gh secret list -R firecrawl/cli                            # expect CATALOG_DEPLOY_KEY
gh secret list -R firecrawl/firecrawl                      # expect CATALOG_DEPLOY_KEY
gh secret list -R firecrawl/skills                         # expect WORKFLOWS_MIRROR_DEPLOY_KEY
```

If the sync workflows already exist on `main`, also trigger one end-to-end run:

```bash
gh workflow run sync-catalog.yml -R firecrawl/cli && gh run watch -R firecrawl/cli
```

Success: a `Sync from firecrawl/cli@<sha>` commit lands on `firecrawl/skills` `main`, or the run logs "No changes to sync". Failure `Permission denied (publickey)` means a public/private half was swapped — redo that pair.

## Report back

- Output of the five verify commands
- Whether the test run succeeded
- Anything you had to deviate from
