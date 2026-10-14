# wispr-flow-linux-native-modules

> Standalone repo (`github.com/wispr-flow-linux/native-modules`), split out of the
> `wispr-flow-linux` monorepo. `native-v*` Release assets hold prebuilt Linux
> sqlite addons, which the main `wispr-flow-linux` package build downloads
> instead of compiling them itself. Lives in its own repo so these CI-consumed
> artifacts don't inflate the main project's Release download counts.

Clean-room Linux rebuild of the two native sqlite addons Wispr Flow ships only as
**Windows PE `.node`**:

- `better-sqlite3-multiple-ciphers` — the encrypted SQLCipher DB engine
- `sqlite3` — plain sqlite

On Linux they must be rebuilt against the target Electron's V8 / Node ABI
(Electron 44.7.2: V8 15.2 / Node 24, ABI 149).
`better-sqlite3-multiple-ciphers` does **not** compile against V8 14.8 or newer unpatched;
the clean-room patch in [`scripts/patches/`](scripts/patches/) is applied to a
pristine public-npm checkout. **No Wispr Flow code is involved** — these are
rebuilt entirely from public npm + the in-repo V8 patch, mirroring the
[clean-room helper](https://github.com/wispr-flow-linux/helper).

## Layout

| Path | Role |
|---|---|
| `scripts/rebuild-native-modules.sh` | The single source of rebuild truth (deterministic, lockfile-pinned). |
| `scripts/native-modules/` | Pinned `package.json` / `package-lock.json` + the Electron smoke test. |
| `scripts/patches/v8-14.8-better-sqlite3-multiple-ciphers.patch` | The V8 ABI patch (written for 14.8, still applies on 15.2). |
| `.github/workflows/build-native-modules.yml` | build (old glibc) → validate (real Electron 44) → publish to a `native-v*` release. |

## Releases

Each `native-v*` release ships, per arch (`x86_64`, `aarch64`):

```
better_sqlite3-<arch>.node   node_sqlite3-<arch>.node
native-modules-<arch>.lock   (provenance: electron, abi, patch_sha256, …)
SHA256SUMS                   (over all .node)
```

Built on **glibc 2.28** (`manylinux_2_28`) so the addons load on older-but-still-
supported distros, and validated by loading each `.node` under real Electron 44
plus an encrypted-DB round-trip + wrong-key smoke test.

## Consumed by

The main build pins a release tag in
[`native-modules-version.txt`](https://github.com/wispr-flow-linux/wispr-flow-linux/blob/main/native-modules-version.txt)
and stages the matching assets via
[`scripts/setup/fetch-native-bin.sh`](https://github.com/wispr-flow-linux/wispr-flow-linux/blob/main/scripts/setup/fetch-native-bin.sh),
which verifies each download against `SHA256SUMS` **and** checks the asset's
`native-modules.lock` `patch_sha256`/`abi` against the consuming checkout — ELF
magic alone can't tell a stale, wrong-ABI, or wrong-patch build from a correct
one.

## Publishing a new build

Run the **Build Native Modules** workflow (`workflow_dispatch`) with
`publish=true` and `release_tag=native-vN`, then bump
`native-modules-version.txt` in the main repo to the new tag. A pull request
touching the rebuild script, the pinned project, the patch, or the workflow runs
build+validate **without** publishing, so changes are proven before release.

## License

Public domain ([UNLICENSE](UNLICENSE)).
