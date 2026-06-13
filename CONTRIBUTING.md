# Contributing & Release Ritual — Pour

## Commits
Format: Conventional Commits — `type(scope): summary`

- **type**: `feat` | `fix` | `refactor` | `chore` | `docs`
- **scope**: screen or system — `inventory`, `analytics`, `lifecycle`,
  `purchasing`, `dashboard`, `menu`, `calc`, `data-layer`, `standards`
- **summary**: imperative, lowercase, no period

Body (optional): what changed and why, wrapped ~72 chars.

Example:

    feat(analytics): add variance watchlist to dashboard

    Surface top-N SKUs by unexplained variance with drill-through
    to the lifecycle map.

## Tags (version archive)
Format: `v{major}.{minor}-{scope}` during screen-by-screen work.
Switch to semver (`vX.Y.Z`) once broader releases begin.

Each meaningful iteration gets a tag:

    git tag -a v0.3-lifecycle -m "Product Lifecycle Map hero screen complete"
    git push origin v0.3-lifecycle

## Ritual (per update)
1. Stage changes in GitHub Desktop
2. Paste the structured commit message (summary + description)
3. Commit and push
4. If the update is a meaningful iteration, run the tag commands

## Ritual (per tag)
When cutting a tag, also update the changelog:
1. Move the `[Unreleased]` notes into a new versioned entry in `CHANGELOG.md`
   matching the tag name and date
2. Commit that as `docs(standards): changelog for vX.Y-scope`
3. Then create and push the tag

Reference any former version at any time via `git checkout <tag>`.
