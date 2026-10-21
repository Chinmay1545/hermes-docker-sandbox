# Local AI Agent in a Docker Sandbox

A fully local AI agent that runs inside a Docker container. [Hermes Agent](https://hermes-agent.nousresearch.com) runs in the container with its own browser, terminal and file tools. The model (Gemma 4 12B) runs on my Mac's GPU through Ollama. No cloud API or API key is used.

Built for the CSE485 "Growing a Competency" assignment at ASU (skill: containerization with Docker).

## How it works

```
Mac (host)                                Docker container
-------------------------------           -----------------------------------
Ollama + gemma4:12b-mlx                   Hermes Agent
runs on the Apple GPU (Metal)   <-----    browser, terminal, file tools
listening on port 11434         HTTP      reaches the Mac at
                                          host.docker.internal:11434

~/.hermes  <------------------- mounted as /opt/data (config, chats, output files)
```

- **Why the model is outside Docker:** Docker on a Mac runs inside a small Linux VM that can't use the Apple GPU. Running the model natively keeps it fast.
- **Why the agent is inside Docker:** the agent can run commands and browse the web. In a container, it can only touch the container and the one folder I share (`/opt/data`), not the rest of my Mac.

## Run it

1. Start Ollama so containers can reach it, with a bigger context window:
   ```sh
   OLLAMA_CONTEXT_LENGTH=65536 OLLAMA_HOST=0.0.0.0 ollama serve
   ```
2. Pull the model (optional: build the 64k version with the `Modelfile`):
   ```sh
   ollama pull gemma4:12b-mlx
   ollama create gemma4-12b-64k -f Modelfile
   ```
3. Check that a container can see the model:
   ```sh
   docker run --rm curlimages/curl http://host.docker.internal:11434/v1/models
   ```
4. Run the Hermes setup once, choosing **Custom endpoint**, `http://host.docker.internal:11434/v1`, **Chat Completions**, terminal backend **Local**, and browser **Local Browser**:
   ```sh
   docker run -it --rm -v ~/.hermes:/opt/data nousresearch/hermes-agent setup
   ```
   My resulting config is in [`hermes/config.yaml`](hermes/config.yaml) (comments removed). The key parts are the `model` section pointing at Ollama, `terminal.backend: "local"`, `browser.cloud_provider: local`, and `platform_toolsets.cli` with only the 5 tools I kept.
5. Start the agent:
   ```sh
   docker run -it --rm --shm-size=1g -v ~/.hermes:/opt/data nousresearch/hermes-agent
   ```
   Or the same thing with the compose file in this repo: `docker compose run --rm hermes`.
