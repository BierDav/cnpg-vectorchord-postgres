# This fork

`patched` is the image the shared CloudNativePG cluster runs:
CloudNativePG's PostgreSQL 18 operand on Debian trixie with VectorChord, built by
`Dockerfile.custom` and published as
`ghcr.io/bierdav/cnpg-vectorchord-postgres:<pg>-vchord<vchord>-custom.<n>`
(also `:<pg>-vchord<vchord>`). The platform (`selfhosted-platform`,
`manifests/postgres/cluster.yaml`) pins it by digest and pre-pulls it on every
node (`charts/image-preload`).

Upstream, [tensorchord/cloudnative-vectorchord](https://github.com/tensorchord/cloudnative-vectorchord),
builds bookworm images for every PostgreSQL major from `versions.yaml`. This
fork changes none of upstream's files; it adds its own build.

## Keeping up with upstream

[fork-kit](https://github.com/BierDav/fork-kit) merges upstream `main` into
`patched` through one sync pull request. The sync runs centrally, from
BierDav/fork-kit's `sync-forks.yml`, with the fork-kit App; this repository
holds no App key. `.github/workflows/fork-kit.yml` is only the guard, which
sets the required `fork-kit/hold` status. Upstream's own `docker.yml` is
disabled here: it publishes to tensorchord's registries.

## Versions are pinned here

`Dockerfile.custom` pins the trixie base by digest and VectorChord by version
and checksum. It does not follow upstream's `versions.yaml`; the build prints a
notice when upstream builds newer versions. To upgrade:

- PostgreSQL minor: change the `FROM` tag and digest.
- VectorChord: change `VECTORCHORD_VERSION` and both checksums (from the
  release's `.deb` files). After the new image runs, `ALTER EXTENSION vchord
  UPDATE;` in the Photos database, and check the Gallery server supports
  that version first.

Every push to `patched` builds both architectures, tests them (user 26, a
fresh cluster with `vchord` preloaded, `CREATE EXTENSION vchord CASCADE` at the
pinned version, a `vchordrq` index), publishes the multi-arch image and creates
a GitHub release. Pull requests into `patched` build and test only.

## Repository settings this relies on

- **A ruleset on `patched`:** a pull request and the checks `build (amd64)`,
  `build (arm64)` and `fork-kit/hold`; no force pushes or deletion.
  Repository admins may bypass it.
- **Auto-merge** and merge commits are allowed.
- The `cnpg-vectorchord-postgres` package on GHCR is public: the nodes pull it
  without credentials.
