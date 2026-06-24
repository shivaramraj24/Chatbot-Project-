# Chatbot Project (DS210 Mini Project 2)

A multi-version LLM-powered chatbot built in Rust using [kalosm](https://github.com/floneum/floneum/tree/main/interfaces/kalosm) and [Rocket](https://rocket.rs/). The project runs a locally hosted Llama model behind a web server, with each version building on the last — starting from a stateless single-turn bot and ending with per-user persistent session storage.

## Overview

The idea behind this project was to iteratively build up a chatbot from scratch, adding functionality version by version rather than jumping straight to the final product. Each version (`v1` through `v5`) adds one meaningful layer of complexity, making it easier to understand what each piece actually contributes.

The backend is Rocket, handling HTTP routing and JSON serialization. The LLM inference runs locally via kalosm's `Llama` wrapper. The frontend is a plain HTML page (`index.html`) that talks to the Rocket endpoints.

## Versions

**v1 — Basic single-turn chat**
Wraps the Llama model in a `ChatbotV1` struct. Each call to `chat_with_user` opens a fresh chat session with a system prompt ("act like a drunk comedian") and sends the message. No memory between turns.

**v2 — Stateful conversation within a session**
Upgrades to a persistent `Chat<Llama>` session stored inside the struct, so the model can reference earlier parts of the conversation within a single run.

**v3 — Multi-user support**
Introduces a user identifier so multiple users can each have their own isolated chat session. Sessions are tracked by username rather than sharing one global context.

**v4 — File-based session persistence**
Adds disk-backed sessions via `file_library`. Each user's chat history is saved to a `.txt` file (e.g. `alice.txt`) and reloaded on future visits using `fixed_load_session`. Also exposes a `get_history` method to retrieve past messages. The pirate persona makes a cameo here.

**v5 — Final version / endpoint integration**
Ties everything together with the full Rocket endpoint setup and CORS configuration. The webserver in `webserver.rs` routes incoming requests to the correct chatbot version.

## Project Structure

```
├── main.rs          # Rocket launch entrypoint
├── lib.rs           # Crate root, module declarations
├── mod.rs           # Module organization
├── webserver.rs     # Rocket server setup and CORS config
├── endpoints.rs     # HTTP route handlers
├── adapter.rs       # Adapter layer between endpoints and chatbot logic
├── llama_model.rs   # Model loading / initialization
├── file_library.rs  # File I/O for saving and loading chat sessions
├── experiment.rs    # Scratch file for testing
├── v1.rs            # Chatbot version 1
├── v2.rs            # Chatbot version 2
├── v3.rs            # Chatbot version 3
├── v4.rs            # Chatbot version 4 (file persistence)
├── v5.rs            # Chatbot version 5 (final)
├── index.html       # Frontend UI
├── Cargo.toml
└── Cargo.lock
```

## Dependencies

- [`kalosm`](https://crates.io/crates/kalosm) `0.4.0` — local LLM inference with Llama support
- [`rocket`](https://crates.io/crates/rocket) `0.5.1` — async web framework
- [`rocket_cors`](https://crates.io/crates/rocket_cors) `0.6.0` — CORS middleware for Rocket
- [`serde`](https://crates.io/crates/serde) / [`serde_json`](https://crates.io/crates/serde_json) — JSON serialization for request/response types
- [`tokio`](https://crates.io/crates/tokio) — async runtime (dev dependency for tests)

## Running

```bash
cargo build
cargo run
```

Rocket will start on `http://localhost:8000` by default. Open `index.html` in a browser or point requests at the exposed endpoints.

> **Note:** kalosm will download the Llama model weights on first run. This takes a few minutes and requires a decent amount of disk space (~4GB depending on the model variant). Subsequent runs use the cached weights.

## Notes

- The `fix` crate (local path dependency) provides `fixed_load_session`, a helper that handles some quirks with deserializing kalosm's session format.
- Each version is a self-contained struct — the endpoint layer picks which version to use at runtime via the adapter.
- This was built as part of BU's DS210 (Programming for Data Science) course. The versioned structure follows the assignment spec, which asked us to incrementally implement each chatbot capability and understand what changes between versions rather than just submitting a finished product.
