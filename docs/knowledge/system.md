# System Overview

Pier is a desktop app built on **Tauri v2**: a Rust backend and a web frontend rendered in the system webview.

## Requirements

### Supported OS

| OS | Minimum | Webview |
|---|---|---|
| macOS | 10.15 (Tauri v2 minimum); primary dev target | WKWebView (system) |
| Windows | 10 or later | WebView2 (preinstalled on Windows 11) |
| Linux | Distro that ships `webkit2gtk-4.1` | WebKitGTK 4.1 |

Pier is developed on macOS first. Windows and Linux are untested until noted otherwise.

### Toolchain

| Tool | Version | Notes |
|---|---|---|
| Rust (`rustc`, `cargo`) | 1.77.2 or later (Tauri v2 minimum) | Install via `rustup`. Dev machine: 1.99.1 |
| Node.js | Current LTS or later | Dev machine: 26.12.1 |
| pnpm | 10.x | Dev machine: 10.11.1 |
| Tauri CLI | 2.x | `cargo install tauri-cli --version "^2"` or `@tauri-apps/cli` |
| Git | 2.5 or later (worktree support) | Pier shells out to or links against git. Dev machine: 2.47.1 |

Pin exact versions in `rust-toolchain.toml` and `package.json` (`engines`) once `apps/` has code, and update this table to match.

### Platform dependencies

macOS:

```sh
xcode-select --install
```

Linux (Debian/Ubuntu):

```sh
sudo apt install \
  libwebkit2gtk-4.1-dev \
  build-essential \
  curl wget file \
  libxdo-dev \
  libssl-dev \
  libayatana-appindicator3-dev \
  librsvg2-dev
```

Windows: Microsoft C++ Build Tools (Desktop development with C++) and WebView2.

### Run

```sh
cd apps
pnpm install
pnpm tauri dev
```

Rust only (from the repo root):

```sh
cargo check
cargo test
```

## Repo layout

```
pier/
├── Cargo.toml          # Rust workspace
├── apps/               # Tauri project root
│   ├── package.json
│   ├── components.json # shadcn config
│   ├── src/            # frontend (FSD, see AGENTS.md)
│   └── src-tauri/      # Rust crate `pier`, Tauri glue only
├── crates/
│   ├── pier-core/      # core logic, no Tauri dependency
│   └── pier-shared/    # types/utilities shared across crates
├── docs/
├── scripts/
└── tests/
```

## Architecture

```
┌──────────────────────────────┐
│ Frontend (webview)           │
│ React + Tailwind CSS         │
│ shadcn/ui (Base UI)          │
└──────────────┬───────────────┘
               │ Tauri IPC (invoke / events)
┌──────────────▼───────────────┐
│ Backend (Rust, Tauri v2)     │
│ commands, state, services    │
└──────────────┬───────────────┘
               │
        OS / filesystem / git
```

## Stack

| Layer | Technology |
|---|---|
| App shell | Tauri v2 |
| Frontend styling | Tailwind CSS |
| Frontend components | shadcn/ui on Base UI primitives |
| Backend | Rust |
| Frontend ↔ backend | Tauri commands (`invoke`) and events |

## Frontend

- Styling uses Tailwind CSS v4 (`@tailwindcss/vite`, no `tailwind.config`). Theme tokens live in `apps/src/index.css`.
- UI components come from shadcn/ui, using the **Base UI** variant (not Radix) as the primitive layer. Components are copied into the repo and owned by the project. They live in `apps/src/shared/ui` and are added with `pnpm dlx shadcn@latest add <name>` (config in `apps/components.json`). Import with the `@/` alias (maps to `apps/src`).
- The frontend holds no privileged logic. Anything touching the filesystem, git, or processes goes through a Tauri command.

## Backend

- Written in Rust, exposed to the frontend as Tauri commands.
- Owns all system access: filesystem, git operations, process spawning.
- Long-running work runs off the main thread and reports progress to the frontend through events.
- Errors are returned as typed, serializable values so the frontend can render them.

## Conventions

- Frontend and backend communicate only through the Tauri IPC boundary. Keep command names and payload types in sync on both sides.
- Add new UI with shadcn/ui components before writing custom ones.
- Specs for features live in `docs/spec/`; this file covers cross-cutting system knowledge only.
