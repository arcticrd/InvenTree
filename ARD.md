# ARD InvenTree

This fork runs ARD's InvenTree: an upstream InvenTree release plus a small set of
ARD patches, built into the image `ghcr.io/arcticrd/inventree`.

## Branches and images

- `ard/<version>` (for example `ard/1.5.7`): the upstream release tag with the ARD
  patches on top. Each patch is its own commit.
- Tag a commit on that branch `<version>-ard.<n>` (for example `1.5.7-ard.1`) to build
  and push `ghcr.io/arcticrd/inventree:<tag>` (`.github/workflows/ard-image.yaml`).
- The image is private. The work VPS pulls it with a read-only token
  (`docker login ghcr.io` with a classic token that has only `read:packages`, then
  `docker logout`). Live never talks to the registry: the image is copied from the
  VPS with `docker save ghcr.io/arcticrd/inventree:<tag> | gzip | ssh <live> 'gunzip | docker load'`.
- Staging and live use that image in place of `inventree/inventree` (an `image:` line
  for `inventree-server` and `inventree-worker` in `docker-compose.override.yml`).
  After switching images, run `invoke plugins` in the worker and restart it, then
  `docker compose exec inventree-server invoke static` so the proxy serves the
  image's web UI. `invoke static` empties the static folder, so on live copy the ARD
  logo back afterwards: `sudo cp -p inventree-data/branding/ard-logo.svg
  inventree-data/static/branding/` (`INVENTREE_CUSTOM_LOGO=branding/ard-logo.svg`).

## Patches

| Patch | Why |
| --- | --- |
| Revision column shown by default (sales order lines and allocations, purchase order lines, build orders and build lines, stock), and shown once again in browsers that hid it under the old default | ARD part revisions are versions; people must see which version they sell, buy, build or stock |

## Upgrading to a new InvenTree release

1. `git fetch upstream --tags` (upstream = `https://github.com/inventree/InvenTree`).
2. `git switch -c ard/<new version> <new version>`, then cherry-pick the ARD patch
   commits from the previous `ard/*` branch (`git log <old version>..ard/<old version>`).
3. Push, tag `<new version>-ard.1`, wait for the image, then follow InvenTree's upgrade
   steps with the new image name on staging first, then live.
