# OneCorp custom Outline fork

This fork tracks [`outline/outline`](https://github.com/outline/outline) with a small set of OneCorp-specific changes. The production Outline instance at https://kb.onecorp.au pulls a Docker image built from this fork.

## What's different from upstream

| Change | File(s) | Why |
|---|---|---|
| Fireflies embed provider | `shared/editor/embeds/index.tsx`, `public/images/fireflies.png` | OneCorp uses Fireflies meeting recordings throughout the Sales KB. Adds auto-embed for `share.fireflies.ai/embed/meetings/...` URLs and `app.fireflies.ai/view/...` URLs. |
| GHCR build workflow | `.github/workflows/build-ghcr.yml` | Builds and pushes a custom image to `ghcr.io/onecorp-australia/outline:latest` on every push to `main` and `feature/**` branches. The Hostinger VPS pulls from this registry. |

## Image registry

- **Registry:** GitHub Container Registry (GHCR), scoped to the `OneCorp-Australia` org
- **Image:** `ghcr.io/onecorp-australia/outline`
- **Tags:**
  - `latest` — the most recent build of the `main` branch
  - `<branch-name>` — tip of each feature branch (e.g. `feature-fireflies-embed`)
  - `<branch>-<sha7>` — immutable per-commit reference for rollback
- **Visibility:** private (requires `read:packages` PAT to pull)

## How the Hostinger VPS uses this image

In `/opt/outline/docker-compose.yml` (or wherever the deploy lives) on the VPS, the `image:` line points at `ghcr.io/onecorp-australia/outline:latest` instead of upstream `outlinewiki/outline:latest`. To deploy a new build:

```bash
# On the VPS, after a GitHub Actions build completes
cd /opt/outline
docker compose pull
docker compose up -d
```

To roll back to a specific older build, change the tag to the `<branch>-<sha7>` form and re-pull.

## Keeping the fork current with upstream

Outline releases new versions every few weeks. To pull those in:

```bash
git remote add upstream https://github.com/outline/outline.git    # one-time
git fetch upstream
git checkout main
git merge upstream/main     # or rebase, your call
# Resolve any conflicts in shared/editor/embeds/index.tsx
git push origin main
```

GitHub Actions will rebuild and republish automatically on the push.

## Future plan

The Fireflies embed is generally useful — it's been submitted as a PR to `outline/outline` upstream. If accepted, this fork can be dropped (revert the deployment to `outlinewiki/outline:latest`).
