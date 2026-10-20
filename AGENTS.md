# setup-wright

Cached `setup-wright` GitHub composite Action: installs the Wright CLI and caches the verified executable. Read the organization routing in [`wrightkit/.github`](https://github.com/wrightkit/.github/blob/main/AGENTS.md) first. This file adds only what is specific to this repository.

- Wright owns the release archives, checksums, and the `latest/version` pointer. This repository only consumes them. Do not change that contract here.
- Keep the Action to setup. No command wrappers, no provider/project/build caching, no write token or secrets.
- Keep the cache key exact. Never add a restore prefix. Verify SHA-256 before cache save and before use, and never execute an unverified cached binary.
- `action.yml` is the public contract (inputs, outputs, `uses: wrightkit/setup-wright@v1`). Change it only with `README.md` and the helper tests in the same PR.
- Run `scripts/test-setup-wright.sh`, `actionlint`, and `shellcheck -S warning scripts/*.sh` before a PR. The `smoke` CI job is the real consuming-workflow check and pins an exact Wright version.
- Never create tags or releases by hand. `.github/workflows/release.yml` owns `vX.Y.Z` and the moving `v1`.
- Pin third-party actions to a full commit SHA.
