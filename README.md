# ReadmeForge Action

A thin-client GitHub Action that keeps your `README.md` in sync with your code.
On every push it sends the diff + current README to the ReadmeForge API, which
returns an updated README. **No AI key needed in your repo** — the API holds it.

## Quick start

```yaml
name: readme-sync
on:
  push:
    branches: [main]

jobs:
  sync:
    # Prevents infinite loops when using mode: commit
    if: "!contains(github.event.head_commit.message, '[skip readmeforge]')"
    runs-on: ubuntu-latest
    permissions:
      contents: write       # push branches / commit README
      pull-requests: write  # open PRs (mode: pr)
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # required: the Action diffs against the previous commit

      - uses: haimhm/readmeforge-action@v1
        with:
          mode: pr   # or 'commit' (Pro)
          # license-key: ${{ secrets.READMEFORGE_LICENSE_KEY }}  # Pro only
```

> Replace `haimhm/readmeforge-action@v1` with the published Action reference
> once it is released on the GitHub Marketplace.

## Inputs

| Input | Default | Description |
|---|---|---|
| `api-url` | `https://readmeforge-api.readmeforge.workers.dev/v1/sync` | ReadmeForge API endpoint (override for self-hosting) |
| `license-key` | `""` | Pro license key from Gumroad. Store as a repo secret (`READMEFORGE_LICENSE_KEY`) |
| `mode` | `pr` | `pr` opens a review-first pull request · `commit` pushes directly (Pro) |
| `readme-path` | `README.md` | Path to the README file |
| `github-token` | `${{ github.token }}` | Token for pushing / opening PRs |

## Outputs

| Output | Description |
|---|---|
| `updated` | `true` if the README changed |
| `tier` | `free` or `pro` — which API tier served this run |
| `pr-url` | PR URL (only in `pr` mode) |

## Tiers

| | Free | Pro ($9/mo) |
|---|---|---|
| Public repos | ✅ 20 syncs/month | ✅ 1000 syncs/month |
| Private repos | ❌ | ✅ |
| `mode: commit` (auto-merge) | ❌ | ✅ |

Free tier needs no key at all. For Pro, buy a license on Gumroad, then add it
as a repository secret named `READMEFORGE_LICENSE_KEY` and pass it via the
`license-key` input.

## Notes

- **Loop guard:** in `commit` mode the bot commits with `[skip readmeforge]` in
  the message. Keep the `if:` condition on the job so the workflow doesn't
  trigger itself.
- **First push to a branch** (where `before` is all zeros) is handled: the
  Action diffs against the empty tree.
- **Permissions:** the default `GITHUB_TOKEN` is enough. For `pr` mode the repo
  must allow GitHub Actions to create pull requests
  (Settings → Actions → General → Workflow permissions).
- **What leaves your repo:** the unified diff of the push and the current
  README, sent to the ReadmeForge API over HTTPS. Your code is never stored —
  only a monthly per-repo usage counter.
- **No diff / no doc-relevant changes:** the Action exits quietly with
  `updated=false` and touches nothing.
