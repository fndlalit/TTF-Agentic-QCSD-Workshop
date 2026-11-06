# Container verification

Verified 6 November 2026 against agentic-qe 3.15.18, Node 22.23, on Linux. Reviewed by running the container's own scripts in a disposable copy of this repository.

| Check | Result |
|---|---|
| `npm ci` and `aqe init --auto --skip-code-index` (what post-create runs) | Passed: 86 skills, 60 AI agents |
| `bash .devcontainer/prepare-workshop.sh` | Passed: backup taken, `src/` indexed, six seed patterns imported |
| `bash .devcontainer/verify.sh` | Passed: 21 source files indexed; 6/6 seed patterns with 384-dimensional embeddings; 76 patterns; integrity `ok` |
| `node .devcontainer/verify-mcp.mjs` | Passed: handshake and tool list, 91 tools; memory recall query returned 0 entries (reported as a limitation) |
| `bash .devcontainer/test-lifecycle.sh` | Passed |
| `bash .devcontainer/smoke-app.sh` | Passed: catalog, cart, missing-order HTTP 400; no payment attempted |
| `npm run test:coverage` | 12 files, 333 tests passed; 61.62% statements overall |
| `npx --no-install tsc --noEmit` | Passed |
| Hypergraph after container-style indexing | 110 nodes / 89 edges (function 68, module 21, file 21) |

## Not verified here

- **The container image build itself.** The sandbox used for this check has no Docker daemon, so `docker build` and a real Codespace were not run. The GitHub Actions workflow in `.github/workflows/devcontainer.yml` builds the image and repeats the preparation, MCP check, coverage, type check and app smoke check on Linux amd64: read its first run on the Actions tab after the first push. Create one Codespace before the session to confirm the full path.
- **Agent sign-in inside the container** and the paid exercises as run from a container. The exercises were run unattended with Claude Code from a manual setup; see LAB.md for the measured results.
- **Tools other than Claude Code.** The step-list prompts were run through Claude Code only, not on Copilot, Kiro or Codex.
- **MCP memory recall on non-Claude agents.** Unchanged from TROUBLESHOOTING.md: verify recall on your chosen agent, and use the reports fallback in Exercise 5 if needed.

The app's intentional defects and its dependency lockfile were preserved.
