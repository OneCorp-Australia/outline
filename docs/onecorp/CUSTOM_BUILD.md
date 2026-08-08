# OneCorp custom Outline fork

This fork tracks [`outline/outline`](https://github.com/outline/outline) with a small set of OneCorp-specific changes. The production Outline instance at https://kb.onecorp.au pulls a Docker image built from this fork.

## What's different from upstream

| Change | File(s) | Why |
|---|---|---|
| GHCR build workflow | `.github/workflows/build-ghcr.yml` | Builds and pushes a custom image to `ghcr.io/onecorp-australia/outline:latest` on every push to `main` and `feature/**` branches. The Hostinger VPS pulls from this registry. |

There are currently no source-code changes from upstream — the fork exists purely to
control how and where the image is built. Keeping it that way makes upstream merges
trivial, so prefer configuration or a generic Outline feature over patching source.

## Image registry

- **Registry:** GitHub Container Registry (GHCR), scoped to the `OneCorp-Australia` org
- **Image:** `ghcr.io/onecorp-australia/outline`
- **Tags:**
  - `latest` — the most recent build of the `main` branch
  - `<branch-name>` — tip of each feature branch (e.g. `feature-my-change`)
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
git push origin main
```

GitHub Actions will rebuild and republish automatically on the push.

## Embedding Fireflies recordings

No dedicated Fireflies embed provider exists. Use Outline's built-in **Embed** block
instead: insert an Embed block and paste the meeting URL. Two things to note —

- Pasting a Fireflies link into a document on its own will **not** auto-convert to an
  embed; the Embed block has to be inserted deliberately.
- Paste the **share** URL (`share.fireflies.ai/embed/meetings/...`), not an
  `app.fireflies.ai/view/...` link. The app URL is not frameable and will render blank.
  The meeting also has to be share-enabled in Fireflies.

## Future plan

If this fork ever carries no build-pipeline changes either, it can be dropped entirely
and the deployment reverted to `outlinewiki/outline:latest`.
