# Local LLM Coding Environment

A lightweight, fully local LLM setup optimized for efficient agentic coding on consumer-grade hardware. This repository provides a configuration and orchestration layer to run high-performance models like Gemma 4 and Qwen 3.6 using Llama-Swap and the Pi coding agent.

## Overview

This setup is designed to enable LLM-assisted coding without the need for expensive cloud APIs or high-end GPU clusters. It leverages specialized tools to manage model switching, server orchestration, and a minimal agentic interface, making it ideal for laptops and workstations with limited VRAM.

## Key Components

- **[Llama-Swap](https://github.com/mostlygeek/llama-swap)**: Orchestrates `llama-server` instances and automates model switching/swapping.
- **[Pi](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent)**: A minimal, lightweight agentic coding harness designed for efficient tool-calling and low overhead.
- **GGUF Models**: Support for high-performance quantized models such as:
  - [Gemma 4 26B](https://huggingface.co/ggml-org/gemma-4-26B-A4B-it-GGUF)
  - [Qwen 3.6 27B](https://huggingface.co/unsloth/Qwen3.6-27B-GGUF)

## Hardware Requirements & Design Philosophy

This configuration is optimized for mid-range consumer hardware (e.g., a gaming laptop). The primary goal is to minimize latency and system overhead.

**Target Hardware Example:**
- **CPU**: AMD Ryzen 9 8945H
- **GPU**: NVIDIA RTX 4060 (8GB VRAM)
- **RAM**: 32 GB

**Design Philosophy:**
Unlike heavy-weight agentic tools that rely on massive system prompts, this setup uses **Pi** for its minimal footprint. By outsourcing the majority of rules and system instructions to the user, Pi maintains high responsiveness even on hardware with limited compute resources.

## Getting Started

### 1. Prerequisites 

#### 1.1 Installing Llama.cpp

Llama.cpp must be installed to have access to llama-server that runs in the backend. Follow the instructions for your system. https://github.com/ggml-org/llama.cpp/blob/master/docs/install.md

#### 1.2 Model Preparation

You can use pre-downloaded `.gguf` files or leverage the `-hf` flag in Llama-Swap to download models directly from HuggingFace.

**Recommended Models:**
- **Gemma 4 26B (High Performance):** [Download Link](https://huggingface.co/ggml-org/gemma-4-26B-A4B-it-GGUF/tree/main)
- **Qwen 3.6 27B (High Accuracy):** [Download Link](https://huggingface.co/unsloth/Qwen3.6-27B-GGUF/tree/main)

*Note: Model performance depends heavily on your available VRAM. If running on limited hardware, prioritize smaller quantizations.*

### 2. Installing Llama-Swap

The easiest way to get started is by using the pre-compiled binary.

```bash
# Download the Windows AMD64 binary
curl -L -o llama-swap.zip https://github.com/mostlygeek/llama-swap/releases/download/v206/llama-swap_206_windows_amd64.zip

# Extract the contents and add the directory to your PATH (optional)
unzip llama-swap.zip
```

### 3. Configuration and Execution

Llama-Swap uses a `.yml` configuration file to manage model endpoints and parameters. A sample configuration is provided in this repository.

**Run Llama-Swap:**
```bash
llama-swap.exe --config ./llama-swap-config.yml --listen localhost:LLSWAP_PORT
```
*Replace `LLSWAP_PORT` with your desired port. You can access the Llama-Swap UI at `http://127.0.0.1:LLSWAP_PORT`.*

**Important:** In the configuration file, use `${PORT}` for internal Llama-Swap port assignment. Do **not** replace this placeholder manually.

#### Running as a Background Service (Windows)

To run the server persistently in the background using PowerShell:

```powershell
Start-Process -FilePath "llama-swap.exe" -ArgumentList "--config","./llama-swap-config.yml","--listen","localhost:LLSWAP_PORT" -RedirectStandardOutput llama-swap.log -RedirectStandardError llama-swap.err -WindowStyle Hidden
```

#### Running as a Background Service (Linux/macOS)

```bash
nohup ./llama-swap --config ./llama-swap-config.yml --listen localhost:LLSWAP_PORT > llama-swap.log 2>&1 & 
```

### 4. Setting Up Pi

Follow the setup instructions on the [Pi repository](https://github.com/badlogic/pi-mode/tree/main/packages/coding-agent).

After installing Pi, you must configure it to use the Llama-Swap OpenAI-compatible endpoints.

1. Locate your Pi installation directory (default: `~/.pi`).
2. Navigate to `$PI_DIR/agent/extensions`. Create the `extensions` directory if it doesn't exist.
3. Create a `custom_llms.ts` file with the following content. Ensure the `id`s match the model IDs defined in your `llama-swap-config.yml`.

```typescript
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  // Register new provider with models
  pi.registerProvider("llama", {
    baseUrl: "http://127.0.0.1:LLSWAP_PORT/v1",
    apiKey: "llama",
    api: "openai-completions",
    models: [
      {
        id: "gemma4-26b-no-reasoning",
        name: "Gemma4 26B No Reasoning",
        reasoning: false,
        input: ["text", "image"],
        cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
        contextWindow: 128000,
        maxTokens: 4096
      },
      {
        id: "gemma4-26b-reasoning",
        name: "Gemma4 26B Reasoning",
        reasoning: true,
        input: ["api", "image"],
        cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
        contextWindow: 128000,
        maxTokens: 4096
      },
      {
        id: "qwen3.6-27b-reasoning",
        name: "QWEN3.6 27B Reasoning",
        reasoning: true,
        input: ["text", "image"],
        cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
        contextWindow: 128000,
        maxTokens: 4096
      },
      {
        id: "qwen3.6-27b-no-reasoning",
        name: "QWEN3.6 27B No Reasoning",
        reasoning: false,
        input: ["text", "image"],
        cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
        contextWindow: 128000,
        maxTokens: 4096
      }
    ]
  });
}
```
