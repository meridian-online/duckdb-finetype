# duckdb-finetype — archived

> **This repository is archived and no longer maintained.**
> The FineType DuckDB extension is now built and released directly from the
> **[`meridian-online/finetype`](https://github.com/meridian-online/finetype)** repo —
> development, releases, and the DuckDB community-extension submission all live there.

## Installing the extension

Installation is unchanged — the `finetype` community extension is published from the
`finetype` repo:

```sql
INSTALL finetype FROM community;
LOAD finetype;
```

## Where things moved

| What | Now lives in |
|---|---|
| Extension source + build | [`meridian-online/finetype`](https://github.com/meridian-online/finetype) |
| Releases (binaries, Homebrew, community extension) | [`meridian-online/finetype`](https://github.com/meridian-online/finetype/releases) — see its release workflow |
| Docs (functions, type taxonomy) | [`meridian-online/finetype`](https://github.com/meridian-online/finetype#readme) |

This repo stays read-only for historical reference; its contents are preserved in git history.

## License

MIT
