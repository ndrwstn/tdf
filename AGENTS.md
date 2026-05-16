# AGENTS.md

## Quick facts

- This repo is a single Rust crate, not a Cargo workspace. Main binary: `src/main.rs` (`tdf`). `src/lib.rs` exists mainly to share code with benches.
- `ratatui/` and `ratatui-image/` are git submodules, but `Cargo.toml` uses pinned git dependencies by default. The local `path = ...` overrides are commented out.
- Toolchain is **nightly** (`rust-toolchain.toml`), edition **2024**, minimum Rust version **1.95**.

## Validation commands

- Match CI locally:
  1. `cargo clippy --locked -- -D warnings`
  2. `cargo test --locked`
  3. `cargo fmt -- --check`
  4. `cargo test --locked --benches -- adobe_example`
  5. `cargo build --locked --profile dev`
- Single unit test: `cargo test --locked skip::tests::iter_works`
- Run one benchmark target directly: `cargo bench --bench rendering`

## Build / runtime quirks

- README's baseline release build is `cargo build --release`, but CI's normal compile check is `cargo build --locked --profile dev`.
- There is a more aggressive release script at `scripts/build_most_optimized.sh`; it uses nightly-only `-Z build-std` flags and builds `--profile production` for the host target.
- Build docs say the usual missing system deps are `libfontconfig` and `clang`. Linux CI also installs `libgoogle-perftools-dev` / `google-perftools`.
- Optional features are `epub`, `cbz`, and `tracing`. `tracing` only gates `console-subscriber` setup in `main.rs`.

## Code layout

- `src/main.rs` owns CLI parsing (`xflags`), terminal setup/teardown, file watching/hot reload, and orchestration.
- `src/renderer.rs` renders document pages; `src/converter.rs` converts rendered pages for terminal protocols; `src/kitty.rs` handles image display/protocol actions; `src/tui.rs` owns layout/input/help UI.
- Bench assets live under `benches/*.pdf`; the `rendering` bench exercises the shared library API with those sample PDFs.

## Style constraints

- Rust formatting is intentionally non-default: tabs (`.editorconfig`), `hard_tabs = true`, `trailing_comma = "Never"`, and crate-grouped imports (`.rustfmt.toml`). Run `cargo fmt` rather than guessing.
- `Cargo.toml` enables a very large set of Clippy lints at `warn`; CI turns all warnings into errors via `cargo clippy -- -D warnings`.
