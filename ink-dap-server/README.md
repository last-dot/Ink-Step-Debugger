# Ink! v6 Debug Adapter

Step-by-step debugger for Ink! v6 smart contracts, implementing the Debug Adapter Protocol (DAP) for VS Code integration.

## Overview

This Rust binary acts as a bridge between:
- VS Code Debug UI (via DAP protocol over stdio)
- Rust-based PolkaVM execution environment

## Architecture

```
VS Code <--(DAP/stdio)--> Rust DAP Server <--(HTTP :9229)--> ink-debug-rpc (sandbox)
```

## Development Setup

Build the release binary:

```bash
cargo build --release
```

The binary is picked up automatically by the VS Code extension from `target/release/ink-dap-server`.

## Project Structure

```
ink-dap-server/
├── src/
│   ├── main.rs             # Entry point, DAP request loop
│   ├── command_handler.rs  # DAP command routing and handlers
│   ├── service.rs          # HTTP endpoints (/log, /pause)
│   ├── state.rs            # DAP session state
│   ├── types.rs            # Shared types
│   ├── utils.rs            # Helpers
│   └── log.rs              # Async log channel
└── Cargo.toml
```

## Tasks

- [x] Implement DAP handshake with VS Code extension
- [x] Implement HTTP log endpoint for sandbox step events
- [x] Implement launch / threads / stack trace / disconnect handlers
- [x] Implement breakpoint line storage
- [ ] Implement execution pause on breakpoint hit (M2)
- [ ] Implement stepping (M2)

## Team

- Rust DAP Server: Maliketh
- VS Code Extension: TBD
