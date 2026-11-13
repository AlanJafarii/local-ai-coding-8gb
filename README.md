<div align="center">

# 🚀 Free Local AI Coding Setup

### Turn VS Code into a private, offline, Cursor-style AI editor — optimized for 8GB VRAM

**Ollama** · **Qwen 2.5 Coder** · **Continue.dev**

![Ollama](https://img.shields.io/badge/Ollama-local%20LLM-black?style=for-the-badge)
![Qwen](https://img.shields.io/badge/Qwen2.5--Coder-14B%20%7C%201.5B-6f42c1?style=for-the-badge)
![VS Code](https://img.shields.io/badge/VS%20Code-Continue.dev-007ACC?style=for-the-badge&logo=visualstudiocode&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)
![Cost](https://img.shields.io/badge/cost-%240%2Fmonth-brightgreen?style=for-the-badge)

</div>

---

## 📖 Overview

Stop paying $20/month for Cursor or GitHub Copilot. This repository contains the exact configuration, a custom `Modelfile`, and VRAM optimization tricks to turn **VS Code** into a **private, offline, and fast AI code editor**.

Your code never leaves your machine.

### ✨ Features

- 💸 **100% free** — no subscriptions, no API keys
- 🔒 **Private & offline** — everything runs locally on your GPU
- ⚡ **Fast Tab autocomplete** with a lightweight 1.5B model
- 💬 **Chat & refactoring** with a 14B coding model
- 🔎 **`@codebase` search** via local embeddings (RAG)
- 🧠 **Expanded 8K context window** (fixes Ollama's default 2048-token trap)
- 🎛️ **VRAM optimizations**: Flash Attention + 8-bit KV cache

---

## 📑 Table of Contents

1. [Hardware Cheat Sheet](#-hardware-cheat-sheet)
2. [Prerequisites](#-prerequisites)
3. [Installation](#️-installation)
4. [Keyboard Shortcuts](#️-keyboard-shortcuts)
5. [Repository Structure](#-repository-structure)
6. [Troubleshooting](#-troubleshooting)
7. [FAQ](#-faq)
8. [Contributing](#-contributing)
9. [License](#-license)

---

## 📊 Hardware Cheat Sheet

The secret to high speed on local GPUs is picking the right **4-bit quantization (`q4_K_S` or `q4_K_M`)**, so the model weights plus the KV context cache fit inside your VRAM without spilling into slow system RAM.

| Your GPU VRAM | Chat / Refactor Model | Tab-Autocomplete Model | Recommended Context (`num_ctx`) |
| :--- | :--- | :--- | :--- |
| **4GB – 6GB** | `qwen2.5-coder:7b-instruct-q4_K_S` | `qwen2.5-coder:1.5b` | `4096` |
| **8GB (Sweet Spot)** | `qwen2.5-coder:14b-instruct-q4_K_S` | `qwen2.5-coder:1.5b` | `8192` |
| **12GB – 16GB** | `qwen2.5-coder:14b-instruct-q5_K_M` | `qwen2.5-coder:3b` | `16384` |
| **24GB+ (RTX 3090/4090)** | `qwen2.5-coder:32b-instruct-q4_K_M` | `qwen2.5-coder:7b` | `32768` |

> **ℹ️ Note for 8GB cards:** the 14B Q4 model is slightly larger than 8GB, so Ollama will offload a few layers to system RAM. It still works well, but if you want 100% GPU speed, use `qwen2.5-coder:7b-instruct-q4_K_M` and the same steps below. Check with `ollama ps` (the `PROCESSOR` column shows the GPU/CPU split).

---

## ✅ Prerequisites

- A GPU with at least 4GB VRAM (NVIDIA recommended; Apple Silicon also works)
- [Ollama](https://ollama.com/) installed
- [Visual Studio Code](https://code.visualstudio.com/)
- ~15GB of free disk space (for the 8GB tier)

---

## 🛠️ Installation

### Step 1 — Install Ollama & pull the models

1. Download and install [Ollama](https://ollama.com/).
2. Open a terminal and pull the models (8GB GPU setup):

```bash
# 1. 14B Q4_K_S model for heavy coding & refactoring
ollama pull qwen2.5-coder:14b-instruct-q4_K_S

# 2. Lightweight 1.5B model for instant Tab autocomplete
ollama pull qwen2.5-coder:1.5b

# 3. Nomic Embed for local codebase RAG (@codebase search)
ollama pull nomic-embed-text
```

### Step 2 — Fix the context window trap (Modelfile)

By default, Ollama limits models to a **2048-token** context window, which makes the AI forget long files. Let's expand it to **8192 tokens**.

1. Use the [`Modelfile`](./Modelfile) from this repository (or create one in your current directory):

```dockerfile
FROM qwen2.5-coder:14b-instruct-q4_K_S

# Expand context window from default 2048 to 8192 tokens for large files
PARAMETER num_ctx 8192

# Lower temperature for more deterministic, bug-free code generation
PARAMETER temperature 0.2

# Prevent repetitive loops
PARAMETER repeat_penalty 1.05

# Custom system prompt optimized for clean software engineering
SYSTEM """
You are an expert senior software engineer.
Write clean, modular, well-documented, and production-ready code.
Skip unnecessary conversational filler and go straight to the code and technical explanation.
"""
```

2. Build your custom 8K-context model:

```bash
ollama create qwen-coder-8k -f Modelfile
```

3. Verify it was created:

```bash
ollama list
```

You should see `qwen-coder-8k:latest` in the list.

### Step 3 — Pro VRAM optimization *(optional but recommended)*

To squeeze more context into an 8GB GPU, enable **Flash Attention** and **8-bit KV cache quantization** with these environment variables.

**Windows (PowerShell as Admin):**

```powershell
[System.Environment]::SetEnvironmentVariable('OLLAMA_FLASH_ATTENTION', '1', 'User')
[System.Environment]::SetEnvironmentVariable('OLLAMA_KV_CACHE_TYPE', 'q8_0', 'User')
```

Then restart Ollama from the system tray.

**Linux / macOS:**

```bash
export OLLAMA_FLASH_ATTENTION=1
export OLLAMA_KV_CACHE_TYPE="q8_0"
```

> Add these lines to your `~/.bashrc` or `~/.zshrc` to make them permanent. If Ollama runs as a systemd service, set them in the service environment instead.

### Step 4 — Configure Continue.dev in VS Code

1. Open VS Code → Extensions (`Ctrl+Shift+X`) → install **[Continue](https://continue.dev)**.
2. Click the **⚙️ gear icon** at the bottom right of the Continue sidebar to open the config.
3. Replace its contents with the [`config.json`](./config.json) from this repository and save:

```json
{
  "models": [
    {
      "title": "Qwen Coder 14B (8K)",
      "provider": "ollama",
      "model": "qwen-coder-8k"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen Coder 1.5B (Autocomplete)",
    "provider": "ollama",
    "model": "qwen2.5-coder:1.5b"
  },
  "embeddingsProvider": {
    "provider": "ollama",
    "model": "nomic-embed-text"
  }
}
```

> **Note:** newer versions of Continue may use `config.yaml` instead of `config.json`. If your extension opens a YAML file, use the same three models (chat, autocomplete, embeddings) in that format.

🎉 **Done!** Open any project and start coding with AI — fully offline.

---

## ⌨️ Keyboard Shortcuts

| Shortcut | Action |
| :--- | :--- |
| `Ctrl + L` (`Cmd + L`) | Open AI chat / add selected code to chat context |
| `Ctrl + I` (`Cmd + I`) | Inline code editing & refactoring (Cursor-style) |
| `@codebase` | Type in chat to search across your entire project locally |
| `Tab` | Accept inline autocomplete suggestion |

---

## 📁 Repository Structure

```text
.
├── README.md      # This guide
├── Modelfile      # Custom Ollama model (8K context, tuned parameters)
└── config.json    # Continue.dev configuration
```

---

## 🩺 Troubleshooting

| Problem | Solution |
| :--- | :--- |
| Model is slow / uses CPU | Run `ollama ps`. If it's not 100% GPU, use a smaller model or lower `num_ctx`. |
| Out-of-memory errors | Lower `num_ctx` (e.g. `4096`) or switch to the 7B model. |
| AI forgets earlier code | Make sure Continue uses `qwen-coder-8k`, not the base model. |
| Autocomplete not showing | Confirm `qwen2.5-coder:1.5b` is pulled and Ollama is running. |
| `@codebase` returns nothing | Confirm `nomic-embed-text` is pulled, then re-index from Continue. |
| Env variables not applied | Fully restart Ollama (quit from tray, reopen). |

---

## ❓ FAQ

**Is it really free?**
Yes. All models are open-weight and run on your own hardware. The only cost is electricity.

**Does my code leave my computer?**
No. Ollama and Continue run locally, so it works offline after the models are downloaded.

**Can I use a different model?**
Yes. Change the `FROM` line in the `Modelfile` and the model names in `config.json`.

**Is it as good as Cursor or Copilot?**
For everyday coding, refactoring, and explaining code it's very capable, but large cloud models can still be stronger on complex, multi-file tasks. The trade-off is privacy and zero cost.

---

## 🤝 Contributing

Contributions are welcome!

1. Fork the repository
2. Create a branch: `git checkout -b feature/my-improvement`
3. Commit your changes: `git commit -m "Add my improvement"`
4. Push and open a Pull Request

Found an issue or have a hardware config that works well? Please open an [issue](../../issues).

---

## 📜 License

Distributed under the **MIT License**. See `LICENSE` for details.

---

## ⭐ Support the Project

If this setup saved you **$240/year** on AI subscriptions, please give this repository a **Star ⭐** and subscribe to the **YouTube channel** for more local AI engineering & computer vision tutorials!

<div align="center">

**Made with ❤️ for developers who value privacy and speed.**

</div>
