# omarchy-pkgs

The user's fork of omacom/omarchy-pkgs: PKGBUILD recipes and tooling for the packages Omarchy
publishes at `https://pkgs.omarchy.org/<channel>/<arch>` (channels `edge` → `rc` → `stable`,
arches `x86_64` and `aarch64`). The fork exists to send package PRs upstream (branches
`cloudmail`, `ghost`, `omacloud`, `onecloud`, each adding one `pkgbuilds/<name>/`).

Upstream moves fast and local `master` may lag it badly. Before any work: `git fetch upstream`,
branch from `upstream/master`, and trust upstream's README.md and `docs/` over this file.

## Code index (as of upstream/master, October 2026)

- `pkgbuilds/<package>/` — one recipe per package: `PKGBUILD`, `.omarchy/package.json`
  (metadata: `source`, `upstream` watch, `release_ring`, `channels`, `pinned`, `rebuild_on`, ...;
  table in README "Package metadata"), optional `.omarchy/upstream.sh` hook and `test.sh`.
- `bin/` — tooling: `add-package` (scaffold or one-time AUR import), `build` (Docker build into
  `build-output/<channel>/<arch>/`), `list-packages`/`bin/repo list` (`--json`), `sync-upstream`,
  `sync-rebuilds`, `omarchy-release` and `repo advance` (release train), plus old repository-host
  scripts (`repo release|sync|push|deploy`, `setup`, timers) that must not be run.
- `build/` — builder image and in-container build script. `helpers/` — shared shell/Python
  (`upstream-watch.py`, `package-metadata.sh`, ...). `ci/` — x86_64 builder pool (DigitalOcean).
- `tests/` — shell/Python/Node tests run by `.github/workflows/test.yml`.
- `.github/workflows/` — the pipeline: `build-pr.yml` (builds changed packages, check `result`),
  `test.yml` (`self-tests`, `build-isolation`), `publish.yml` (signs and publishes on push to
  master), `sync-upstream.yml`/`sync-rebuilds.yml`/`track-branches.yml` (bot PRs).
- `docs/` — `builds.md`, `upstream-sources.md`, `rebuild-triggers.md`, `releases.md`.

## Commands

Needs git, bash, jq, curl, zstd-capable tar, and Docker/Podman (on Arch also Python 3.11+,
`vercmp`, `bsdtar` for upstream checks).

```sh
bin/add-package my-package --local --scaffold        # new recipe
bin/build --package my-package --dry-run             # plan only; does not validate
bin/build --package my-package [--arch aarch64]      # unsigned build into build-output/edge/<arch>/
python helpers/upstream-watch.py check pkgbuilds/my-package   # upstream watch finds the release
tests/pinned-sources.sh                               # every git source pinned
updpkgsums                                            # in the package dir, after changing sources
```

## Conventions and gotchas

- One package per PR, touching only `pkgbuilds/<package>/` (plus an optional new test and its one
  line in `test.yml`); anything else needs a maintainer merge. Tooling changes go in a separate
  PR first: PRs build with `master`'s tooling.
- Pin every git source (`#commit=` / `#tag=`); list every supported arch in `arch=()`; new
  packages declare an `upstream` `watch` so bots propose updates. Same-version packaging changes
  bump `pkgrel`; new versions reset it to 1; never set `epoch` without agreement.
- Never reuse a published filename for different bytes (CDN caches by name). Publishing happens
  only through `publish.yml`; never run `bin/repo release`, `sync`, `push` or `deploy`.
- `build-output/`, `pkgs.omarchy.org/`, `.SRCINFO` and makepkg leftovers are gitignored; delete
  the source clone `updpkgsums` leaves before committing.
