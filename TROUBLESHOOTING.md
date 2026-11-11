# Troubleshooting

Detail behind the short notes in [LAB.md](./LAB.md). You do not need any of this to
run the workshop; it is here for anyone who hits one of these and wants to know why.

## `aqe learning import` prints the same thing whether it worked or not

The import always finishes with output of this shape:

```
  Total patterns: 6
  Imported: 0
  Skipped: 6
```

The CLI counts a pattern as "skipped" whenever the optional ANN index cannot be
opened, including when SQLite, the authoritative store, took the write. So
`Imported: 0` tells you nothing either way.

`aqe learning stats` is the reliable check:

- **`Total: 0`**: there is no embedder, nothing was stored, and nothing will be
  until one is installed. See setup step 5 in the [README](./README.md), then
  re-run the import.
- **`Total: 28`**: it worked, with a smaller base. `aqe learning stats` was run once
  before the embedder was installed, which uses up AQE's one-time cross-domain seeding.
  You have AQE's 22 base patterns plus your six. Exercise 5 works as written.
- **`Total: 76`**: it worked: AQE's own foundational patterns plus your six. Do
  not re-run the import. The total grows as the exercises store learnings.

All three cases were confirmed on this repo with agentic-qe 3.14.1 and 3.14.8 (embedder present,
removed, and installed late), and the 76 case again on 3.15.18 (6 November 2026).

## `[VectorSearch] WARN: semantic vector search is not active`

From AQE 3.15 on, `aqe learning stats` and the first command that opens the vector index can print this single line, even with the embedder installed. It means the optional approximate-nearest-neighbour index was opened before an embedding runtime had initialised, so AQE leaves it unopened and searches SQLite instead. Your patterns are stored with their 384-dimensional embeddings either way. The reliable check is unchanged: `aqe learning stats` reading `Total: 76` after the seed import means setup worked. Confirmed on agentic-qe 3.15.18 on 6 November 2026, together with a Claude Code Exercise 5 that recalled four stored patterns.

## `VECTOR_SPACE_UNVERIFIED` and `brain.rvf`

The persistent vector index stays unverified without native RuVector provenance,
and AQE falls back to SQLite. SQLite is authoritative, so patterns are stored; only
the approximate-nearest-neighbour index is skipped. `brain.rvf.corrupt-NNNN` files
in `.agentic-qe/` are quarantined empty indexes, not lost data.

## Memory through the MCP tools on non-Claude agents

Claude Code captures and recalls learnings through the ReasoningBank hooks. Other
agents go through the `memory_store` / `memory_query` MCP tools, and in 3.14.1 that
round-trip is unreliable: `memory_store` can write without returning a response, and
`memory_retrieve` may not read the key back. Loading the seed brain and confirming
`aqe learning stats` reads 76 is the supported path for Exercise 5 on those tools.

## `npm install -g agentic-qe@latest` or `aqe init --auto` fails

- **Node version first.** Run `node -v`. AQE declares `node >= 22.13`. The session page lists Node 20, which installs with an `EBADENGINE` warning (and a slow native build) but is not supported. Install Node 22 LTS or newer, open a new terminal, and rerun.
- **`EACCES` or `permission denied` on the global install.** Install to your own folder instead:

  ```bash
  mkdir -p ~/.npm-global
  npm config set prefix ~/.npm-global
  export PATH=~/.npm-global/bin:$PATH
  npm install -g agentic-qe@latest
  ```

  Add the `export PATH=...` line to `~/.zshrc` or `~/.bashrc`.
- **Corporate proxy or conference wifi.** Set `npm config set proxy` and `https-proxy`, or use the Codespaces route in [docs/DEVCONTAINER.md](./docs/DEVCONTAINER.md).
- On Linux, a clean install and `aqe init --auto` succeeded on Node 20, 22 and 24 with agentic-qe 3.15.6, and on Node 22 with 3.15.18, so a failure on your machine is almost always one of the three above. Paste the first lines of your error to a facilitator.

## Coverage reads about 38%

Your clone predates the test-scope fix, so AQE's own scaffolding is being counted in
the denominator. `git pull`, then `npm run test:coverage` again. The app reads
**61.62%** overall, and `src/lib/payment-retry.ts` reads 100% of statements and
97.29% of branches.
