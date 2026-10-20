# setup-wright

A GitHub Action that installs the [Wright](https://github.com/wrightkit/wright) CLI and caches the verified executable. It only sets up Wright. Command selection, arguments, and gating stay in ordinary `run: wright ...` steps, so annotations, step summaries, and exit codes are Wright's own.

## Usage

Set up once, then run Wright as often as needed:

```yaml
jobs:
  wright:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v7
      - uses: wrightkit/setup-wright@v1
        with:
          version: 0.29.0
      - run: wright check
      - run: wright lint
      - run: wright analyze
```

| Input | Default | Meaning |
| --- | --- | --- |
| `version` | `latest` | Exact Wright version such as `0.29.0` (a leading `v` is accepted), or `latest`. |
| `cache` | `true` | Cache the verified executable. `false` downloads on every run. |

| Output | Meaning |
| --- | --- |
| `version` | Concrete Wright version installed. |
| `cache-hit` | `true` when a valid cached executable avoided the release download. |

Supported runners: Linux x86_64, macOS x86_64, macOS arm64, and Windows x86_64, the targets with published Wright release archives. Any other platform fails with an `unsupported platform` error.

## Versioning

The Action has its own versions, independent of Wright's. Use `@v1` for the moving major tag, an exact tag such as `@v1.0.0`, or a commit SHA. The `@ref` selects the Action's code. The `version` input selects the Wright binary.

## Version pinning

* An exact `version` is deterministic and, on a cache hit, makes no request to the release distribution.
* `latest` first reads the small `latest/version` pointer, then uses the concrete version for every later step. A cache hit for that version never re-downloads the archive. A new pointer value selects a new cache entry and installs the new binary.
* Pin an exact version in workflows that must be reproducible.

## Cache identity

Only the `wright` executable and a SHA-256 sidecar are cached, with the Actions cache service of the consuming repository. The key is exact:

```text
setup-wright-v1-<version>-<target-triple>
```

* There is no restore prefix, so a different version, OS, or architecture never restores into the wrong executable.
* `v1` is the cache layout. Changing the layout bumps it and abandons old entries.
* Actions caches are scoped to the consuming repository and ref. A pull request writes to its own scope, not to the default branch, and a fork pull request with a read-only token only restores. The Action needs no secrets and no permissions beyond `contents: read`.

On a miss the Action downloads the archive and its `.sha256` from the immutable release route, verifies the checksum before extraction, and only then saves the executable.

After every restore the Action recomputes the executable's SHA-256 against the sidecar and requires `wright --version` to report the requested version. A failing entry is deleted, a warning is printed, and the release is downloaded and verified again. An invalid entry keeps its key until the cache service evicts it, because Actions cache keys are immutable, so each run downloads again until then. The sidecar detects corruption and a wrong version. It does not authenticate the cache against a writer who can alter both files. That trust boundary is the consuming repository.

## What is not cached

Provider downloads, project sources, analysis or lint results, and build output are never cached. Providers remain managed by Wright itself. Set `cache: false` to skip the cache service entirely.

## Why a separate repository

The Action started inside `wrightkit/wright` (`uses: wrightkit/wright/.github/actions/setup-wright@<release-ref>`). The owner moved it here because that call path is awkward, and because consumers need a stable major tag that is independent of Wright's release refs.

## Development

* `scripts/test-setup-wright.sh` runs the helper against a mock release distribution: cache hit and miss (zero requests on a hit), version and `latest` resolution, key separation, corrupt and wrong-version caches, checksum mismatch, a missing release, and unsupported platforms.
* CI runs `actionlint`, `shellcheck`, those tests, and a smoke job that uses the Action as `uses: ./` on Linux, macOS, and Windows with a published Wright release. It installs twice and requires a cache hit on the second run.
* Releases follow [release-please](.github/workflows/release.yml): merge the Release PR to create the `vX.Y.Z` tag and move the major tag.

License: MIT.
