<p align="center">
  <img src="docs/banner.svg" alt="dsh-purge" width="640">
</p>

# dsh-purge (enhanced fork)

A DeepSeek Harness jailbreak/settings plugin. It rewrites the default identity, permission policy, and tool limits inside the locally installed `@deepseek-ai/*` packages, and adds a **Rules** section in Settings for inspecting patches in groups, applying/reverting them, and keeping multiple `AGENTS.md` / `CLAUDE.md` rule sets. It re-checks and re-applies on startup, so an `npm` upgrade that overwrites `node_modules` needs no manual re-run.

Aligned with **dsh 0.1.2-rc.1**, Node.js ≥18. On other versions, unmatched originals show as pending or skipped; nothing is rewritten blindly.

This repository is an enhancement fork. Delta over upstream `v1.3.5`:

- **New patch #38** — dynamic TUN fake-ip pass-through. When Clash / mihomo takes over DNS in fake-ip mode, upstream rejects every hostname as an SSRF target (`resolves to a non-public IP address`); this patch admits pool addresses behind a sentry probe and restores rejection as soon as the tunnel goes away, with no hardcoded allowlist.
- Fixes patch #22's `timeoutMs` anchor, which previously appended another `0` on every apply.
- Fixes patch #38's anchor so the helper is injected exactly once.
- Node.js 24 compatibility: `fsExists()` gained a type guard, removing the `fs.existsSync` deprecation warning and blank output rows.
- Matches the `prefix` / `personaPrefix` keys and `>-` / `|-` block scalar forms, and adds a `web-fetch-http` path mapping.

Full details in [CHANGELOG.md](CHANGELOG.md).

## Install

After installing you must **fully quit and restart** DeepSeek Harness before the Rules section appears; the `@deepseek-ai` packages are only changed once you press Apply.

```sh
# Web profile
dsh plugin --profile web add github:mortyandtimo/dsh-purge-plus

# Desktop profile
dsh plugin --profile default add github:mortyandtimo/dsh-purge-plus
```

From a local checkout instead:

```sh
git clone https://github.com/mortyandtimo/dsh-purge-plus.git
cd dsh-purge-plus
dsh plugin --profile web add .
```

Or hand this whole README to a local coding agent and let it install, apply, and restart on its own.

### After installing

1. Restart the dsh process, otherwise neither the Rules section nor the chat commands load.
2. Apply patches: `/purge apply` in chat, or Apply in Settings → Rules, or `dsh-purge --apply`.
3. Verify: `/purge status` prints `DSH_HOME` and the patch list; the Rules card appears in Settings (Ctrl+F5 if cached).
4. `#20` / `#21` need `dsh-web-fetch-http`, and `#28` / `#29` need `dsh-liangshen`. Skipping them when those plugins are absent is normal — do not retry.

## Usage

```sh
# CLI
dsh-purge --status | --apply | --revert | --edit

# Chat
/purge status | apply | revert | edit | help
/rules list | use <id> | create <id> | delete <id> | reset

# Model tools
purge_status   purge_apply   purge_revert
```

Applied patches load only after a restart. The Settings restart button is the only thing that restarts; nothing restarts automatically.

## Revert and uninstall

- Every target file is backed up as `<file>.dshpurge.bak` before it is modified; Revert or `/purge revert` restores from the backup and deletes it. Re-applying is idempotent.
- `prompt-inject.md` is a user file and survives a revert.
- Uninstall:

```sh
# Revert in Settings first, then
dsh plugin --profile web remove dsh-purge
dsh plugin --profile default remove dsh-purge
```

## Notes

- Changes are limited to rendering copy, default policy, and execution logic inside the local `@deepseek-ai/*` packages, plus the override file and rule sets under `$DSH_HOME`. The Harness source repository is never modified.
- No hardcoded drive letters: locations resolve through `$DSH_HOME` / `DSH_BASE` → a `.dsh` beside the dsh launcher → the system default `~/.dsh`. When nothing matches it asks for `DSH_BASE` rather than guessing.
- Third-party plugins are out of scope (the Windows no-flash work is a runtime patch; their repositories are not touched).
- After an upgrade, unmatched originals report `pattern_not_found` or stay pending — nothing is rewritten blindly.
- License: MIT, see [LICENSE](LICENSE).

## Local checks

```sh
node --check lib/index.js
node --check lib/core.js
node --check client.js
```
