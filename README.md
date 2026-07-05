# rust-boilerplate

Opinionated cargo-generate template for a new Rust cargo workspace. Bootstraps:

- Cargo workspace with a single example lib crate under `crates/{{crate_name}}/`.
- MSRV pin via `rust-toolchain.toml` (single source of truth for CI too).
- `.cargo/config.toml` with cross-compilation targets + custom profiles (`production`, `production-with-debug`, `ci`, `profiling`, `profiling-sharp`, `flamegraph`).
- `lat.md/` knowledge-graph scaffolding.
- Agent config for Claude Code, Copilot, and generic Codex agents (`AGENTS.md`, `CLAUDE.md`, `.claude/`, `.agents/`, `.github/copilot-instructions.md`).
- MCP servers wired for `lat`, `fff-mcp`, `rust-analyzer-mcp` (VS Code + Claude Code).
- Shared reusable GitHub Actions workflows: `ci`, `pr-title` lint, `release` (`release-plz`).
- `release-plz.toml` conventional-commits changelog config.
- `specs/` layout for the spec-driven workflow (requirements → design → tasks → validation).
- MIT license + CHANGELOG stub.

## Usage

```bash
cargo install cargo-generate  # once per machine

cargo generate \
  --path /path/to/this/template \
  --name my-module
```

Or straight from git if you publish the template as its own repo:

```bash
cargo generate --git https://github.com/<owner>/rust-boilerplate --name my-module
```

`cargo generate` prompts for each placeholder declared in `cargo-generate.toml`:

| Placeholder             | Purpose                                                                     |
| ----------------------- | --------------------------------------------------------------------------- |
| `project-name`          | Kebab-case module / repo name (e.g. `my-module`).                           |
| `crate_name`            | snake_case crate name (auto-derived from `project-name` by default).        |
| `description`           | One-line description written into `Cargo.toml` + `README.md` + crate docs.  |
| `absolute_path`         | Absolute path to the generated module dir. Pins `fff-mcp` server arg.       |
| `author_name`           | Author name for `Cargo.toml` + `LICENSE`.                                   |
| `author_email`          | Author email for `Cargo.toml`.                                              |
| `homepage`              | Homepage URL.                                                               |
| `repository`            | Repository URL.                                                             |
| `msrv`                  | MSRV pin. Written to `rust-toolchain.toml` + `[workspace.package]`.         |
| `ci_owner`              | GitHub owner (user or org) hosting the shared `ci` repo.                    |
| `ci_ref`                | Git ref of the shared `ci` repo to pin against (branch, tag, or SHA).       |
| `publish_to_crates_io`  | Whether `release-plz` publishes to crates.io on release-PR merge.           |

## After generation

1. `cd <new-module>`.
2. `git init && git add -A && git commit -m "chore: initial commit"`.
3. Install local git hooks: `just hooks` (runs `lefthook install`). Requires `lefthook` + `convco` on PATH (`cargo install lefthook convco` or `brew install lefthook convco`).
4. Set `absolute_path` correctly in `.mcp.json` and `.vscode/mcp.json` if you moved the folder.
5. Set required secrets in the new repo: `RELEASE_PLZ_TOKEN` (PAT with `contents: write` + `pull-requests: write`) and, if publishing, `CARGO_REGISTRY_TOKEN`.
6. Push. First push triggers CI matrix.

## Local git hooks (lefthook)

`lefthook.yml` at the repo root defines three hook stages:

| Stage        | Runs                                                                        |
| ------------ | --------------------------------------------------------------------------- |
| `pre-commit` | branch-name check, `cargo fmt --check`, `cargo clippy -D warnings`, `lat check` (parallel). |
| `commit-msg` | `convco check` — Conventional Commits gate.                                 |
| `pre-push`   | block direct push to `main`/`master`, then `cargo nextest run --workspace`. |

Bypass a single hook with `LEFTHOOK=0 git commit …`. Never bypass in shared work — CI enforces the same gates and will re-fail. To skip only one specific command, use `LEFTHOOK_EXCLUDE=clippy git commit …`.

To disable a check in a specific repo without editing the committed config, copy the block into an untracked `lefthook-local.yml` (already gitignored) and set `skip: true`.

## License

Dual-licensed **`MIT OR Apache-2.0`** (Rust ecosystem convention — matches `serde`, `tokio`, `hyper`). Ships:

- `LICENSE-MIT` — MIT license text with your copyright holder.
- `LICENSE-APACHE` — verbatim Apache 2.0.
- `NOTICE` — Apache 2.0 attribution notice.

Both licenses grant permission; downstream picks either.

### Switching to a different license

If a specific module needs a different license (e.g. proprietary, AGPL, single-MIT), update these places consistently — none are auto-derived:

1. Replace or delete `LICENSE-MIT` / `LICENSE-APACHE` / `NOTICE`. For a single license, drop the extras and add a single `LICENSE`.
2. `Cargo.toml` → `[workspace.package] license = "MIT OR Apache-2.0"` → your SPDX identifier (`Apache-2.0`, `MIT`, `AGPL-3.0-only`, `GPL-3.0-or-later`, `MPL-2.0`, `LicenseRef-Proprietary` for closed-source, …).
3. Every `crates/*/Cargo.toml` — same field. Workspace inheritance is opt-in per field; generated crate `Cargo.toml` sets `license` explicitly so no silent divergence.
4. This `README.md` and each crate `README.md` — swap the "License" section.
5. For proprietary crates also set `publish = false` in the crate `Cargo.toml` to guarantee they never go to crates.io by accident.

## Layout

```
<generated-module>/
  .cargo/config.toml        cross-compile targets + custom profiles
  .github/
    workflows/              thin caller stubs (ci, pr-title, release)
    copilot-instructions.md
  .claude/                  Claude Code local settings + skills
  .agents/                  generic agent skills
  .vscode/mcp.json          MCP servers for VS Code
  .mcp.json                 MCP servers for Claude Code
  crates/
    {{crate_name}}/         example lib crate
      Cargo.toml
      src/lib.rs
      tests/integration.rs
      README.md
  docs/                     project docs
  lat.md/                   lat.md knowledge graph
  specs/                    spec-driven workflow artifacts
  AGENTS.md                 agent-facing project rules (memory, search, code comments…)
  CLAUDE.md                 one-line `@AGENTS.md` import
  CHANGELOG.md              maintained by release-plz
  Cargo.toml                workspace root
  LICENSE                   MIT
  README.md                 this file (regenerated with the module name)
  release-plz.toml          conventional-commits changelog config
  rust-toolchain.toml       MSRV pin (SOT for CI too)
```

## Single-crate vs workspace

Default = workspace layout with one example lib crate. To collapse for a single-crate repo: move `crates/{{crate_name}}/` contents up to the root, delete the workspace `[workspace]` block from `Cargo.toml`, and swap `edition.workspace = true` → `edition = "2024"` (etc.) in the crate's own `Cargo.toml`. Keeping the workspace layout is recommended even for one-crate projects — it lets you add more crates later without a Cargo.toml surgery.

## What is excluded from the template

`cargo-generate.toml`'s `ignore` list drops content that must not leak into new modules:

- `specs/**` — example specs (each generated module starts with an empty `specs/` dir).
- `docs/*.md`, `docs/*.pdf` — reference docs kept in the boilerplate for author convenience only.
- `cargo-generate.toml`, this `README.md`.
