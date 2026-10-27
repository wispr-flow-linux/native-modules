# wispr-flow-linux native-modules - Development Notes

This repo rebuilds the two native sqlite addons Wispr Flow ships only as
Windows `.node` files, for Linux, and publishes them as pinned release assets
the main repo's build downloads.

```bash
scripts/rebuild-native-modules.sh   # the single source of rebuild truth
```

## Required reading

- The main repo's
  [`docs/learnings/electron42-v8-sqlite.md`](https://github.com/wispr-flow-linux/wispr-flow-linux/blob/main/docs/learnings/electron42-v8-sqlite.md)
  (why the V8 14.8 patch exists) and decisions D-007 and D-009 in
  `docs/decisions.md`.
- The main repo's
  [`MAINTAINING.md`](https://github.com/wispr-flow-linux/wispr-flow-linux/blob/main/MAINTAINING.md)
  for how this repo releases and what waits for the maintainer.

## Rules that bite

- Clean room: built only from public npm packages and the in-repo patch. No
  Wispr Flow code.
- The ABI is tied to the Electron the main repo stages. Targeting a new
  Electron here is only useful once the main repo moves to it; a mismatched
  ABI fails `fetch-native-bin.sh`'s lock check, which is the point.
- Builds run in `manylinux_2_28` (glibc 2.28 floor) and are validated by
  loading each `.node` under real Electron with an encrypted-DB round trip.
  Keep both steps; ELF magic alone can't tell a wrong-ABI build from a good
  one.
- `package-lock.json` pins the exact addon versions. Change it deliberately,
  never as a side effect.
- Shell follows the main repo's bash style guide: tabs, 80 columns, `[[ ]]`,
  no `set -e`.

## Releasing

Run **Build Native Modules** (`workflow_dispatch`) with `publish=true` and
`release_tag=native-vN`, then bump `native-modules-version.txt` in the main
repo. Publishing waits for the maintainer.

## Issues, PRs, and commits

Same conventions as the main repo: short and plain, evidence in the commit
body. AI-assisted PRs end with a
`Co-Authored-By: Claude <model> <noreply@anthropic.com>` trailer; commits
carry `Co-Authored-By: Claude <claude@anthropic.com>`.
