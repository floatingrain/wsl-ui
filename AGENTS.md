# WSL UI — Agent Guide

This document contains essential context for AI coding agents working on the WSL UI codebase. WSL UI is a lightweight desktop application for managing Windows Subsystem for Linux (WSL) distributions.

## Project Overview

- **Name**: WSL UI
- **Developer**: Octasoft Ltd
- **Repository**: https://github.com/octasoft-ltd/wsl-ui
- **Website**: https://wsl-ui.octasoft.co.uk
- **Frontend License**: GPL-3.0-only
- **Backend License**: BUSL-1.1 (see `src-tauri/Cargo.toml`)

WSL UI is a Tauri 2.x desktop application with a React 19 + TypeScript frontend and a Rust backend. It provides a visual interface for listing, starting, stopping, installing, exporting, importing, and configuring WSL distributions, as well as managing `.wslconfig` / `wsl.conf` settings, custom actions, disk mounts, and RDP connections.

**This is a Windows-only application.** It must be built and run on Windows (not inside WSL) because it relies on the `wsl.exe` CLI, Windows Registry access, and Windows-specific APIs.

## Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Desktop framework | Tauri 2.x | Native window, system tray, OS integration |
| Frontend | React 19 + Vite 7 + TypeScript 5.9 | UI components and build tooling |
| Styling | Tailwind CSS 4.x | Utility-first CSS with custom theme-aware CSS variables |
| State management | Zustand 5 | Lightweight global state |
| i18n | i18next + react-i18next | 15 supported languages (10 namespaces each) |
| Backend | Rust (edition 2021) | WSL command execution, file I/O, registry access |
| Shared crate | `wsl-core` | WSL output parsing logic, separate from Tauri for testability |
| Unit tests | Vitest + jsdom + @testing-library/react | Frontend unit tests |
| E2E tests | WebDriverIO + tauri-driver + Edge WebDriver | End-to-end UI tests |
| Rust tests | `cargo test` | Backend unit tests |
| Release automation | release-please | Conventional-commit based versioning |

## Project Structure

```
wsl-ui/
├── src/                          # React frontend
│   ├── components/               # UI components
│   │   ├── ui/                   # Reusable low-level components (Button, Input, Modal)
│   │   ├── settings/             # Settings page sub-components
│   │   └── icons/                # SVG icon components
│   ├── services/                 # Tauri API wrappers
│   │   ├── wslService.ts         # Main WSL command invocations
│   │   ├── actionsService.ts     # Custom actions API
│   │   ├── telemetryService.ts   # Aptabase telemetry (opt-in)
│   │   └── lxcCatalogService.ts  # LXC community catalog fetching
│   ├── store/                    # Zustand state stores
│   │   ├── distroStore.ts        # Distribution list and operations
│   │   ├── settingsStore.ts      # App settings persistence
│   │   ├── mountStore.ts         # Disk mount state
│   │   └── ...                   # Other focused stores (health, actions, etc.)
│   ├── hooks/                    # Custom React hooks
│   ├── types/                    # TypeScript type definitions
│   ├── utils/                    # Utility functions (errors, logger)
│   ├── constants/                # Shared constants
│   ├── themes/                   # Theme system (17 built-in + custom)
│   ├── i18n/                     # Internationalization
│   │   └── locales/              # JSON translation files per language
│   └── test/                     # Test files
│       ├── setup.ts              # Vitest setup (mocks i18next & Tauri APIs)
│       └── e2e/                  # WebDriverIO E2E tests
│           └── specs/            # E2E spec files
├── src-tauri/                    # Rust backend
│   ├── src/
│   │   ├── main.rs               # Tauri app setup, tray icon, event handlers
│   │   ├── commands.rs           # All #[tauri::command] handlers
│   │   ├── error.rs              # Unified AppError type (thiserror)
│   │   ├── settings.rs           # App settings read/write
│   │   ├── validation.rs         # Input validation (distro names, paths, URLs)
│   │   ├── actions.rs            # Custom actions logic
│   │   ├── distro_catalog.rs     # Distribution catalog management
│   │   ├── download.rs           # Download logic for rootfs images
│   │   ├── metadata.rs           # Distro metadata persistence
│   │   ├── oci/                  # OCI container image support
│   │   └── wsl/                  # WSL operations
│   │       ├── mod.rs            # Module re-exports
│   │       ├── service.rs        # WslService high-level operations
│   │       ├── core.rs           # Core WSL CLI operations
│   │       ├── executor/         # Anti-corruption layer
│   │       │   ├── mod.rs        # Global executor initialization
│   │       │   ├── wsl_command/  # WSL CLI executor (real + mock)
│   │       │   ├── terminal/     # Terminal/IDE launching (real + mock)
│   │       │   └── resource/     # Resource monitoring (real + mock)
│   │       ├── resources.rs      # Memory, CPU, disk usage monitoring
│   │       ├── install.rs        # Distribution installation
│   │       ├── import_export.rs  # Import/export operations
│   │       ├── info.rs           # WSL version and system info
│   │       └── types.rs          # WSL types and errors
│   └── tauri.conf.json           # Tauri configuration
├── crates/
│   └── wsl-core/                 # Shared Rust library
│       ├── src/lib.rs            # Re-exports parser and types
│       ├── src/parser.rs         # `wsl --list --verbose` output parsing
│       └── src/types.rs          # Distribution, DistroState, WslError
├── scripts/                      # Build and utility scripts
├── docs/                         # User-facing documentation
└── public/                       # Static assets
```

