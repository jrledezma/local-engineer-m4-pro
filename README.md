# OpenCode Ultra-Optimized Local Environment for Mac M4 Pro (48GB)

This repository contains the production-ready configuration files to run an ultra-fast, local, and 100% offline AI software engineering agent using **OpenCode** and **Ollama (Qwen 3 Coder 30B)**. 

It is tailor-made to exploit the unified memory bandwidth of Apple Silicon chips, specifically tested on a **MacBook Pro M4 Pro with 48GB of Unified Memory**.

## Why this Configuration Exists (The Problem it Solves)
Standard local agent setups frequently suffer from **stream interruption, freezing during the planning phase, loops, and broken formatting** due to raw control characters (`\n\r`). 

This setup fixes those issues by stripping artificial thread limits, enforcing absolute determinism at the provider level, and implementing strict agent constraints.

---

## Architecture & Optimizations Explained

### 1. Hardware & Memory Optimization (`oc` Script)
Traditional setups force CPU thread limits or aggressive KV Cache quantization (`q4_0`) which degrade reasoning over large contexts. This setup:
* **Removes Thread Caps:** Allows macOS and Apple Metal API to dynamically orchestrate GPU and Performance cores.
* **Native f16 KV Cache:** Preserves maximum precision across long contexts without degradation.
* **V8 Engine Tuning:** Allocates `--max-old-space-size=4096` to prevent global Node.js plugins from lagging.
* **Caffeinate Integration:** Prevents system sleep during long background build tasks.

### 2. Context & Buffer Management (`Modelfile`)
* **49,152 Context Window:** The ideal sweet-spot for a 30B model on 48GB RAM before encountering performance swap or memory degradation.
* **Controlled Output (`num_predict 8192`):** Prevents Ollama from flooding the transmission buffer, eliminating client-side connection dropouts.
* **Strict Stop Tokens:** Enforces `<|im_end|>` to cleanly cut generation when a sub-task completes.

### 3. Structural Constraints (`config.json`)
* **Deterministic Sampling:** `temperature: 0.0`, `top_k: 1`, and `top_p: 1.0` eliminate creative variations, ensuring math and code syntax remain mathematically rigid.
* **Formatting Escapes:** Explicitly bans literal escape sequences (`\n`, `\t`) that crash terminal-pty rendering engines.
* **Auto-Compaction:** Triggers local history pruning with a `4096` token reserve, ensuring the agent self-manages memory automatically before saturating the context window.

---

## Installation Guide

### Step 1: Install Prerequisites (Ollama & OpenCode)
Before configuring the environment, ensure you have both core runtimes installed on your macOS system:

* **Ollama:** Download and install the official Apple Silicon binary from [ollama.com](https://ollama.com) or install it via Homebrew:
  ```bash
  brew install ollama
  ```
* **OpenCode CLI:** Install the core agent engine via bash download pipe:
  ```bash
  curl -fsSL https://opencode.ai | bash
  ```

### Step 2: Install Global Dependencies
Install the official OpenCode background utilities and formatting engines globally via npm:
```bash
npm install -g @nick-vi/opencode-type-inject envsitter-guard opencode-pty @mohak34/opencode-notifier
```

### Step 3: Provision the Local AI Model
Pull the core model weights and compile the optimized development variant using Ollama:
```bash
# Pull the base model weights
ollama pull qwen3-coder:30b

# Build the customized model using the repository's Modelfile
ollama create qwen-coder-dev -f Modelfile
```

### Step 4: Deploy Config Files
Move the provided configuration files into your local directory structure:
```bash
mkdir -p ~/.config/opencode
cp config.json ~/.config/opencode/config.json
cp opencode-notifier.json ~/.config/opencode/opencode-notifier.json
```

### Step 5: Setup the Execution Alias
Move the optimized execution script into your system binaries and grand execution permissions:
```bash
sudo cp oc /usr/local/bin/oc
sudo chmod +x /usr/local/bin/oc
```

Now you can spin up your ultra-fast, local coding assistant anywhere by simply typing:
```bash
oc
```

---

## Operational Mechanics (How to use it like a Senior Engineer)
While this setup mimics the speed and autonomy of cloud tools like *Claude Code*, local models operate best under incremental supervision. 

1. **Maintain State Files:** Keep `agents.md`, `specs.md`, and `todo.md` in your project root. The agent relies on these files to structure its tasks.
2. **Segment Scope:** Do not ask the agent to build entire platforms in a single prompt. Command it to solve tasks step-by-step using its interactive checklist.
3. **Resetting Context:** If a task accumulates heavy data and you notice micro-delays in reasoning, type `/new` in the interface to wipe the buffer while preserving the sidebar todo state.
