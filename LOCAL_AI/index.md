---
layout: default
title: Local AI
---

# Local AI

- [Learning materials](#learning-materials)
- [Local AI assistants](#local-ai-assistants)
- [Complete flow](#complete-flow)
- [Vocabulary](#vocabulary)
- [Ollama + Codex](#ollama--codex)
- [Complete hands-on tutorial](#complete-hands-on-tutorial)

---

## Learning materials

- Cluster of 2 NVIDIA DGX Spark
- Laptop

---

## Local AI assistants

- [Ollama](https://ollama.com/library/gemma4)
  - **Metrics:**
    - Size — amount of RAM needed (e.g. `4b` = 4 billion parameters)
    - Context — how much the model can remember in a chat session
      (128k context ≈ 96 000 words)
    - Quantization — compression technique, trading off size against accuracy
- [Open WebUI](https://openwebui.com)
  - Ollama is a model provider, Open WebUI is the interface.
  - Can be run with `docker run`.
  - Use Tailscale to expose it widely.

---

## Complete flow

`docker run` Open WebUI, open `localhost:3000`, configure the URL of the Ollama server,
configure the CUDA toolkit (see the Local AI repo).

---

## Vocabulary

- **Models** — generate tokens.
- **Runtimes** — load models into memory (e.g. Ollama loading `gemma4`).
- **Harnesses** — Claude Code, Codex, Open Clauw.

---

## Ollama + Codex

```bash
ollama launch codex --model gemma4:26b-a4b-it-q4_K_M
```

Can be run on a remote machine (server): install Ollama on the server and access it via
URL + port number.

---

## Complete hands-on tutorial

Using only Ollama in a terminal on the local machine.

- **Temperature** — close to 0 is predictable, good for code; close to 1 is more
  creative, good for brainstorming.

```bash
ollama create toto -f Modelfile   # set personality, temperature, context length
```

[Back to home]({{ site.baseurl }}/)