## Build and Development Commands

Prerequisites: Node.js 18+, Rust (latest stable), Windows (not WSL).

```bash
# Install dependencies
pnpm install

# Development mode (starts Vite dev server + Tauri app)
pnpm run tauri:dev

# Production build (NSIS + MSI installers)
pnpm run tauri:build

# Frontend unit tests (Vitest, watch mode)
pnpm run test

# Frontend unit tests (single run)
pnpm run test:run

# Frontend unit tests with coverage
pnpm run test:coverage

# Rust unit tests
(cd src-tauri && cargo test)
(cd crates/wsl-core && cargo test)

# E2E tests (requires debug build first)
pnpm run tauri:build:debug   # or: pnpm run tauri:build -- --debug --no-bundle
pnpm run test:e2e:dev        # Runs in mock mode (WSL_MOCK=1)

# Generate app icons from source
pnpm run generate:icons
```

### Vite Configuration Notes
- Vite dev server runs on fixed port `1420` (`strictPort: true`).
- Vite is configured to ignore `src-tauri/**` during file watching.
- Vitest is embedded in `vite.config.ts` (globals enabled, jsdom environment, setup file at `src/test/setup.ts`).

## Code Style Guidelines

### TypeScript / React (Frontend)
- **Strict mode is required.** `tsconfig.json` enables `strict`, `noUnusedLocals`, `noUnusedParameters`, and `noFallthroughCasesInSwitch`.
- Use **functional components with hooks** exclusively. No class components.
- Co-locate tests with source files: `ComponentName.test.tsx` next to `ComponentName.tsx`.
- Use Zustand for global state. Each store is a focused module in `src/store/` with its own `.test.ts` file.
- Use Tailwind CSS utility classes. Custom theme colors are accessed via `theme-*` prefixes (e.g., `bg-theme-bg-primary`, `text-theme-text-secondary`). These map to CSS custom properties managed by the theme system.
- Use `useTranslation("namespace")` for i18n. Cross-namespace references use the `"common:key"` syntax.
- Log debug/info messages using `src/utils/logger.ts` (wraps `console` with prefixes).

### Rust (Backend)
- Use `thiserror` for structured error types (`src-tauri/src/error.rs`).
- All Tauri commands are defined in `src-tauri/src/commands.rs` and registered in `main.rs` via `tauri::generate_handler!`.
- Heavy WSL operations must be wrapped in `tokio::task::spawn_blocking` to avoid blocking the async runtime.
- The executor pattern in `src-tauri/src/wsl/executor/` separates real system calls from mock implementations.
- Doc comments (`///`) are expected for public APIs.

### Theme System
- Themes are defined in `src/themes/themes.ts` as arrays of `Theme` objects.
- CSS custom properties (`--bg-primary`, `--text-primary`, etc.) are applied to `document.documentElement` by `ThemeProvider.tsx`.
- Tailwind v4 `@theme` block in `src/index.css` maps these custom properties to `theme-*` color utilities.
- The default theme is "obsidian". There are 17 built-in themes plus a fully customizable "custom" theme.

## Testing Instructions

### Unit Tests (Frontend)
- Run with Vitest. Environment is `jsdom`.
- Setup file (`src/test/setup.ts`) mocks:
  - `react-i18next` → returns actual English translations from JSON files
  - `@tauri-apps/api/core` → `invoke` is a `vi.fn()` mock
  - `@tauri-apps/plugin-dialog` → `save` and `open` are mocks
- When writing tests for components that use Tauri APIs, mock the specific invoke calls with `vi.mocked(invoke).mockResolvedValue(...)`.
- Tests for stores should mock `wslService` and other service methods.

### Rust Tests
- Located inline in `#[cfg(test)]` modules within source files.
- `wsl-core` crate tests focus on parsing edge cases (UTF-16 LE decoding, distro names with spaces, default markers, etc.).
- `src-tauri` tests cover version comparison, error conversions, and validation logic.

### E2E Tests
- Framework: WebDriverIO with `tauri-driver` (WebDriver protocol for Tauri apps).
- Browser: Microsoft Edge (via `msedgedriver.exe`). The `wdio.conf.ts` auto-downloads the correct driver version.
- **Mock mode**: E2E tests run with `WSL_MOCK=1`, which causes the Rust backend to use mock executors instead of calling real `wsl.exe`. This means E2E tests do not require WSL to be installed.
- Specs live in `src/test/e2e/specs/`.
- Screenshots on failure are saved to `test-results/screenshots/`.
- Video recording is supported via `RECORD_VIDEO=1` (used for demo generation).
- To run E2E locally:
  1. Build debug binary: `pnpm run tauri:build:debug`
  2. Run tests: `pnpm run test:e2e:dev`

