# llama.cpp + pi config

Local config for running LLMs with [llama.cpp](https://github.com/ggml-org/llama.cpp) and using them in [pi](https://github.com/badlogic/pi-mono) as a custom provider.

## Files

| File | Purpose |
| --- | --- |
| `preset.ini` | llama.cpp router-mode preset: one section per model profile (shared defaults in `[*]`, per-model overrides) |
| `models.json` | pi custom-provider config: exposes the local llama.cpp server to pi |

## Running the llama.cpp preset

Start the router server with the preset (models are auto-downloaded from Hugging Face on first run):

```sh
llama-server --models-preset ./preset.ini
```

Each section in `preset.ini` becomes an available model, served by its section name (e.g. `Qwen3.8-27B`, `Gemma-4-E4B`). The `hf = <repo>:<tag>` key in each section specifies which GGUF to fetch.

To run just one model without the router, use its `hf` value directly:

```sh
llama-server -hf unsloth/Qwen3.8-27B-GGUF:UD-Q4_K_M
```

CLI arguments override preset values, so you can also tweak a single option on the fly:

```sh
llama-server --models-preset ./preset.ini --temp 0.1
```

The server listens on `http://localhost:8080/v1` by default.

### Bind address and port

By default the server binds to `127.0.0.1` (local access only). Use `--port` to change the port and `--host` to bind to a different address:

```sh
# Different port
llama-server --models-preset ./preset.ini --port 9090

# Bind to all IPv4 interfaces (reachable from other machines on the network)
llama-server --models-preset ./preset.ini --host 0.0.0.0 --port 9090
```

The flags work the same in single-model mode (`-hf ...`).

> **Note:** if you change the port or bind address, update `baseUrl` in `models.json` to match (e.g. `http://192.168.1.10:9090/v1`). Binding to `0.0.0.0` exposes the server on your network — llama.cpp has no authentication by default, so protect it with `--api-key` or firewall rules on untrusted networks.

## Configuring pi

Link the model config into pi (a symlink works, so this repo stays the single source of truth):

```sh
ln -s ~/Projects/llama/models.json ~/.pi/agent/models.json
```

It registers a `llama-cpp` provider pointing at the local server (`http://localhost:8080/v1`, OpenAI-compatible API). Model `id`s must match the preset section names, so keep both files in sync when adding/removing models.

Then start pi and select a model, e.g. `/model llama-cpp/Qwen3.8-27B`.
