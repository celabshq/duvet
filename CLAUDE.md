# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is duvet?

Duvet is a requirements traceability tool that links implementation code to specification documents (RFCs, Markdown specs). It parses annotations in source comments, matches them against specs, and generates coverage reports. The project dogfoods itself — `.duvet/config.toml` tracks duvet's own requirements.

## Commands

All primary tasks go through `cargo xtask`:

```bash
cargo xtask build                     # Build (dev mode); compiles Rust + React UI
cargo xtask test                      # All tests (unit + integration)
cargo xtask test --unit               # Unit tests only
cargo xtask test --integration        # Integration tests only
cargo xtask test --update-snapshots   # Regenerate snapshot baselines
cargo xtask checks                    # fmt + clippy + typos + copyright headers
```

Run a single Rust test:
```bash
cargo test -p duvet -- test_name
```

For the React frontend in `duvet/www/`:
```bash
make          # Build bundle → public/script.js
npm start     # Dev server
npm test      # Jest tests
```

Checks enforce: `cargo fmt` (nightly), `cargo clippy -D warnings`, `typos-cli`, and copyright headers on all `.rs`/`.js` files.

## Architecture

### Workspace crates

- **`duvet`** — CLI binary (`init`, `extract`, `report` subcommands) + embedded React UI (`duvet/www/`)
- **`duvet-core`** — Shared utilities: VFS, caching, HTTP fetching, diagnostics, hashing
- **`duvet-macros`** — Proc-macro `#[query]` for memoized async queries used internally
- **`xtask`** — Build automation (not published)

### Data flow

1. **Comment parser** (`duvet/src/comment/`) scans source files for annotation patterns like `//= spec_id#section` and `//# requirement text`
2. **Specification parser** (`duvet/src/specification/`) parses RFC/IETF or Markdown specs into structured requirements with RFC 2119 levels (MUST, SHALL, SHOULD, MAY)
3. **Annotation set** (`duvet/src/annotation.rs`) links parsed comments to spec requirements
4. **Report generator** (`duvet/src/report/`) produces HTML, JSON, LCOV, CI, or snapshot output

### Integration tests

Tests live in `integration/*.toml` — each TOML file describes a test scenario (often downloading a real open-source project and running duvet against it). Run with `cargo xtask test --integration`. Snapshots are stored alongside configs and updated with `--update-snapshots`.

### Key configuration

Project-level config is in `.duvet/config.toml` (TOML, schema v0.4.0). It specifies which source globs to scan and which report formats to emit.

## Toolchain

Pinned in `rust-toolchain` (currently 1.88). Formatting requires nightly (`rustup toolchain install nightly`). Git LFS is used for large binary fixtures.
