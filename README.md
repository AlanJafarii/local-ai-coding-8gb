# 🚀 100% Free Local AI Coding Setup (Optimized for 8GB VRAM)

Stop paying $20/month for Cursor or GitHub Copilot. This repository contains the exact configuration, custom `Modelfile`, and VRAM optimization tricks to turn **VS Code** into a private, offline, and blazing-fast AI code editor using **Ollama**, **Qwen 2.5 Coder**, and **Continue.dev**.

---

## 📊 Hardware Cheat Sheet (Which Model Should You Pull?)

The secret to high speed on local GPUs is picking the right **4-bit quantization (`q4_K_S` or `q4_K_M`)** so the model weights + KV context cache fit 100% inside your VRAM without spilling over into slow system RAM.

| Your GPU VRAM | Best Chat / Refactor Model | Best Tab-Autocomplete Model | Recommended Context (`num_ctx`) |
| :--- | :--- | :--- | :--- |
| **4GB - 6GB** | `qwen2.5-coder:7b-instruct-q4_K_S` | `qwen2.5-coder:1.5b` | `4096` |
| **8GB (Sweet Spot)** | `qwen2.5-coder:14b-instruct-q4_K_S` | `qwen2.5-coder:1.5b` | `8192` |
| **12GB - 16GB** | `qwen2.5-coder:14b-instruct-q5_K_M` | `qwen2.5-coder:3b` | `16384` |
| **24GB+ (RTX 3090/4090)** | `qwen2.5-coder:32b-instruct-q4_K_M` | `qwen2.5-coder:7b` | `32768` |

---

## 🛠️ Step-by-Step Installation

### Step 1: Install Ollama & Pull the Models
1. Download and install [Ollama](https://ollama.com/).
2. Open your terminal and pull our two main models (for an 8GB GPU setup) plus the embedding model for `@codebase` indexing:

```bash
# 1. Pull the 14B Q4_K_S model for heavy coding & refactoring
ollama pull qwen2.5-coder:14b-instruct-q4_K_S

# 2. Pull the lightweight 1.5B model for instant Tab Autocomplete
ollama pull qwen2.5-coder:1.5b

# 3. Pull Nomic Embed for local codebase RAG (@codebase search)
ollama pull nomic-embed-text
