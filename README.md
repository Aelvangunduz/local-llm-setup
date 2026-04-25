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

### Download Model Files

Model files for Qwen3.6 27B can be found here: https://huggingface.co/unsloth/Qwen3.6-27B-GGUF/tree/main I downloaded the Qwen3.6-27B-Q4_K_M.gguf quantization but get whatever you want. This _barely_ runs on my machine, too slow to be of any use really, so depending on your hardware it may be good or impossible to run. Gemma 4 26B is here: https://huggingface.co/ggml-org/gemma-4-26B-A4B-it-GGUF/tree/main It is SIGNIFICANTLY faster on my hardware.

You can also just not download these and use the `-hf` flag with the provider/model names in the config file. This will download the snapshots into a default HuggingFace directory in your computer and reference that. 

Sample command to replace the command in config:

```
llama-server --port ${PORT} -hf ggml-org/gemma-4-26B-A4B-it-GGUF --no-mmap --fit on -c 128000 --reasoning on
```

### Install Llama-Swap
While there are several options for installing Llama-Swap, I opted for the binary. Full list of options are available [in their repo](https://github.com/mostlygeek/llama-swap).

**Download** (Make sure to choose the binary for your OS.)

```bash
curl -L -o llama-swap https://github.com/mostlygeek/llama-swap/releases/download/v206/llama-swap_206_windows_amd64.zip
```



Extract the binary on a preferred location, add that location to PATH (optional) and run like:

```
llama-swap.exe --config C:\Users\Elvan\Projects\local-llm-setup\llama-swap-config.yml --listen localhost:$LLSWAP_PORT
```
Replace $LLSWAP_PORT with any port you prefer. **This will be your main URL**. You can now visit http://127.0.0.1:LLSWAP_PORT to see your Llama-Swap UI. 

Llama-swap relies on a configuration file. The minimal config file I am using is present in this repo, but the configurations can vary widely. Mostly my config specifies the model files and their names/configs. Refer to [documentation](https://github.com/mostlygeek/llama-swap/blob/main/docs/configuration.md) for more details.


**Important** DO NOT replace `${PORT}` with a port number. This is a setting that allows llama-swap to **automatically** assign a port to each model. So `LLSWAP_PORT` is a user assigned port number, whatever port you want to use to access the UI and the models but `${PORT}` is an internal reference that must not be replaced.

### Running as a Permanent Server

I personally opted to run this as a permanent service in the background so I don't have to worry about terminal windows. Here is the Powershell command for this:

```Powershell
Start-Process -FilePath "llama-swap.exe" -ArgumentList "--config","C:\Users\Elvan\Projects\local-llm-setup\llama-swap-config.yml","--listen","localhost:$LLSWAP_PORT" -RedirectStandardOutput
  llama-swap.log -RedirectStandardError llama-swap.err -WindowStyle Hidden
```

You can do this on Linux/macOs similarly, just swap the variables with binary, config file:

```bash
nohup ${llama-swap-binary} --config ${llama-swap-config} --listen localhost:$LLSWAP_PORT > llama-swap.log 2>&1 & 
```

## Setting Up Pi

Follow the setup instructions on the [repo](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent).

After installing Pi, it must be made aware of the Llama-swap's OpenAI compatible rest endpoints. 

Steps:

- Find where the Pi was installed to (i.e. PI_DIR). By default it is in `~/.pi`
- Go to $PI_DIR/agent/extenstions. If you haven't installed extensions before, this dir may not exist, if so create it.
- Create a `custom_llms.ts` file with the contents similar to what's below. Pi will automatically source this file and your models will show under /model command. Model names can be whatever you like but the **ids must match to llama-swap model ids**.

```TypeScript
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
        input: ["text", "image"],
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
