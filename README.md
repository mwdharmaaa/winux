# Winux

Modular userland extension layer and tiling orchestrator transforming Windows NT into a Unix-like experience.

---

## 1. Concept & Architectural Vision

Traditional Windows desktop personalization relies on heavy electron widgets or fragile surface-level theme patching. `winux` approaches the problem from an operating systems perspective: treating Windows NT as a capable kernel host while replacing its floating window paradigm and fragmented CLI workflow with a decoupled, high-performance Unix userland.

Rather than attempting full kernel emulation (handled natively by WSL 2), `winux` acts as an event-driven userland extension layer hooking into Win32 subsystem events, organizing window geometry deterministically, and exposing control via standard POSIX-friendly pipelines.

```
+-------------------------------------------------------------------+
|                        User Applications                          |
|             (Terminals, Editors, Browsers, Native GUI)             |
+-------------------------------------------------------------------+
                                  ^
                                  | Win32 Window Positioning (SetWindowPos)
+---------------------------------+---------------------------------+
|                         Winux Daemon                              |
|  +--------------------+  +-------------------+  +---------------+ |
|  | Event Hook Listener|  | Layout Engine     |  | IPC Server    | |
|  | (SetWinEventHook)  |  | (BSP Tree Matrix) |  | (Named Pipes) | |
|  +--------------------+  +-------------------+  +---------------+ |
+---------------------------------+---------------------------------+
                                  ^
                                  | Instructions & State Queries
+---------------------------------+---------------------------------+
|                      CLI Tooling (winuxctl)                       |
|           Config (TOML/KDL) | Hotkey Daemon | Status Bar          |
+-------------------------------------------------------------------+
                                  |
+---------------------------------+---------------------------------+
|                        Windows NT Kernel                          |
|             User32 / DWM / GDI / Win32 Subsystems                 |
+-------------------------------------------------------------------+
```

---

## 2. Core Pillars

### 2.1 Event-Driven Window Management (The Composable Compositor)
* Subscribes to OS-wide window lifecycle events (`EVENT_OBJECT_CREATE`, `EVENT_OBJECT_DESTROY`, `EVENT_SYSTEM_FOREGROUND`) using out-of-context Win32 event hooks.
* Eliminates floating clutter by calculating viewport subdivisions via a deterministic Binary Space Partitioning (BSP) tree.
* Manages multi-monitor geometry with per-workspace isolation, zero screen tearing, and sub-millisecond layout reflows.

### 2.2 Client-Daemon IPC Architecture
* The core runtime runs silently in the background as an unprivileged daemon process.
* Exposes a fast local IPC interface over Windows Named Pipes (`\\.\pipe\winux-ipc`).
* Allows external tools, shells, and hotkey managers (e.g., `whkd`, AutoHotkey, or native Rust keybinders) to inspect and alter workspace state via simple JSON or binary payloads.

### 2.3 Unix-Grade CLI Orchestration (`winuxctl`)
* Provides a declarative CLI companion to query workspace status, move active windows, toggle floating modes, and switch workspaces.
* Emits scriptable output formats (JSON/plain text) compatible with Unix pipelines (`jq`, `grep`, `awk`).
* Integrates directly into modern terminal environments (PowerShell 7, Nushell, Zsh via MSYS2/WSL).

### 2.4 Declarative, Version-Controlled Configuration
* Single configuration file (`~/.config/winux/config.toml` or `config.kdl`).
* Hot-reloading on file modification without restarting the background daemon.
* Rule-based application filtering (floating exceptions, title matching, process name regex).

---

## 3. Planned Project Structure

Following single-responsibility atomic design and modular separation:

```text
winux/
|-- daemon/
|   |-- src/
|   |   |-- hooks/           # Win32 event listeners (SetWinEventHook)
|   |   |-- layout/          # Binary Space Partitioning (BSP) matrix algorithms
|   |   |-- monitor/         # Display geometry and multi-monitor detection
|   |   |-- workspace/       # Virtual workspace state and navigation
|   |   `-- main.rs          # Daemon entrypoint and event loop
|-- cli/
|   |-- src/
|   |   |-- commands/        # Action handlers (focus, swap, toggle, query)
|   |   `-- main.rs          # winuxctl CLI binary entrypoint
|-- ipc/
|   |-- src/
|   |   |-- protocol.rs      # Named pipe wire protocol, request/response DTOs
|   |   `-- client.rs        # IPC connection client
|-- config/
|   |-- src/
|   |   |-- schema.rs        # Configuration structs and validation
|   |   `-- loader.rs        # File watcher and TOML parser
|-- assets/                  # Diagrams, schemas, and assets
`-- README.md
```

---

## 4. Roadmap & Implementation Phases

* **Phase 1: Win32 Event Daemon**: Minimal C++ or Rust background process capturing window creation, focus changes, and termination without memory leaks.
* **Phase 2: BSP Layout Matrix**: Monolithic layout engine calculating tile coordinates across primary and secondary monitors.
* **Phase 3: Named Pipe IPC**: Client-server contract for external control via `winuxctl`.
* **Phase 4: Rule Engine & Workspaces**: Virtual desktop support, process blacklist/whitelist, and auto-tiling rules.
* **Phase 5: Status Bar & Launcher Integration**: Native integration hooks for Zebar, Yasb, Flow Launcher, and Polybar-like status renderers.

---

## 5. Development & Contribution

### Prerequisites
* Windows 10/11 (Build 19041 or higher)
* Rust toolchain (stable, `x86_64-pc-windows-msvc`) or MSVC C++20 compiler
* Windows SDK (User32, Shell32, DwmApi headers)

### Building
```powershell
# Clone the repository
git clone https://github.com/mwdharmaaa/winux.git
cd winux

# Verify toolchain
cargo check
```

---

## 6. License

Dual-licensed under MIT or Apache-2.0 at your option.
