# Changelog

`@adpharm/mcp-server-filesystem-ro` — a read-only fork of
[`@modelcontextprotocol/server-filesystem`](https://www.npmjs.com/package/@modelcontextprotocol/server-filesystem).

This package uses its **own** semantic version line (it does not mirror upstream's version). Each entry records the
upstream version the release was built from, so the lineage is always recoverable. The current upstream base is also
stored in the `upstreamVersion` field of `package.json`. New releases are added below the marker by `task fs-ro:ship`.

<!-- releases -->

## 0.1.0 (2026-05-31, upstream 0.6.3)
- Initial `@adpharm` read-only fork of `@modelcontextprotocol/server-filesystem` 0.6.3.
- Only tools annotated `readOnlyHint: true` are exposed; the four mutating tools (`write_file`, `edit_file`, `create_directory`, `move_file`) are filtered out at registration.
