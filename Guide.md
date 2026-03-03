# Step-Debugger for Ink! v6 - Testing Guide

This guide describes how to **manually test** the `ink-trace-extension` end-to-end inside VSCode.
The project is built on Ink! v6 — make sure to use cargo darkly for all builds and runs to ensure compatibility.

---

## Architecture

### Current Architecture (Milestone 1)

```
+-------------------------------+
|  VS Code Extension            |
|  ink-trace-extension          |
|  launches DAP server binary   |
+-------------------------------+
            | DebugAdapterExecutable (stdio)
            v
+-------------------------------+
|  Rust DAP Server              |
|  ink-dap-server               |
|  handles DAP protocol         |
+-------------------------------+
            | HTTP POST to localhost:9229
            v
+-------------------------------+
|  HTTP Bridge                  |
|  ink-debug-rpc                |  <-- temporary, PoC only
|  embedded in the sandbox      |
+-------------------------------+
            | step callbacks / PC logging
            v
+-------------------------------+
|  Sandbox                      |
|  ink-debugger                 |
|  patched DRink! +             |
|  pallet-revive                |
+-------------------------------+
            | executes
            v
+-------------------------------+
|  PolkaVM interpreter          |
|  contract execution           |
+-------------------------------+
```

`ink-debug-rpc` is a temporary HTTP bridge introduced during Milestone 1 to decouple the sandbox from the DAP server during the proof-of-concept phase. It will be removed in the final architecture.

### Final Architecture (planned)

```
+-------------------------------+
|  VS Code Extension            |
|  ink-trace-extension          |
|  ships DAP server binary      |
+-------------------------------+
            | DebugAdapterExecutable (stdio)
            v
+-------------------------------+
|  Rust DAP Server              |
|  ink-dap-server               |
|  bundled inside extension     |
+-------------------------------+
            | direct Rust crate dependency
            v
+-------------------------------+
|  Sandbox                      |
|  DRink! + pallet-revive       |
|  step tracing hooks           |
+-------------------------------+
            | executes
            v
+-------------------------------+
|  PolkaVM interpreter          |
|  contract execution           |
+-------------------------------+
```

Key differences from the current state:
- `ink-debug-rpc` and the HTTP layer are removed. The DAP server depends on the sandbox directly as a Rust crate.
- The DAP server binary is bundled inside the extension package. Zero setup for the end user.

---

## 1. Build the DAP Server

The extension launches the `ink-dap-server` binary automatically, so it must be built before running the extension.

1. Open a terminal and **navigate to**: `ink-dap-server`
2. Build the release binary:
   ```bash
   cargo build --release
   ```
3. The binary will be placed at `ink-dap-server/target/release/ink-dap-server`.
   The extension picks it up from this location automatically.


## 2. Launch the Extension

1. Open VSCode and **open the `ink-trace-extension/` folder**.
2. Run command `npm install`
3. Run command `npm run compile`
4. Press `F5` or run **"Launch Extension"** from the Run menu. This starts the extension and opens a **new VSCode window** (Extension Host).


## 3. Open the Sample Workspace

1. In the **Extension Host window**, open: `ink-trace-extension/sampleWorkspace`
2. Wait until the workspace finishes building.
You should see the **"Run Test | Debug"** buttons above each DRink test in `lib.rs`.


## 4. Run a DRink Test

1. Press `F5` or run **"Launch"** from the Run menu. This starts the debug session — you'll see confirmation in the debug console that the **Rust DAP server** is running and responding. Since breakpoints and step functionality are not yet implemented, you can stop the session at this point.
2. Click **"Run Test"** on any test (or press Run button from the Run menu).
Wait for the build in the terminal to complete.
3. This triggers the **DRink test pipeline**:
- **Macro execution** - starts the DRink flow.
- **`cargo-contract build`** - compiles all contract blobs.
- **Contract execution** - runs inside our custom environment built on top of **DRink**.


## 5. Observe Logs

1. Open the **Debug Console** in VSCode.
2. You should see logs with `[ink_debug_rpc::sandbox_rpc]` prefix.
3. These logs come from our **custom sandbox RPC** — each log line corresponds to a **program counter (step)** in contract execution.

**Success Criteria**
- `ink-dap-server` binary is built successfully.
- Extension launches correctly.
- Sample workspace builds.
- DRink test runs end-to-end.
- Console shows `[ink_debug_rpc::sandbox_rpc]` logs with step info.