### CI/CD
- `.github/workflows/e2e.yml`:
  - `unit-tests` job runs on Ubuntu (fast).
  - `rust-tests` job runs on Windows (required for `winreg`).
  - `e2e-tests` job is manual-only (`workflow_dispatch` with `run_e2e=true`).
  - `build` summary job gates PRs.
- `.github/workflows/release.yml`: Manual workflow to rebuild a tagged release for x64 and arm64, producing MSI, NSIS, MSIX, and portable EXE artifacts.
- `.github/workflows/release-please.yml`: Automated conventional-commit releases.

## Key Architecture Decisions

### Executor Pattern (Anti-Corruption Layer)
The `src-tauri/src/wsl/executor/` module abstracts all external system interactions:
- `wsl_command`: Interfacing with `wsl.exe`
- `terminal`: Launching Windows Terminal, File Explorer, IDEs
- `resource`: Reading system memory and CPU usage

Each submodule has `real.rs` and `mock.rs` implementations. At startup, `init_executors()` checks `utils::is_mock_mode()` (true when `WSL_MOCK=1` env var is set) and installs the appropriate implementations into global `OnceLock` singletons. This allows the entire app to run in mock mode for E2E testing without WSL installed.

### State Management
- Frontend uses Zustand with separate stores for each domain:
  - `distroStore`: Distribution list, start/stop/delete operations, error handling
  - `settingsStore`: App settings (persisted to disk via Tauri commands)
  - `mountStore`: VHD and physical disk mounts
  - `actionsStore`: Custom user-defined actions
  - `healthStore` / `preflightStore`: WSL health checks and readiness
- Stores can call into each other (e.g., `distroStore` calls `useNotificationStore`).

### Polling
- A centralized polling hook (`usePolling.ts`) coordinates background refreshes:
  - Distribution list: every 10 seconds
  - Resource stats (CPU/memory): every 10 seconds
  - Health check: every 30 seconds
  - Mounted disks: every 10 seconds
- Tray menu is refreshed asynchronously on mouse enter to avoid blocking startup.

### Error Handling
- Rust: `AppError` enum in `error.rs` covers WSL, validation, file, download, and config errors. Commands return `Result<T, String>` for Tauri compatibility.
- Frontend: `parseError()` utility classifies errors. Timeout errors are detected by keyword matching ("timed out", "timeout", "taking too long") and surfaced with a force-kill button in the UI.

### Single Instance Enforcement
- Uses `tauri-plugin-single-instance`. If the user launches the app while it is already running in the tray, the existing window is shown instead of spawning a new process.

## Security Considerations

- **Input validation** is performed on the Rust backend for all user-provided values: distro names, file paths, URLs, and WSL versions (`src-tauri/src/validation.rs`).
- **`hidden_command` utility**: Runs Windows console applications with `CREATE_NO_WINDOW` flag to prevent console window flashes.
- **CSP**: Set to `null` in `tauri.conf.json` because this is a local desktop app with no web content.
- **Telemetry**: Aptabase telemetry is strictly opt-in. The app shows a prompt on first launch. No data is collected unless the user explicitly accepts.
- **No elevated privileges**: The app runs as a standard user process. Operations requiring admin rights (e.g., mounting physical disks) are delegated to `wsl.exe` or Windows APIs which prompt for UAC elevation naturally.

## Localization

- 15 languages supported. English is bundled eagerly; all others are lazy-loaded.
- Each language has 10 namespace JSON files: `common`, `header`, `dashboard`, `dialogs`, `settings`, `actions`, `install`, `errors`, `help`, `statusbar`.
- RTL support is implemented for Arabic (`dir: "rtl"`).
- To add a language: create directory under `src/i18n/locales/`, copy English files, translate values (never keys), create `index.ts`, register in `src/i18n/index.ts`.

## Release Process

- Versions are managed by **release-please** using conventional commits.
- It synchronizes the version across:
  - `package.json`
  - `src-tauri/Cargo.toml`
  - `src-tauri/tauri.conf.json`
  - `crates/wsl-core/Cargo.toml`
- The `CHANGELOG.md` is auto-generated from commit messages.
- For emergency releases, use the manual `release.yml` workflow.

## Common Pitfalls

- **Do not build inside WSL.** The app needs Windows Registry access and `wsl.exe` integration.
- **Port 1420** is hardcoded for Vite dev server. If it is occupied, `tauri dev` will fail.
- **E2E tests require a debug build first.** The `wdio.conf.ts` looks for the binary in `src-tauri/target/debug/wsl-ui.exe`.
- **Edge WebDriver version must match** the installed Edge browser. The WDIO config auto-downloads it, but this requires PowerShell and internet access.
- **Mock mode** is not just for E2E tests; it can also be used for frontend development on machines without WSL by setting `WSL_MOCK=1`.
- **WSL feature detection**: The app probes for `--distribution-id` support (WSL >= 2.4.4) at runtime and caches the result. Older WSL versions fall back to `-d <name>`.
