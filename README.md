# Local LLM Setup
This repo containsa fully local LLM setup for local coding. 

## Tools

The setup consists of the following tools:
- [Llama-Swap](https://github.com/mostlygeek/llama-swap) to manage the llama-server and model swaps
- LLMs used: [Gemma4-26B-A4B-it-Q4_K_M](https://huggingface.co/ggml-org/gemma-4-26B-A4B-it-GGUF) & [Qwen3.6-27B-Q4_K_M](https://huggingface.co/unsloth/Qwen3.6-27B-GGUF) but any LLM with a *.gguf file would work. Mainly, these files are available from [GGML](https://huggingface.co/ggml-org) or [Unsloth](https://huggingface.co/unsloth) however there are other developers also.
- [Pi](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) as a minimal agentic coding harness

### Why These Tools Specifically?

This experiment is done to see if LLM-assisted coding is viable on a user-grade laptop. There is no dedicated Mac Mini M5 or a GPU farm. The owner of this repo is running this setup on a gaming laptop with the following specs:

- AMD Ryzen 9 8945H w/ Radeon 780M Graphic
- NVIDIA 4060 RTX 8GB Laptop GPU
- 32 GB RAM

This limits viable options both in terms of model selection/quantization and the agentic CLI harness used. The setup was tested with Claude Code using a custom base url, as well as OpenCode. But due to those tools being very feature-rich and containing large system prompts, LLM's answer to a simple "Hi" was several minutes in both. Pi on the other hand is a minimal harness, that can still perform tool-calling but outsources majority of the rules and system prompts to the user, which makes it very lightweight.

Finally, Llama-Swap handles spinning up/down llama-servers for switching models. Llama.ccp is actually what runs out models, however Llama-swap packages llama-server and other llama-* tools together to manage switching between models with different parameters.

## Getting Started
Getting started with this setup is fairly straightforward. 

### Install Llama-Swap
While there are several options for installing Llama-Swap, I opted for the binary. Full list of options are available [in their repo](https://github.com/mostlygeek/llama-swap).

**Download** (Make sure to choose the binary for your OS.)

```bash
curl -L -o llama-swap https://github.com/mostlygeek/llama-swap/releases/download/v206/llama-swap_206_windows_amd64.zip
```



Extract the binary on a preferred location, add that location to PATH (optional) and run like:

```
llama-swap.exe --config C:\Users\Elvan\Projects\local-llm-setup\llama-swap-config.yml --listen localhost:8989
```

Llama-swap relies on a configuration file. The minimal config file I am using is present in this repo, but the configurations can vary widely. Mostly my config specifies the model files and their names/configs. Refer to [documentation](https://github.com/mostlygeek/llama-swap/blob/main/docs/configuration.md) for more details.

### Running as a Permanent Server

I personally opted to run this as a permanent service in the background so I don't have to worry about terminal windows. Here is the Powershell command for this:

```Powershell
Start-Process -FilePath "llama-swap.exe" -ArgumentList "--config","C:\Users\Elvan\Projects\local-llm-setup\llama-swap-config.yml","--listen","localhost:8989" -RedirectStandardOutput
  llama-swap.log -RedirectStandardError llama-swap.err -WindowStyle Hidden
```

You can do this on Linux/macOs similarly, just swap the variables with binary, config file and your desired port:

```bash
nohup ${llama-swap-binary} --config ${llama-swap-config} --listen localhost:$PORT > llama-swap.log 2>&1 & 
```

## Setting Up Pi

Follow the setup instructions on the [repo](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent).

After installing Pi, we need to make it aware of our custom server.
