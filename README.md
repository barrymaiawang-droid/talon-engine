# Talon

Talon runs Qwen3.8-Flash-Next on one AMD Strix Halo with 128 GB of memory, with an OpenAI-compatible API. Free to use: see [LICENSE](LICENSE).

## Install

**You need:** AMD Ryzen AI Max+ 395 (Radeon 8060S) with 128 GB, Linux with glibc 2.38 or newer (Ubuntu 24.04 or later, Debian 13, Fedora 40 or later), and about 125 GB of free disk. Tested on Fedora 44 with kernel 7.2.

**1. Set the GPU memory limit (once, needs root).** Set the BIOS GPU memory (UMA) to its minimum, add these kernel boot parameters, and reboot:

```text
amdgpu.gttsize=126976 ttm.pages_limit=32505856
```

Add yourself to the `render` and `video` groups, then log in again.

**2. Download Talon.**

```bash
curl -LO https://github.com/barrymaiawang-droid/talon-engine/releases/download/v0.1.0/talon-0.1.0-linux-x86_64.tar.gz
tar -xzf talon-0.1.0-linux-x86_64.tar.gz
cd talon-0.1.0-linux-x86_64
```

**3. Download the model.** Accept the licence on the [model page](https://huggingface.co/cybermonkpunk/Qwen3.8-Flash-Next-Talon-E1) and [create a read token](https://huggingface.co/settings/tokens). The download verifies every file and can be resumed by running it again.

```bash
HF_TOKEN=<your token> ./fetch-model.sh
```

**4. Start the server.** Stop anything else using the GPU first. It checks the machine and prints the fix for any problem.

```bash
./talon
```

**5. Try it** from another terminal, once `curl localhost:18160/health` reports `"status":"ok"` (about a minute):

```bash
curl localhost:18160/v1/chat/completions -H 'content-type: application/json' \
  -d '{"messages":[{"role":"user","content":"What is 17*23+41? Reply with just the number."}]}'
```

The reply contains `432`. Point any OpenAI client at `http://localhost:18160/v1`; the API key can be anything.

### For AI agents

Run steps 2 to 5 as written. Step 1 needs a reboot and root: ask the user to do it. Long steps: run `./fetch-model.sh` and `./talon` in the background. Success signals: the fetch prints `Model ready in <dir>`; the server's `/health` returns `"status":"ok"`. If `./talon` exits with code 2, it printed each problem and its fix on stderr: apply them and retry. Options: `./talon --help` (`--model-dir`, `--port`, `--ctx`).

## Speed

One Strix Halo, version 0.1.0, prompt cache off, thinking off, 256-token answers, two runs of each. Tokens per second.

| Context depth | Reading the prompt | Writing, code | Writing, prose |
| ---: | ---: | ---: | ---: |
| 2,048 | 1,400 | 65 | 57 |
| 8,192 | 1,460 | 65 | 58 |
| 32,768 | 1,390 | 62 | 56 |
| 65,536 | 1,310 | 58 | 54 |
| 131,072 | 1,190 | 59 | 51 |
| 200,000 | 1,090 | 59 | 50 |
| 258,000 | 1,020 | 58 | 47 |

With the prompt cache on, the next turn of a 29,000-token chat started in about 1 second instead of 22.

## Features

- Context up to 262,144 tokens
- OpenAI-compatible API: `/v1/chat/completions`, `/v1/completions`, `/v1/models`, with streaming
- Tool calls and thinking (reasoning) output
- Prompt cache in memory and on disk (up to 64 GiB in `~/.cache/talon/pcache`), so long conversations continue quickly
- Speculative decoding for requests without a temperature; set a temperature to sample
- Stops repeating loops: a hidden nudge gets the model moving again
- One self-contained download: no ROCm or Python to install

Requests run one at a time, in order.
