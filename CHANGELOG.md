# Changelog

## 2026-05-31

- feat: `@adpharm/mcp-server-filesystem-ro` — read-only filesystem server; `index.ts` guard drops any tool not annotated `readOnlyHint: true`.
- docs: `README.md` + `CLAUDE.md` document the Adpharm read-only fork (rationale, maintainer tasks, upstream-merge guide).
- chore: `Taskfile.yml` adds `fs-ro:*` maintainer tasks (install/build/test/verify-readonly/ship/sync-upstream).
- chore: add `.devcontainer/`, `.claude/` settings + skills, `.agents/skills`, and `skills-lock.json`.
- feat: fork uses independent semver (`@adpharm` ro starts at `0.1.0`); `fs-ro:ship` records upstream base in `package.json` `upstreamVersion` + `src/filesystem/CHANGELOG.md`.
- docs: `src/filesystem/README.md` drops the 4 mutating tools + their annotation rows; reflects read-only build (every tool `readOnlyHint: true`).
