# webtor-rs

## Project Overview
`webtor-rs` is a Rust-based implementation of a Tor client designed to run in web browsers via WebAssembly (WASM). It aims to provide anonymous HTTP/HTTPS requests through the Tor network using Snowflake bridges, offering a privacy-focused alternative to native Tor clients within a web context.

The project is a rewrite of the TypeScript-based `tor-hazae41` and is structured as a Rust workspace with three main components:
*   **`webtor`**: The core Rust library implementing Tor client logic, circuit management, and relay selection.
*   **`webtor-wasm`**: WebAssembly bindings that expose the core functionality to JavaScript/TypeScript.
*   **`webtor-demo`**: A demonstration web application showcasing the client's capabilities.

## Architecture & Key Components

### `webtor/` (Core Library)
*   **`client.rs`**: Main entry point (`TorClient`) managing circuit lifecycles and requests.
*   **`circuit.rs`**: Handles creation, maintenance, and graceful updates of Tor circuits. Now fully integrated with `arti` (using `ClientTunnel`).
*   **`snowflake.rs`**: Implements WebSocket-based communication with Snowflake bridges.
*   **`http.rs`**: Provides an HTTP client interface for making requests through Tor circuits.
*   **`relay.rs`**: Logic for selecting guard, middle, and exit relays. Supports `arti`'s `CircTarget` trait.

### `webtor-wasm/` (WASM Bindings)
*   Exposes a Promise-based JavaScript API.
*   Handles memory management between Rust and JS.
*   Provides TypeScript definitions.

### `webtor-demo/` (Demo App)
*   A static web application to test and demonstrate the client.
*   Located in `src/` (Rust logic) and `static/` (HTML/assets).

## Development & Usage

### Prerequisites
*   **Rust**: Stable toolchain (`rustup`).
*   **wasm-pack**: For building WASM modules (`curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh`).
*   **Node.js**: Optional, for development server and convenience scripts.

### Building
The project uses a shell script and `package.json` for build orchestration.

*   **Full Build:**
    ```bash
    ./build.sh
    # OR
    npm run build
    ```
*   **WASM Only:**
    ```bash
    npm run build:wasm
    ```

### Running the Demo
After building, the demo can be served locally:

```bash
npm run serve
# OR manually:
cd webtor-demo/static && python3 -m http.server 8000
```
Access at: `http://localhost:8000`

### Testing
*   **Run All Tests:**
    ```bash
    cargo test --workspace
    ```
    *Note: Some tests requiring browser APIs may fail in a native environment.*

## Conventions
*   **Language**: Rust (2021 edition).
*   **WASM Tooling**: `wasm-pack` with `wasm-bindgen`.
*   **Async Runtime**: `tokio` (for native/testing) and `wasm-bindgen-futures` (for web).
*   **Error Handling**: Uses `thiserror` and `anyhow`.
*   **Logging**: `tracing` ecosystem.

## Recent Changes
*   **Tor Circuit Creation**: Implemented full circuit creation using `arti`'s `ClientTunnel` API.
    *   Updated `webtor/src/circuit.rs` to use `tor_proto::channel::Channel::new_tunnel` and `PendingClientTunnel::create_firsthop_fast`.
    *   Updated `webtor/src/relay.rs` to implement `OwnedCircTarget` construction for relays.
    *   Added `make_circ_params` helper to construct default circuit parameters.
    *   Updated `Circuit` struct to hold `ClientTunnel`.
    *   Added dependencies: `tor-protover`, `tor-units`, `rand` to `webtor` and root workspace.
    *   **PR Review Fixes**:
        *   Addressed issue where bridge/first hop was missing from circuit relay list.
        *   Added error logging for circuit reactor task.
        *   Fixed one-time fetch to ensure channel establishment.
        *   Renamed `create_initial_circuit` to `establish_channel` for clarity.
        *   **Path Selection Fix**:
            *   Updated `RelayCriteria` to support excluding specific relay fingerprints.
            *   Updated `CircuitManager::create_circuit` to enforce distinct relays for each hop (Bridge != Middle != Exit).
            *   Updated `RelayManager::select_relay` to randomize selection among top candidates instead of being deterministic.
*   **Channel Integration**: Passed shared `Channel` from `TorClient` to `CircuitManager` to enable circuit creation on the established Snowflake channel.